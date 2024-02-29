# 架构篇二：构建基于 Git 为数据中心的 CMS

### 简介

或许你也用过 Hexo / Jekyll / Octopress 这样的静态博客，他们的原理都是类似的。我们有一个代码库用于生成静态页面，然后这些静态页面会被 PUSH 到 Github Pages 上。

从我们设计系统的角度来说，我们会在 Github 上有三个主要代码库：

1.  Content。用于存放编辑器生成的 JSON 文件，这样我们就可以 GET 这些资源，并用 Backbone / Angular / React 这些前端框架来搭建 SPA。
2.  Code。开发者在这里存放他们的代码，如主题、静态文件生成器、资源文件等等。
3.  Builder。在这里它是运行于 Travis CI 上的一些脚本文件，用于 Clone 代码，并执行 Code 中的脚本。

以及一些额外的服务，当且仅当你有一些额外的功能需求的时候。

1.  Extend Service。当我们需要搜索服务时，我们就需要这样的一些服务。如我正考虑使用 Python 的 whoosh 来完成这个功能，这时候我计划用 Flask 框架，但是只是计划中——因为没有合适的中间件。
2.  Editor。相比于前面的那些知识这一步适合更重要，也就是为什么生成的格式是 JSON 而不是 Markdown 的原理。对于非程序员来说，要熟练掌握 Markdown 不是一件容易的事。于是，一个考虑中的方案就是使用 Electron + Node.js 来生成 API，最后通过 GitHub API V3 来实现上传。
3.  Mobile App。

So，这一个过程是如何进行的。

### 用户场景

整个过程的 Pipeline 如下所示：

1.  编辑使用他们的编辑器来编辑的内容并点击发布，然后这个内容就可以通过 GitHub API 上传到 Content 这个 Repo 里。
2.  这时候需要有一个 WebHooks 监测到了 Content 代码库的变化，便运行 Builder 这个代码库的 Travis CI。
3.  这个 Builder 脚本首先，会设置一些基本的 git 配置。然后 clone Content 和 Code 的代码，接着运行构建命令，生成新的内容。
4.  然后 Builder Commit 内容，并 PUSH 内容。

在这种情形中，编辑能否完成工作就不依赖于网站——脱稿又少了 个借口。这时候网站出错的概率太小了——你不需要一个缓存服务器、HTTP 服务器，由于没有动态生成的内容，你也不需要守护进程。这些内容都是静态文件，你可以将他们放在任何可以提供静态文件托管的地方——CloudFront、S3 等等。或者你再相信自己的服务器，Nginx 可是全球第二好（第一还没出现）的静态文件服务器。

开发人员只在需要的时候去修改网站的一些内容。So，你可能会担心如果这时候修改的东西有问题了怎么办。

1.  使用这种模式就意味着你需要有测试来覆盖这些构建工具、生成工具。
2.  相比于自己的代码，别人的 CMS 更可靠？

需要注意的是如果你上一次构建成功，你生成的文件都是正常的，那么你只需要回滚开发相关的代码即可。旧的代码仍然可以工作得很好。其次，由于生成的是静态文件，查错的成本就比较低。最后，重新放上之前的静态文件。

### Code: 生成静态页面

Assemble 是一个使用 Node.js，Grunt.js，Gulp，Yeoman 等来实现的静态网页生成系统。这样的生成器有很多，Zurb Foundation, Zurb Ink, Less.js / lesscss.org, Topcoat, Web Experience Toolkit 等组织都使用这个工具来生成。这个工具似乎上个 Release 在一年多以前，现在正在开始 0.6。虽然，这并不重要，但是还是顺便一说。

我们所要做的就是在我们的`Gruntfile.js`中写相应的生成代码。

```
          assemble: {
      options: {
        flatten: true,
        partials: ['templates/includes/*.hbs'],
        layoutdir: 'templates/layouts',
        data: 'content/blogs.json',
        layout: 'default.hbs'
      },
      site: {
        files: {'dest/': ['templates/*.hbs']}
      },
      blogs: {
        options: {
          flatten: true,
          layoutdir: 'templates/layouts',
          data: 'content/*.json',
          partials: ['templates/includes/*.hbs'],
          pages: pages
        },
        files: [
          { dest: './dest/blog/', src: '!*' }
        ]
      }
    }

```

配置中的 site 用于生成页面相关的内容，blogs 则可以根据 json 文件的文件名生成对就的 html 文件存储到 blog 目录中。

生成后的目录结果如下图所示：

