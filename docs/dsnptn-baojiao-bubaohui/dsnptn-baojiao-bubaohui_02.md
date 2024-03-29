# 简单工厂模式

# 简单工厂模式

## 活字印刷 面向对象

话说三国时期，曹操带领百万大军攻打东吴，大军在长江赤壁驻扎，军船连成一片，眼看就要灭掉东吴，统一天下，曹操大悦，于是大宴众文武，在酒席间，曹操诗兴大发，不觉吟道：喝酒唱歌，人生真爽。众文武齐呼：“丞相好诗！于是一臣子速命印刷工匠刻板印刷，以便流传天下。”

样张出来给曹操一看，曹操感觉不妥，说到：“喝与唱，此话过俗，应该为‘对酒当歌’较好！”，于是此臣就命工匠重新来过。工匠眼看连夜刻板之工，彻底白费，心中叫苦不迭。只得照办。”

样张再次出来请曹操过目，曹操细细一品，感觉还是不好，说：“人生真爽太过直接，应改问语才够意境，因此应改为‘对酒当歌，人生几何？’当臣子转告工匠之时，工匠晕倒！”

为何三国时期的工匠有如此的问题？

当时活字印刷还未发明，所以要改字的时候必须要整个刻板重刻。如果有活字印刷，则只需更改四个字就可，其余工作都未白做，岂不妙哉。

*   要改，只需要更改之字，此为可维护。
*   这写字并非用完这次就无用，完全可以在后来的印刷中重复使用，此乃可复用。
*   此诗若要加字，只需另刻字加入即可，这是可扩展。
*   字的排列其实可能是竖排也可能是横排，此时只需要将活字移动就可以做到满足排列需求，此是灵活性好

面对对象的分析设计编程思想，通过封装，继承多态把程序的耦合度降低，传统印刷术的问题就在于所有的字都刻在同一版面上造成耦合度太高所致，开始用设计模式使得程序更加灵活，易于修改，易于复用。

## 简单工厂模式

例：计算器，到底要实例化谁，将来会不会增加实例化的对象，把很容易变化的地方用一个单独的类来做这个创造实例的过程，这个就是工厂。 简单的运算工厂类

```
public class OperationFactory
{
    public static operation createOperate(string operate)
    {
    Operation oper = null;
    switch (operate)
    {
        case "+":
            oper = new OperationAdd();
            break;
        case "-":
            oper = new OperationSub();
            break;
        case "*":
            oper = new OperationMul();
            break;
        case "/":
            oper = new OperationDiv();
            break;
    }
    return oper;
    }
} 
```

客户端代码

```
Operation oper;
oper = OperationFactory.createOperate("+");
oper.NumberA = 1;
oper.NumberB = 2;
double result = oper.GetResult(); 
```

这样，以后需要增加各种复杂运算，比如平方根，立方根，自然对数等等，只要增加相对应的运算子类就可以了。