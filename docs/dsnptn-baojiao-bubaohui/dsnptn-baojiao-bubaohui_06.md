# 接口隔离原则

# 接口隔离原则(Interface Segregation Principle)

## 简介

接口隔离原则（英语：interface-segregation principles， 缩写：ISP）指明没有客户(client)应该被迫依赖于它不使用方法。接口隔离原则(ISP)拆分非常庞大臃肿的接口成为更小的和更具体的接口，这样客户将会只需要知道他们感兴趣的方法。这种缩小的接口也被称为角色接口（role interfaces）。接口隔离原则(ISP)的目的是系统解开耦合，从而容易重构，更改和重新部署。接口隔离原则是在 SOLID (面向对象设计)中五个面向对象设计(OOD)的原则之一，类似于在 GRASP (面向对象设计)中的高内聚性。

## 实例

定义：客户端不应该依赖它不需要的接口；一个类对另一个类的依赖应该建立在最小的接口上。 问题由来：类 A 通过接口 I 依赖类 B，类 C 通过接口 I 依赖类 D，如果接口 I 对于类 A 和类 B 来说不是最小接口，则类 B 和类 D 必须去实现他们不需要的方法。 解决方案：将臃肿的接口 I 拆分为独立的几个接口，类 A 和类 C 分别与他们需要的接口建立依赖关系。也就是采用接口隔离原则。 举例来说明接口隔离原则：

![pic1](img/pic0.jpg)

这个图的意思是：类 A 依赖接口 I 中的方法 1、方法 2、方法 3，类 B 是对类 A 依赖的实现。类 C 依赖接口 I 中的方法 1、方法 4、方法 5，类 D 是对类 C 依赖的实现。对于类 B 和类 D 来说，虽然他们都存在着用不到的方法（也就是图中红色字体标记的方法），但由于实现了接口 I，所以也必须要实现这些用不到的方法。对类图不熟悉的可以参照程序代码来理解，代码如下：

```
interface I {  
    public void method1();  
    public void method2();  
    public void method3();  
    public void method4();  
    public void method5();  
}  

class A{  
    public void depend1(I i){  
        i.method1();  
    }  
    public void depend2(I i){  
        i.method2();  
    }  
    public void depend3(I i){  
        i.method3();  
    }  
}  

class B implements I{  
    public void method1() {  
        System.out.println("类 B 实现接口 I 的方法 1");  
    }  
    public void method2() {  
        System.out.println("类 B 实现接口 I 的方法 2");  
    }  
    public void method3() {  
        System.out.println("类 B 实现接口 I 的方法 3");  
    }  
    //对于类 B 来说，method4 和 method5 不是必需的，但是由于接口 A 中有这两个方法，  
    //所以在实现过程中即使这两个方法的方法体为空，也要将这两个没有作用的方法进行实现。  
    public void method4() {}  
    public void method5() {}  
}  

class C{  
    public void depend1(I i){  
        i.method1();  
    }  
    public void depend2(I i){  
        i.method4();  
    }  
    public void depend3(I i){  
        i.method5();  
    }  
}  

class D implements I{  
    public void method1() {  
        System.out.println("类 D 实现接口 I 的方法 1");  
    }  
    //对于类 D 来说，method2 和 method3 不是必需的，但是由于接口 A 中有这两个方法，  
    //所以在实现过程中即使这两个方法的方法体为空，也要将这两个没有作用的方法进行实现。  
    public void method2() {}  
    public void method3() {}  

    public void method4() {  
        System.out.println("类 D 实现接口 I 的方法 4");  
    }  
    public void method5() {  
        System.out.println("类 D 实现接口 I 的方法 5");  
    }  
}  

public class Client{  
    public static void main(String[] args){  
        A a = new A();  
        a.depend1(new B());  
        a.depend2(new B());  
        a.depend3(new B());  

        C c = new C();  
        c.depend1(new D());  
        c.depend2(new D());  
        c.depend3(new D());  
    }  
} 
```

可以看到，如果接口过于臃肿，只要接口中出现的方法，不管对依赖于它的类有没有用处，实现类中都必须去实现这些方法，这显然不是好的设计。如果将这个设计修改为符合接口隔离原则，就必须对接口 I 进行拆分。在这里我们将原有的接口 I 拆分为三个接口，拆分后的设计如图 2 所示： ![pic1](img/pic1.jpg)

```
interface I1 {  
    public void method1();  
}  

interface I2 {  
    public void method2();  
    public void method3();  
}  

interface I3 {  
    public void method4();  
    public void method5();  
}  

class A{  
    public void depend1(I1 i){  
        i.method1();  
    }  
    public void depend2(I2 i){  
        i.method2();  
    }  
    public void depend3(I2 i){  
        i.method3();  
    }  
}  

class B implements I1, I2{  
    public void method1() {  
        System.out.println("类 B 实现接口 I1 的方法 1");  
    }  
    public void method2() {  
        System.out.println("类 B 实现接口 I2 的方法 2");  
    }  
    public void method3() {  
        System.out.println("类 B 实现接口 I2 的方法 3");  
    }  
}  

class C{  
    public void depend1(I1 i){  
        i.method1();  
    }  
    public void depend2(I3 i){  
        i.method4();  
    }  
    public void depend3(I3 i){  
        i.method5();  
    }  
}  

class D implements I1, I3{  
    public void method1() {  
        System.out.println("类 D 实现接口 I1 的方法 1");  
    }  
    public void method4() {  
        System.out.println("类 D 实现接口 I3 的方法 4");  
    }  
    public void method5() {  
        System.out.println("类 D 实现接口 I3 的方法 5");  
    }  
} 
```

接口隔离原则的含义是：建立单一接口，不要建立庞大臃肿的接口，尽量细化接口，接口中的方法尽量少。也就是说，我们要为各个类建立专用的接口，而不要试图去建立一个很庞大的接口供所有依赖它的类去调用。本文例子中，将一个庞大的接口变更为 3 个专用的接口所采用的就是接口隔离原则。在程序设计中，依赖几个专用的接口要比依赖一个综合的接口更灵活。接口是设计时对外部设定的“契约”，通过分散定义多个接口，可以预防外来变更的扩散，提高系统的灵活性和可维护性。

说到这里，很多人会觉的接口隔离原则跟之前的单一职责原则很相似，其实不然。其一，单一职责原则原注重的是职责；而接口隔离原则注重对接口依赖的隔离。其二，单一职责原则主要是约束类，其次才是接口和方法，它针对的是程序中的实现和细节；而接口隔离原则主要约束接口接口，主要针对抽象，针对程序整体框架的构。 采用接口隔离原则对接口进行约束时，要注意以下几点：

*   接口尽量小，但是要有限度。对接口进行细化可以提高程序设计灵活性是不挣的事实，但是如果过小，则会造成接口数量过多，使设计复杂化。所以一定要适度。
*   为依赖接口的类定制服务，只暴露给调用的类它需要的方法，它不需要的方法则隐藏起来。只有专注地为一个模块提供定制服务，才能建立最小的依赖关系。
*   提高内聚，减少对外交互。使接口用最少的方法去完成最多的事情。 运用接口隔离原则，一定要适度，接口设计的过大或过小都不好。设计接口的时候，只有多花些时间去思考和筹划，才能准确地实践这一原则。