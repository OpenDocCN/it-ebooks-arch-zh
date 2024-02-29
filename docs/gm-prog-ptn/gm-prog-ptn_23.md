# 优化模式

## 游戏设计模式

虽然越来越快的硬件解除了大部分软件在性能上的顾虑，对游戏却并非如此。 玩家总是想要更丰富的，更真实的，更激动人心的体验。 到处都是争抢玩家注意力——还有金钱——的游戏，能将硬件的功能发挥至极致的游戏往往获胜。

优化游戏性能是一门高深的艺术，接触到软件的各个层面。 底层程序员掌握硬件架构的种种特质。同时，算法研究者争先恐后地证明谁的过程是最有效率的。

这里，我描述了几个加速游戏的中间层模式。 数据局部性介绍了计算机的存储层次以及如何使用其以获得优势。 脏标识帮你避开不必要的计算。 对象池帮你避开不必要的内存分配。 空间分区加速了虚拟世界和其中元素的空间布局。

## 模式

*   数据局部性
*   脏标识
*   对象池
*   空间分区