```
       .
├── about.html
├── blog
│   ├── blog-posts.html
│   └── blogs.html
├── blog.html
├── css
│   ├── images
│   │   └── banner.jpg
│   └── style.css
├── index.html
└── js
    ├── jquery.min.js
    └── script.js

7 directories, 30 files

```

这里的静态文件内容就是最后我们要发布的内容。

还需要做的一件事情就是：

```
      grunt.registerTask('dev', ['default', 'connect:server', 'watch:site']);

```

用于开发阶段这样的代码就够了，这个和你使用 WebPack + React 似乎相差不了多少。

### Builder: 构建生成工具

Github 与 Travis 之间，可以做一个自动部署的工具。相信已经有很多人在 Github 上玩过这样的东西——先在 Github 上生成 Token，然后用 travis 加密：

```
      travis encrypt-file ssh_key --add

```

加密后的 Key 就会保存到`.travis.yml`文件里，然后就可以在 Travis CI 上 push 你的代码到 Github 上了。

接着，你需要创建个 deploy 脚本，并且在`after_success`执行它：

```
      after_success:
  - test $TRAVIS_PULL_REQUEST == "false" && test $TRAVIS_BRANCH == "master" && bash deploy.sh

```

在这个脚本里，你所需要做的就是 clone content 和 code 中的代码，并执行 code 中的生成脚本，生成新的内容后，提交代码。

```
      #!/bin/bash

set -o errexit -o nounset

rev=$(git rev-parse --short HEAD)

cd stage/

git init
git config user.name "Robot"
git config user.email "robot@phodal.com"

git remote add upstream "https://$GH_TOKEN@github.com/phodal-archive/echeveria-deploy.git"
git fetch upstream
git reset upstream/gh-pages

git clone https://github.com/phodal-archive/echeveria-deploy code
git clone https://github.com/phodal-archive/echeveria-content content
pwd
cp -a content/contents code/content

cd code

npm install
npm install grunt-cli -g
grunt 
mv dest/* ../
cd ../
rm -rf code
rm -rf content

touch .

if [ ! -f CNAME ]; then
    echo "deploy.baimizhou.net" > CNAME
fi

git add -A .
git commit -m "rebuild pages at ${rev}"
git push -q upstream HEAD:gh-pages

```

这就是这个 builder 做的事情——其中最主要的一个任务是`grunt`，它所做的就是:

```
      grunt.registerTask('default', ['clean', 'assemble', 'copy']);

```

### Content：JSON 格式

在使用 Github 和 Travis CI 完成 Content 的时候，发现没有一个好的 Webhook。虽然我们的 Content 只能存储一些数据，但是放一个 trigger 脚本也是可以原谅的。

```
      var Travis = require('travis-ci');

var repo = "phodal-archive/echeveria-deploy";

var travis = new Travis({
    version: '2.0.0'
});

travis.authenticate({
    github_token: process.env.GH_TOKEN

}, function (err, res) {
    if (err) {
        return console.error(err);
    }

    travis.repos(repo.split('/')[0], repo.split('/')[1]).builds.get(function (err, res) {
        if (err) {
            return console.error(err);
        }

        travis.requests.post({
            build_id: res.builds[0].id
        }, function (err, res) {
            if (err) {
                return console.error(err);
            }
            console.log(res.flash[0].notice);
        });
    });
});

```

这里主要依赖于 Travis CI 来完成这部分功能，这时候我们还需要数据。

### 从 Schema 到数据库

