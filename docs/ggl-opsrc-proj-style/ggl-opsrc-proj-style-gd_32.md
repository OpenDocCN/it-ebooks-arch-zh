### Navigation

*   index
*   next |
*   previous |
*   Google 开源项目风格指南 »
*   Shell 风格指南 - 内容目录 »

# 注释

## 文件头

Tip

每个文件的开头是其文件内容的描述。

每个文件必须包含一个顶层注释，对其内容进行简要概述。版权声明和作者信息是可选的。

例如：

> ```
> #!/bin/bash
> #
> # Perform hot backups of Oracle databases. 
> ```

## 功能注释

Tip

任何不是既明显又短的函数都必须被注释。任何库函数无论其长短和复杂性都必须被注释。

其他人通过阅读注释（和帮助信息，如果有的话）就能够学会如何使用你的程序或库函数，而不需要阅读代码。

所有的函数注释应该包含：

*   函数的描述
*   全局变量的使用和修改
*   使用的参数说明
*   返回值，而不是上一条命令运行后默认的退出状态

例如：

> ```
> #!/bin/bash
> #
> # Perform hot backups of Oracle databases.
> 
> export PATH='/usr/xpg4/bin:/usr/bin:/opt/csw/bin:/opt/goog/bin'
> 
> #######################################
> # Cleanup files from the backup dir
> # Globals:
> #   BACKUP_DIR
> #   ORACLE_SID
> # Arguments:
> #   None
> # Returns:
> #   None
> #######################################
> cleanup() {
>   ...
> } 
> ```

## 实现部分的注释

Tip

注释你代码中含有技巧、不明显、有趣的或者重要的部分。

这部分遵循谷歌代码注释的通用做法。不要注释所有代码。如果有一个复杂的算法或者你正在做一些与众不同的，放一个简单的注释。

## TODO 注释

Tip

使用 TODO 注释临时的、短期解决方案的、或者足够好但不够完美的代码。

这与 C++指南中的约定相一致。

TODOs 应该包含全部大写的字符串 TODO，接着是括号中你的用户名。冒号是可选的。最好在 TODO 条目之后加上 bug 或者 ticket 的序号。

例如：

> ```
> # TODO(mrmonkey): Handle the unlikely edge cases (bug ####) 
> ```

© Copyright . Created using [Sphinx](http://sphinx-doc.org/) 1.3.5.