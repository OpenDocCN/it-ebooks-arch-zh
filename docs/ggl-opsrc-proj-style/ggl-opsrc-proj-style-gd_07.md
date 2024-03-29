# 4\. 来自 Google 的奇技

Google 用了很多自己实现的技巧 / 工具使 C++ 代码更加健壮, 我们使用 C++ 的方式可能和你在其它地方见到的有所不同.

## 4.1\. 所有权与智能指针

Tip

动态分配出的对象最好有单一且固定的所有主（onwer）, 且通过智能指针传递所有权（ownership）.

定义：

> 所有权是一种登记／管理动态内存和其它资源的技术。动态分配出的对象的所有主是一个对象或函数，后者负责确保当前者无用时就自动销毁前者。所有权有时可以共享，那么就由最后一个所有主来负责销毁它。甚至也可以不用共享，在代码中直接把所有权传递给其它对象。
> 
> 其实您可以把智能指针当成一个重载了 `*` 和 `->` 的「对象」来看。智能指针类型被用来自动化所有权的登记工作，来确保执行销毁义务到位。[std::unique_ptr](http://en.cppreference.com/w/cpp/memory/unique_ptr) [http://en.cppreference.com/w/cpp/memory/unique_ptr] 是 C++11 新推出的一种智能指针类型，用来表示动态分配出的对象的「独一无二」所有权；当 `std::unique_ptr` 离开作用域，对象就会被销毁。不能复制 `std::unique_ptr`, 但可以把它移动（move）给新所有主。[std::shared_ptr](http://en.cppreference.com/w/cpp/memory/shared_ptr) [http://en.cppreference.com/w/cpp/memory/shared_ptr] 同样表示动态分配对象的所有权，但可以被共享，也可以被复制；对象的所有权由所有复制者共同拥有，最后一个复制者被销毁时，对象也会随着被销毁。

优点：

> *   如果没有清晰、逻辑条理的所有权安排，不可能管理好动态分配的内存。
> *   传递对象的所有权，开销比复制来得小，如果可以复制的话。
> *   传递所有权也比「借用」指针或引用来得简单，毕竟它大大省去了两个用户一起协调对象生命周期的工作。
> *   如果所有权逻辑条理，有文档且不乱来的话，可读性很棒。
> *   可以不用手动完成所有权的登记工作，大大简化了代码，也免去了一大波错误之恼。
> *   对于 const 对象来说，智能指针简单易用，也比深度复制高效。

缺点：

> *   不得不用指针（不管是智能的还是原生的）来表示和传递所有权。指针语义可要比值语义复杂得许多了，特别是在 API 里：您不光要操心所有权，还要顾及别名，生命周期，可变性（mutability）以及其它大大小小问题。
> *   其实值语义的开销经常被高估，所以就所有权的性能来说，可不能光只考虑可读性以及复杂性。
> *   如果 API 依赖所有权的传递，就会害得客户端不得不用单一的内存管理模型。
> *   销毁资源并回收的相关代码不是很明朗。
> *   `std::unique_ptr` 的所有权传递原理是 C++11 的 move 语法，后者毕竟是刚刚推出的，容易迷惑程序员。
> *   如果原本的所有权设计已经够完善了，那么若要引入所有权共享机制，可能不得不重构整个系统。
> *   所有权共享机制的登记工作在运行时进行，开销可能相当不小。
> *   某些极端情况下，所有权被共享的对象永远不会被销毁，比如引用死循环（cyclic references）。
> *   智能指针并不能够完全代替原生指针。

决定：

> 如果必须使用动态分配，倾向于保持分配者的所有权。如果其他地方要使用这个对象，最好传递它的拷贝，或者传递一个不用改变所有权的指针或引用。倾向于使用 `std::unique_ptr` 来明确所有权传递，例如：
> 
> ```
> std::unique_ptr<Foo> FooFactory();
> void FooConsumer(std::unique_ptr<Foo> ptr); 
> ```
> 
> 避免使用共享所有权。如果对性能要求很高，并且操作的对象是不可变的（比如说 `std::shared_ptr<const Foo>` ），这时可以用共享所有权来避免昂贵的拷贝操作。如果确实要使用共享所有权，倾向于使用 `std::shared_ptr` 。
> 
> 不要在新代码中使用 `scoped_ptr `` ，除非你必须兼容老版本的 C++。总是用 ``std::unique_ptr` 代替 `std::auto_ptr` 。

## 4.2\. cpplint

Tip

使用 `cpplint.py` 检查风格错误.

`cpplint.py` 是一个用来分析源文件, 能检查出多种风格错误的工具. 它不并完美, 甚至还会漏报和误报, 但它仍然是一个非常有用的工具. 在行尾加 `// NOLINT`, 或在上一行加 `// NOLINTNEXTLINE`, 可以忽略报错。

某些项目会指导你如何使用他们的项目工具运行 `cpplint.py`. 如果你参与的项目没有提供, 你可以单独下载 [cpplint.py](http://github.com/google/styleguide/blob/gh-pages/cpplint/cpplint.py) [http://github.com/google/styleguide/blob/gh-pages/cpplint/cpplint.py].

## 译者（acgtyrant）笔记

1.  把智能指针当成对象来看待的话，就很好领会它与所指对象之间的关系了。
2.  原来 Rust 的 Ownership 思想是受到了 C++ 智能指针的很大启发啊。
3.  `scoped_ptr` 和 `auto_ptr` 已过时。 现在是 `shared_ptr` 和 `uniqued_ptr` 的天下了。
4.  按本文来说，似乎除了智能指针，还有其它所有权机制，值得留意。
5.  Arch Linux 用户注意了，AUR 有对 cpplint 打包。

© Copyright . Created using [Sphinx](http://sphinx-doc.org/) 1.3.5.