我们在我们数据库中定义好了 Schema——对一个数据库的结构描述。在《[编辑-发布-开发分离](https://www.phodal.com/blog/editing-publishing-coding-seperate/) 》一文中我们说到了 echeveria-content 的一个数据文件如下所示：

```
        {
    "title": "白米粥",
    "author": "白米粥",
    "url": "baimizhou",
    "date": "2015-10-21",
    "description": "# Blog post \n  > This is an example blog post \n Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. ",
    "blogpost": "# Blog post \n  > This is an example blog post \n Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. \n Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum."
  }

```

比起之前的直接生成静态页面这里的数据就是更有意思地一步了，我们从数据库读取数据就是为了生成一个 JSON 文件。何不直接以 JSON 的形式存储文件呢？

我们都定义了这每篇文章的基本元素:

1.  title
2.  author
3.  date
4.  description
5.  content
6.  url

即使我们使用 NoSQL 我们也很难逃离这种模式。我们定义这些数据，为了在使用的时候更方便。存储这些数据只是这个过程中的一部分，下部分就是取出这些数据并对他们进行过滤，取出我们需要的数据。

Web 的骨架就是这么简单，当然 APP 也是如此。难的地方在于存储怎样的数据，返回怎样的数据。不同的网站存储着不同的数据，如淘宝存储的是商品的信息，Google 存储着各种网站的数据——人们需要不同的方式去存储这些数据，为了更好地存储衍生了更多的数据存储方案——于是有了 GFS、Haystack 等等。运营型网站想尽办法为最后一公里努力着，成长型的网站一直在想着怎样更好的返回数据，从更好的用户体验到机器学习。而数据则是这个过程中不变的东西。

尽管，我已经想了很多办法去尽可能减少元素——在最开始的版本里只有标题和内容。然而为了满足我们在数据库中定义的结构，不得不造出来这么多对于一般用户不友好的字段。如链接名是为了存储的文件名而存在的，即这个链接名在最后会变成文件名：

```
      repo.write('master', 'contents/' + data.url + '.json', stringifyData, 'Robot: add article ' + data.title, options, function (err, data) {
      if(data.commit){
        that.setState({message: "上传成功" + JSON.stringify(data)});
        that.refs.snackbar.show();
        that.setState({
          sending: 0
        });
      }
    });

```

然后，上面的数据就会变成一个对象存储到“数据库”中。

今天 ，仍然有很多人用 Word、Excel 来存储数据。因为对于他们来说，这些软件更为直接，他们简单地操作一下就可以对数据进行排序、筛选。数据以怎样的形式存储并不重要，重要的是他们都以文件的形式存储着。

### git 作为 NoSQL 数据库

不同的数据库会以不同的形式存储到文件中去。blob 是 git 中最为基本的存储单位，我们的每个 content 都是一个 blob。redis 可以以 rdb 文件的形式存储到文件系统中。完成一个 CMS，我们并不需要那么多的查询功能。

> 这些上千年的组织机构，只想让人们知道他们想要说的东西。

我们使用 NoSQL 是因为：

1.  不使用关系模型
2.  在集群中运行良好
3.  开源
4.  无模式
5.  数据交换格式

我想其中只有两点对于我来说是比较重要的`集群`与`数据格式`。但是集群和数据格式都不是我们要考虑的问题。。。

我们也不存在数据格式的问题、开源的问题，什么问题都没有。。除了，我们之前说到的查询——但是这是可以解决的问题，我们甚至可以返回不同的历史版本的。在这一点上 git 做得很好，他不会像 WordPress 那样存储多个版本。

JSON 文件 + Nginx 就可以变成这样一个合理的 API，甚至是运行方式。我们可以对其进行增、删、改、查，尽管就当前来说查需要一个额外的软件来执行，但是为了实现一个用得比较少的功能，而去花费大把的时间可能就是在浪费。

git 的“API”提供了丰富的增、删、改功能——你需要 commit 就可以了。我们所要做的就是:

1.  git commit
2.  git push

于是，就会有一个很忙的 Travis-Github Robot 在默默地为你工作。

![Robot 提交代码](img/2015-12-28_5680cac378bad.png)

Robot 提交代码

### 一键发布：编辑器

为了实现之前说到的`编辑-发布-开发分离`的 CMS，我还是花了两天的时间打造了一个面向普通用户的编辑器。效果截图如下所示：

![编辑器](img/2015-12-28_5680cac3a04fc.png)

编辑器

作为一个普通用户，这是一个很简单的软件。除了 Electron + Node.js + React 作了一个 140M 左右的软件，尽管压缩完只有 40M 左右 ，但是还是会把用户吓跑的。不过作为一个快速构建的原型已经很不错了——构建速度很快、并且运行良好。

*   Electron
*   React
*   Material UI
*   Alloy Editor

尽管这个界面看上去还是稍微复杂了一下，还在试着想办法将链接名和日期去掉——问题是为什么会有这两个东西？

Webpack 打包

```
        if (process.env.HOT) {
    mainWindow.loadUrl('file://' + __dirname + '/app/hot-dev-app.html');
  } else {
    mainWindow.loadUrl('file://' + __dirname + '/app/app.html');
  }

```

上传代码

```
      repo.write('master', 'content/' + data.url + '.json', stringifyData, 'Robot: add article ' + data.title, options, function (err, data) {
  if(data.commit){
    that.setState({message: "上传成功" + JSON.stringify(data)});
    that.refs.snackbar.show();
    that.setState({
      sending: 0
    });
  }
});

```

当我们点下发送的时侯，这个内容就直接提交到了 Content Repo 下，如上上图所示。

当我们向 Content Push 代码的时候，就会运行一下 Trigger 脚本：

```
      after_success:
  - node trigger-build.js

```

脚本的代码如下所示：

```
      var Travis = require('travis-ci');

var repo = "phodal-archive/echeveria-deploy";
var travis = new Travis({
    version: '2.0.0'
});

travis.authenticate({
    github_token: process.env.GH_TOKEN

}, function (err, res) {
    if (err) {
        return console.error(err);
    }
    travis.repos(repo.split('/')[0], repo.split('/')[1]).builds.get(function (err, res) {
        if (err) {
            return console.error(err);
        }

        travis.requests.post({
            build_id: res.builds[0].id
        }, function (err, res) {
            if (err) {
                return console.error(err);
            }
            console.log(res.flash[0].notice);
        });
    });
});

```

由于，我们在这个过程我们的 Content 提交的是 JSON 数据，我们可以直接用这些数据做一个 APP。

### 移动应用

为了快速开发，这里我们使用了 Ionic + ngCordova 来开发 ，最后效果图如下所示：

![移动应用](img/2015-12-28_5680cac3c6c3d.png)

移动应用

在这个代码库里，主要由两部分组成：

1.  获取全部文章
2.  获取特定文章

为了获取全部文章就意味着，我们在 Builder 里，需要一个 task 来合并 JSON 文件，并删掉其中的一些无用的内容，如 articleHTML 和 article。最后，将生成一个名为 articles.json。

```
      if (!grunt.file.exists(src))
    throw "JSON source file \"" + chalk.red(src) + "\" not found.";
else {
    var fragment;
    grunt.log.debug("reading JSON source file \"" + chalk.green(src) + "\"");
    try {
        fragment = grunt.file.readJSON(src);
    }
    catch (e) {
        grunt.fail.warn(e);
    }
    fragment.description = sanitizeHtml(fragment.article).substring(0, 200);
    delete fragment.article;
    delete fragment.articleHTML;
    json.push(fragment);
}

```

接着，我们就可以获取所有的文章然后显示~~。在这里又顺便加了一个 pullToRefresh。

```
        .controller('ArticleListsCtrl', function ($scope, Blog) {
    $scope.articles = null;
    $scope.blogOffset = 0;
    $scope.doRefresh = function () {
      Blog.async('http://deploy.baimizhou.net/api/blog/articles.json').then(function (results) {
        $scope.articles = results;
      });
      $scope.$broadcast('scroll.refreshComplete');
      $scope.$apply()
    };
    Blog.async('http://deploy.baimizhou.net/api/blog/articles.json').then(function (results) {
      $scope.articles = results;
    });
  })

```

最后，当我们点击特定的 url，将跳转到相应的页面：

```
      <ion-item class="item item-icon-right" ng-repeat="article in articles" type="item-text-wrap" href="#/app/article/{{article.url}}">
  <h2>{{article.title}}</h2>
  <i class="icon ion-ios-arrow-right"></i>
</ion-item>

```

就会交由相应的 Controller 来处理。

```
        .controller('ArticleCtrl', function ($scope, $stateParams, $sanitize, $sce, Blog) {
    $scope.article = {};
    Blog.async('http://deploy.baimizhou.net/api/' + $stateParams.slug + '.json').then(function (results) {
      $scope.article = results;
      $scope.htmlContent = $sce.trustAsHtml($scope.article.articleHTML);
    });

  });

```

### 小结

尽管没有一个更成熟的环境可以探索这其中的问题，但是我想对于当前这种情况来说，它是非常棒的解决方案。我们面向的不是那些技术人员，而是一般的用户。他们能熟练使用的是：编辑器和 APP。

1.  不会因为后台的升级来困扰他们，也不会受其他组件的影响。
2.  开发人员不需要担心，某个功能影响了编辑器的使用。
3.  Ops 不再担心网站的性能问题——然后要么转为 DevOps、要么被 Fire。

### 其他

最后的代码库：

1.  Content: [`github.com/phodal-archive/echeveria-content`](https://github.com/phodal-archive/echeveria-content)
2.  Code: [`github.com/phodal-archive/echeveria-deploy`](https://github.com/phodal-archive/echeveria-deploy)
3.  移动应用: [`github.com/phodal-archive/echeveria-mobile`](https://github.com/phodal-archive/echeveria-mobile)
4.  桌面应用: [`github.com/phodal/echeveria-editor`](https://github.com/phodal/echeveria-editor)
5.  Github Pages: [`github.com/phodal-archive/echeveria-deploy/tree/gh-pages`](https://github.com/phodal-archive/echeveria-deploy/tree/gh-pages)