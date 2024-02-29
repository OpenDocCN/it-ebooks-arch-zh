### Navigation

*   index
*   next |
*   previous |
*   Google 开源项目风格指南 »

 # Objective-C 风格指南 - 内容目录

*   Google Objective-C Style Guide 中文版
    *   译者的话
        *   ewanke
        *   Yang.Y
    *   背景介绍
    *   例子
*   留白和格式
    *   空格 vs. 制表符
    *   行宽
    *   方法声明和定义
    *   方法调用
    *   `@public` 和 `@private`
    *   异常
    *   协议名
    *   块（闭包）
*   命名
    *   文件名
    *   Objective-C++
    *   类名
    *   类别名
    *   Objective-C 方法名
    *   变量名
        *   普通变量名
        *   实例变量
        *   常量
*   注释
    *   文件注释
        *   版权信息及作者
    *   声明部分的注释
    *   实现部分的注释
    *   对象所有权
*   Cocoa 和 Objective-C 特性
    *   成员变量应该是 `@private`
    *   明确指定构造函数
    *   重载指定构造函数
    *   重载 `NSObject` 的方法
    *   初始化
    *   避免 `+new`
    *   保持公共 API 简单
    *   `#import` and `#include`
    *   使用根框架
    *   构建时即设定 `autorelease`
    *   `autorelease` 优先 `retain` 其次
    *   `init` 和 `dealloc` 内避免使用访问器
    *   按声明顺序销毁实例变量
    *   `setter` 应复制 NSStrings
    *   避免抛异常
    *   nil 检查
    *   BOOL 若干陷阱
    *   属性（Property）
        *   命名
        *   位置
        *   字符串应使用 `copy` 属性（Attribute）
        *   原子性
        *   点引用
    *   没有实例变量的接口
    *   自动 `synthesize` 实例变量
*   Cocoa 模式
    *   委托模式
    *   模型/视图/控制器（MVC） © Copyright . Created using [Sphinx](http://sphinx-doc.org/) 1.3.5.