### Navigation

*   index
*   next |
*   previous |
*   Google 开源项目风格指南 »

# Google 开源项目风格指南 (中文版)

*   在线文档托管在 ReadTheDocs : [在线阅读最新版本](http://zh-google-styleguide.readthedocs.org/) [http://zh-google-styleguide.readthedocs.org/]
*   中文风格指南 GitHub 托管地址：[zh-google-styleguide](https://github.com/zh-google-styleguide/zh-google-styleguide) [https://github.com/zh-google-styleguide/zh-google-styleguide]

Note

声明.

本项目并非 Google 官方项目, 而是由国内程序员凭热情创建和维护.

如果你关注的是 Google 官方英文版, 请移步 [Google Style Guide](https://github.com/google/styleguide) [https://github.com/google/styleguide]

每个较大的开源项目都有自己的风格指南: 关于如何为该项目编写代码的一系列约定 (有时候会比较武断). 当所有代码均保持一致的风格, 在理解大型代码库时更为轻松.

“风格” 的含义涵盖范围广, 从 “变量使用驼峰格式 (camelCase)” 到 “决不使用全局变量” 再到 “决不使用异常”. 英文版项目维护的是在 Google 使用的编程风格指南. 如果你正在修改的项目源自 Google, 你可能会被引导至 英文版项目页面, 以了解项目所使用的风格.

我们已经发布了五份 **中文版** 的风格指南:

1.  [Google C++ 风格指南](http://zh-google-styleguide.readthedocs.org/en/latest/google-cpp-styleguide/) [http://zh-google-styleguide.readthedocs.org/en/latest/google-cpp-styleguide/]
2.  [Google Objective-C 风格指南](http://zh-google-styleguide.readthedocs.org/en/latest/google-objc-styleguide/) [http://zh-google-styleguide.readthedocs.org/en/latest/google-objc-styleguide/]
3.  [Google Python 风格指南](http://zh-google-styleguide.readthedocs.org/en/latest/google-python-styleguide/) [http://zh-google-styleguide.readthedocs.org/en/latest/google-python-styleguide/]
4.  [Google JSON 风格指南](https://github.com/darcyliu/google-styleguide/blob/master/JSONStyleGuide.md) [https://github.com/darcyliu/google-styleguide/blob/master/JSONStyleGuide.md]
5.  [Google Shell 风格指南](http://zh-google-styleguide.readthedocs.org/en/latest/google-shell-styleguide/) [http://zh-google-styleguide.readthedocs.org/en/latest/google-shell-styleguide/]

中文版项目采用 reStructuredText 纯文本标记语法, 并使用 Sphinx 生成 HTML / CHM / PDF 等文档格式.

*   英文版项目还包含 [cpplint](https://github.com/google/styleguide/tree/gh-pages/cpplint) [https://github.com/google/styleguide/tree/gh-pages/cpplint] - 一个用来帮助适应风格准则的工具, 以及 [google-c-style.el](https://raw.githubusercontent.com/google/styleguide/gh-pages/google-c-style.el) [https://raw.githubusercontent.com/google/styleguide/gh-pages/google-c-style.el], Google 风格的 Emacs 配置文件.
*   另外, 招募志愿者翻译 [JavaScript Style Guide](http://google.github.io/styleguide/javascriptguide.xml) [http://google.github.io/styleguide/javascriptguide.xml] 以及 [XML Document Format Style Guide](http://google.github.io/styleguide/xmlstyle.html) [http://google.github.io/styleguide/xmlstyle.html], 有意者请联系 [Yang.Y](https://github.com/yangyubo) [https://github.com/yangyubo].

© Copyright . Created using [Sphinx](http://sphinx-doc.org/) 1.3.5.