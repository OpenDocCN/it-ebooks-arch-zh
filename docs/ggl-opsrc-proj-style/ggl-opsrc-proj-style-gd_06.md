# 3\. 类

类是 C++ 中代码的基本单元. 显然, 它们被广泛使用. 本节列举了在写一个类时的主要注意事项.

## 3.1\. 构造函数的职责

Tip

不要在构造函数中进行复杂的初始化 (尤其是那些有可能失败或者需要调用虚函数的初始化).

定义:

> 在构造函数体中进行初始化操作.

优点:

> 排版方便, 无需担心类是否已经初始化.

缺点:

> 在构造函数中执行操作引起的问题有:
> 
> > *   构造函数中很难上报错误, 不能使用异常.
> > *   操作失败会造成对象初始化失败，进入不确定状态.
> > *   如果在构造函数内调用了自身的虚函数, 这类调用是不会重定向到子类的虚函数实现. 即使当前没有子类化实现, 将来仍是隐患.
> > *   如果有人创建该类型的全局变量 (虽然违背了上节提到的规则), 构造函数将先 `main()` 一步被调用, 有可能破坏构造函数中暗含的假设条件. 例如, [gflags](http://code.google.com/p/google-gflags/) [http://code.google.com/p/google-gflags/] 尚未初始化.

结论:

> 构造函数不得调用虚函数, 或尝试报告一个非致命错误. 如果对象需要进行有意义的 (non-trivial) 初始化, 考虑使用明确的 Init() 方法或使用工厂模式.

## 3.2\. 初始化

Tip

如果类中定义了成员变量, 则必须在类中为每个类提供初始化函数或定义一个构造函数. 若未声明构造函数, 则编译器会生成一个默认的构造函数, 这有可能导致某些成员未被初始化或被初始化为不恰当的值.

定义:

> `new` 一个不带参数的类对象时, 会调用这个类的默认构造函数. 用 `new[]` 创建数组时, 默认构造函数则总是被调用. 在类成员里面进行初始化是指声明一个成员变量的时候使用一个结构例如 `int _count = 17` 或者 `string _name{"abc"}` 来替代 `int _count` 或者 `string _name` 这样的形式.

优点:

> 用户定义的默认构造函数将在没有提供初始化操作时将对象初始化. 这样就保证了对象在被构造之时就处于一个有效且可用的状态, 同时保证了对象在被创建时就处于一个显然”不可能”的状态, 以此帮助调试.

缺点:

> 对代码编写者来说, 这是多余的工作.
> 
> 如果一个成员变量在声明时初始化又在构造函数中初始化, 有可能造成混乱, 因为构造函数中的值会覆盖掉声明中的值.

结论:

> 简单的初始化用类成员初始化完成, 尤其是当一个成员变量要在多个构造函数里用相同的方式初始化的时候.
> 
> 如果你的类中有成员变量没有在类里面进行初始化, 而且没有提供其它构造函数, 你必须定义一个 (不带参数的) 默认构造函数. 把对象的内部状态初始化成一致 / 有效的值无疑是更合理的方式.
> 
> > 这么做的原因是: 如果你没有提供其它构造函数, 又没有定义默认构造函数, 编译器将为你自动生成一个. 编译器生成的构造函数并不会对对象进行合理的初始化.
> > 
> > 如果你定义的类继承现有类, 而你又没有增加新的成员变量, 则不需要为新类定义默认构造函数.

## 3.3\. 显式构造函数

Tip

对单个参数的构造函数使用 C++ 关键字 `explicit`.

定义:

> 通常, 如果构造函数只有一个参数, 可看成是一种隐式转换. 打个比方, 如果你定义了 `Foo::Foo(string name)`, 接着把一个字符串传给一个以 `Foo` 对象为参数的函数, 构造函数 `Foo::Foo(string name)` 将被调用, 并将该字符串转换为一个 `Foo` 的临时对象传给调用函数. 看上去很方便, 但如果你并不希望如此通过转换生成一个新对象的话, 麻烦也随之而来. 为避免构造函数被调用造成隐式转换, 可以将其声明为 `explicit`.
> 
> 除单参数构造函数外, 这一规则也适用于除第一个参数以外的其他参数都具有默认参数的构造函数, 例如 Foo::Foo(string name, int id = 42).

优点:

> 避免不合时宜的变换.

缺点:

> 无

结论:

> 所有单参数构造函数都必须是显式的. 在类定义中, 将关键字 `explicit` 加到单参数构造函数前: `explicit Foo(string name);`
> 
> 例外: 在极少数情况下, 拷贝构造函数可以不声明成 `explicit`. 作为其它类的透明包装器的类也是特例之一. 类似的例外情况应在注释中明确说明.
> 
> 最后, 只有 std::initializer_list 的构造函数可以是非 explicit, 以允许你的类型结构可以使用列表初始化的方式进行赋值. 例如:
> 
> > ```
> > MyType m = {1, 2};
> > MyType MakeMyType() { return {1, 2}; }
> > TakeMyType({1, 2}); 
> > ```

 ## 3.4\. 可拷贝类型和可移动类型

Tip

如果你的类型需要, 就让它们支持拷贝 / 移动. 否则, 就把隐式产生的拷贝和移动函数禁用.

定义:

> 可拷贝类型允许对象在初始化时得到来自相同类型的另一对象的值, 或在赋值时被赋予相同类型的另一对象的值, 同时不改变源对象的值. 对于用户定义的类型, 拷贝操作一般通过拷贝构造函数与拷贝赋值操作符定义. string 类型就是一个可拷贝类型的例子.
> 
> > 可移动类型允许对象在初始化时得到来自相同类型的临时对象的值, 或在赋值时被赋予相同类型的临时对象的值 (因此所有可拷贝对象也是可移动的). std::unique_ptr<int> 就是一个可移动但不可复制的对象的例子. 对于用户定义的类型, 移动操作一般是通过移动构造函数和移动赋值操作符实现的.
> > 
> > 拷贝 / 移动构造函数在某些情况下会被编译器隐式调用. 例如, 通过传值的方式传递对象.

优点:

> 可移动及可拷贝类型的对象可以通过传值的方式进行传递或者返回, 这使得 API 更简单, 更安全也更通用. 与传指针和引用不同, 这样的传递不会造成所有权, 生命周期, 可变性等方面的混乱, 也就没必要在协议中予以明确. 这同时也防止了客户端与实现在非作用域内的交互, 使得它们更容易被理解与维护. 这样的对象可以和需要传值操作的通用 API 一起使用, 例如大多数容器.
> 
> > 拷贝 / 移动构造函数与赋值操作一般来说要比它们的各种替代方案, 比如 Clone(), CopyFrom() or Swap(), 更容易定义, 因为它们能通过编译器产生, 无论是隐式的还是通过 = 默认. 这种方式很简洁, 也保证所有数据成员都会被复制. 拷贝与移动构造函数一般也更高效, 因为它们不需要堆的分配或者是单独的初始化和赋值步骤, 同时, 对于类似省略不必要的拷贝这样的优化它们也更加合适.
> > 
> > 移动操作允许隐式且高效地将源数据转移出右值对象. 这有时能让代码风格更加清晰.

缺点:

> 许多类型都不需要拷贝, 为它们提供拷贝操作会让人迷惑, 也显得荒谬而不合理. 为基类提供拷贝 / 赋值操作是有害的, 因为在使用它们时会造成对象切割. 默认的或者随意的拷贝操作实现可能是不正确的, 这往往导致令人困惑并且难以诊断出的错误.
> 
> > 拷贝构造函数是隐式调用的, 也就是说, 这些调用很容易被忽略. 这会让人迷惑, 尤其是对那些所用的语言约定或强制要求传引用的程序员来说更是如此. 同时, 这从一定程度上说会鼓励过度拷贝, 从而导致性能上的问题.

结论:

> 如果需要就让你的类型可拷贝 / 可移动. 作为一个经验法则, 如果对于你的用户来说这个拷贝操作不是一眼就能看出来的, 那就不要把类型设置为可拷贝. 如果让类型可拷贝, 一定要同时给出拷贝构造函数和赋值操作的定义. 如果让类型可拷贝, 同时移动操作的效率高于拷贝操作, 那么就把移动的两个操作 (移动构造函数和赋值操作) 也给出定义. 如果类型不可拷贝, 但是移动操作的正确性对用户显然可见, 那么把这个类型设置为只可移动并定义移动的两个操作.
> 
> > 建议通过 `= default` 定义拷贝和移动操作. 定义非默认的移动操作目前需要异常. 时刻记得检测默认操作的正确性. 由于存在对象切割的风险, 不要为任何有可能有派生类的对象提供赋值操作或者拷贝 / 移动构造函数 (当然也不要继承有这样的成员函数的类). 如果你的基类需要可复制属性, 请提供一个 `public virtual Clone()` 和一个 `protected` 的拷贝构造函数以供派生类实现.
> > 
> > 如果你的类不需要拷贝 / 移动操作, 请显式地通过 `= delete` 或其他手段禁用之.  ## 3.5\. 委派和继承构造函数

Tip

在能够减少重复代码的情况下使用委派和继承构造函数.

定义:

> 委派和继承构造函数是由 C++11 引进为了减少构造函数重复代码而开发的两种不同的特性. 通过特殊的初始化列表语法, 委派构造函数允许类的一个构造函数调用其他的构造函数. 例如:
> 
> > ```
> > X::X(const string& name) : name_(name) {
> >   ...
> > }
> > 
> > X::X() : X("") { } 
> > ```
> > 
> > 继承构造函数允许派生类直接调用基类的构造函数, 一如继承基类的其他成员函数, 而无需重新声明. 当基类拥有多个构造函数时这一功能尤其有用. 例如:
> > 
> > ```
> > class Base {
> >  public:
> >   Base();
> >   Base(int n);
> >   Base(const string& s);
> >   ...
> > };
> > 
> > class Derived : public Base {
> >  public:
> >   using Base::Base;  // Base's constructors are redeclared here.
> > }; 
> > ```
> 
> 如果派生类的构造函数只是调用基类的构造函数而没有其他行为时, 这一功能特别有用.

优点:

> 委派和继承构造函数可以减少冗余代码, 提高可读性. 委派构造函数对 Java 程序员来说并不陌生.

缺点:

> 使用辅助函数可以预估出委派构造函数的行为. 如果派生类和基类相比引入了新的成员变量, 继承构造函数就会让人迷惑, 因为基类并不知道这些新的成员变量的存在.

结论:

> 只在能够减少冗余代码, 提高可读性的前提下使用委派和继承构造函数. 如果派生类有新的成员变量, 那么使用继承构造函数时要小心. 如果在派生类中对成员变量使用了类内部初始化的话, 继承构造函数还是适用的. 

## 3.6\. 结构体 VS. 类

Tip

仅当只有数据时使用 struct, 其它一概使用 class.

说明:

> 在 C++ 中 struct 和 class 关键字几乎含义一样. 我们为这两个关键字添加我们自己的语义理解, 以便未定义的数据类型选择合适的关键字.
> 
> struct 用来定义包含数据的被动式对象, 也可以包含相关的常量, 但除了存取数据成员之外, 没有别的函数功能. 并且存取功能是通过直接访问位域, 而非函数调用. 除了构造函数, 析构函数, Initialize(), Reset(), Validate() 等类似的函数外, 不能提供其它功能的函数.
> 
> 如果需要更多的函数功能, class 更适合. 如果拿不准, 就用 class.
> 
> 为了和 STL 保持一致, 对于仿函数和 trait 特性可以不用 class 而是使用 struct.
> 
> 注意: 类和结构体的成员变量使用不同的命名规则.

 ## 3.7\. 继承

Tip

使用组合 (composition, YuleFox 注: 这一点也是 GoF 在 <<Design Patterns>> 里反复强调的) 常常比使用继承更合理. 如果使用继承的话, 定义为 `public` 继承.

定义:

> 当子类继承基类时, 子类包含了父基类所有数据及操作的定义. C++ 实践中, 继承主要用于两种场合: 实现继承 (implementation inheritance), 子类继承父类的实现代码; 接口继承 (interface inheritance), 子类仅继承父类的方法名称.

优点:

> 实现继承通过原封不动的复用基类代码减少了代码量. 由于继承是在编译时声明, 程序员和编译器都可以理解相应操作并发现错误. 从编程角度而言, 接口继承是用来强制类输出特定的 API. 在类没有实现 API 中某个必须的方法时, 编译器同样会发现并报告错误.

缺点:

> 对于实现继承, 由于子类的实现代码散布在父类和子类间之间, 要理解其实现变得更加困难. 子类不能重写父类的非虚函数, 当然也就不能修改其实现. 基类也可能定义了一些数据成员, 还要区分基类的实际布局.

结论:

> 所有继承必须是 `public` 的. 如果你想使用私有继承, 你应该替换成把基类的实例作为成员对象的方式.
> 
> 不要过度使用实现继承. 组合常常更合适一些. 尽量做到只在 “是一个” (“is-a”, YuleFox 注: 其他 “has-a” 情况下请使用组合) 的情况下使用继承: 如果 `Bar` 的确 “是一种” Foo, `Bar` 才能继承 `Foo`.
> 
> 必要的话, 析构函数声明为 `virtual`. 如果你的类有虚函数, 则析构函数也应该为虚函数. 注意 数据成员在任何情况下都必须是私有的.
> 
> 当重载一个虚函数, 在衍生类中把它明确的声明为 `virtual`. 理论依据: 如果省略 `virtual` 关键字, 代码阅读者不得不检查所有父类, 以判断该函数是否是虚函数.  ## 3.8\. 多重继承

Tip

真正需要用到多重实现继承的情况少之又少. 只在以下情况我们才允许多重继承: 最多只有一个基类是非抽象类; 其它基类都是以 `Interface` 为后缀的 纯接口类.

定义:

> 多重继承允许子类拥有多个基类. 要将作为 *纯接口* 的基类和具有 *实现* 的基类区别开来.

优点:

> 相比单继承 (见 继承), 多重实现继承可以复用更多的代码.

缺点:

> 真正需要用到多重 *实现* 继承的情况少之又少. 多重实现继承看上去是不错的解决方案, 但你通常也可以找到一个更明确, 更清晰的不同解决方案.

结论:

> 只有当所有父类除第一个外都是 纯接口类 时, 才允许使用多重继承. 为确保它们是纯接口, 这些类必须以 `Interface` 为后缀.

Note

关于该规则, Windows 下有个 特例.  ## 3.9\. 接口

Tip

接口是指满足特定条件的类, 这些类以 `Interface` 为后缀 (不强制).

定义:

> 当一个类满足以下要求时, 称之为纯接口:
> 
> > *   只有纯虚函数 (“`=0`”) 和静态函数 (除了下文提到的析构函数).
> > *   没有非静态数据成员.
> > *   没有定义任何构造函数. 如果有, 也不能带有参数, 并且必须为 `protected`.
> > *   如果它是一个子类, 也只能从满足上述条件并以 `Interface` 为后缀的类继承.
> 
> 接口类不能被直接实例化, 因为它声明了纯虚函数. 为确保接口类的所有实现可被正确销毁, 必须为之声明虚析构函数 (作为上述第 1 条规则的特例, 析构函数不能是纯虚函数). 具体细节可参考 Stroustrup 的 *The C++ Programming Language, 3rd edition* 第 12.4 节.

优点:

> 以 `Interface` 为后缀可以提醒其他人不要为该接口类增加函数实现或非静态数据成员. 这一点对于 多重继承 尤其重要. 另外, 对于 Java 程序员来说, 接口的概念已是深入人心.

缺点:

> `Interface` 后缀增加了类名长度, 为阅读和理解带来不便. 同时，接口特性作为实现细节不应暴露给用户.

结论:

> 只有在满足上述需要时, 类才以 `Interface` 结尾, 但反过来, 满足上述需要的类未必一定以 `Interface` 结尾. 

## 3.10\. 运算符重载

Tip

除少数特定环境外，不要重载运算符.

定义:

> 一个类可以定义诸如 `+` 和 `/` 等运算符, 使其可以像内建类型一样直接操作.

优点:

> 使代码看上去更加直观, 类表现的和内建类型 (如 `int`) 行为一致. 重载运算符使 `Equals()`, `Add()` 等函数名黯然失色. 为了使一些模板函数正确工作, 你可能必须定义操作符.

缺点:

> 虽然操作符重载令代码更加直观, 但也有一些不足:
> 
> *   混淆视听, 让你误以为一些耗时的操作和操作内建类型一样轻巧.
> *   更难定位重载运算符的调用点, 查找 `Equals()` 显然比对应的 `==` 调用点要容易的多.
> *   有的运算符可以对指针进行操作, 容易导致 bug. `Foo + 4` 做的是一件事, 而 `&Foo + 4` 可能做的是完全不同的另一件事. 对于二者, 编译器都不会报错, 使其很难调试;
> 
> 重载还有令你吃惊的副作用. 比如, 重载了 `operator&` 的类不能被前置声明.

结论:

> 一般不要重载运算符. 尤其是赋值操作 (`operator=`) 比较诡异, 应避免重载. 如果需要的话, 可以定义类似 `Equals()`, `CopyFrom()` 等函数.
> 
> 然而, 极少数情况下可能需要重载运算符以便与模板或 “标准” C++ 类互操作 (如 `operator<<(ostream&, const T&)`). 只有被证明是完全合理的才能重载, 但你还是要尽可能避免这样做. 尤其是不要仅仅为了在 STL 容器中用作键值就重载 `operator==` 或 `operator<`; 相反, 你应该在声明容器的时候, 创建相等判断和大小比较的仿函数类型.
> 
> 有些 STL 算法确实需要重载 `operator==` 时, 你可以这么做, 记得别忘了在文档中说明原因.
> 
> 参考 拷贝构造函数 和 函数重载.

## 3.11\. 存取控制

Tip

将 *所有* 数据成员声明为 `private`, 并根据需要提供相应的存取函数. 例如, 某个名为 `foo_` 的变量, 其取值函数是 `foo()`. 还可能需要一个赋值函数 `set_foo()`.

特例是, 静态常量数据成员 (一般写做 kFoo) 不需要是私有成员.

一般在头文件中把存取函数定义成内联函数.

参考 继承 和 函数命名

 ## 3.11\. 声明顺序

Tip

在类中使用特定的声明顺序: `public:` 在 `private:` 之前, 成员函数在数据成员 (变量) 前;

类的访问控制区段的声明顺序依次为: `public:`, `protected:`, `private:`. 如果某区段没内容, 可以不声明.

每个区段内的声明通常按以下顺序:

> *   `typedefs` 和枚举
> *   常量
> *   构造函数
> *   析构函数
> *   成员函数, 含静态成员函数
> *   数据成员, 含静态数据成员

友元声明应该放在 private 区段. 如果用宏 DISALLOW_COPY_AND_ASSIGN 禁用拷贝和赋值, 应当将其置于 private 区段的末尾, 也即整个类声明的末尾. 参见可拷贝类型和可移动类型.

`.cc` 文件中函数的定义应尽可能和声明顺序一致.

不要在类定义中内联大型函数. 通常, 只有那些没有特别意义或性能要求高, 并且是比较短小的函数才能被定义为内联函数. 更多细节参考 内联函数. 

## 3.12\. 编写简短函数

Tip

倾向编写简短, 凝练的函数.

我们承认长函数有时是合理的, 因此并不硬性限制函数的长度. 如果函数超过 40 行, 可以思索一下能不能在不影响程序结构的前提下对其进行分割.

即使一个长函数现在工作的非常好, 一旦有人对其修改, 有可能出现新的问题. 甚至导致难以发现的 bug. 使函数尽量简短, 便于他人阅读和修改代码.

在处理代码时, 你可能会发现复杂的长函数. 不要害怕修改现有代码: 如果证实这些代码使用 / 调试困难, 或者你需要使用其中的一小段代码, 考虑将其分割为更加简短并易于管理的若干函数.

## 译者 (YuleFox) 笔记

1.  不在构造函数中做太多逻辑相关的初始化;
2.  编译器提供的默认构造函数不会对变量进行初始化, 如果定义了其他构造函数, 编译器不再提供, 需要编码者自行提供默认构造函数;
3.  为避免隐式转换, 需将单参数构造函数声明为 `explicit`;
4.  为避免拷贝构造函数, 赋值操作的滥用和编译器自动生成, 可将其声明为 `private` 且无需实现;
5.  仅在作为数据集合时使用 `struct`;
6.  组合 > 实现继承 > 接口继承 > 私有继承, 子类重载的虚函数也要声明 `virtual` 关键字, 虽然编译器允许不这样做;
7.  避免使用多重继承, 使用时, 除一个基类含有实现外, 其他基类均为纯接口;
8.  接口类类名以 `Interface` 为后缀, 除提供带实现的虚析构函数, 静态成员函数外, 其他均为纯虚函数, 不定义非静态数据成员, 不提供构造函数, 提供的话，声明为 `protected`;
9.  为降低复杂性, 尽量不重载操作符, 模板, 标准类中使用时提供文档说明;
10.  存取函数一般内联在头文件中;
11.  声明次序: `public` -> `protected` -> `private`;
12.  函数体尽量短小, 紧凑, 功能单一;

© Copyright . Created using [Sphinx](http://sphinx-doc.org/) 1.3.5.