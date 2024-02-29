### 导航

*   索引
*   下一页 |
*   上一页 |
*   Graphic Design Patterns »
*   行为型模式 »

 # 3\. 观察者模式

目录

*   观察者模式
    *   模式动机
    *   模式定义
    *   模式结构
    *   时序图
    *   代码分析
    *   模式分析
    *   实例
    *   优点
    *   缺点
    *   适用环境
    *   模式应用
    *   模式扩展
    *   总结

## 3.1\. 模式动机

建立一种对象与对象之间的依赖关系，一个对象发生改变时将自动通知其他对象，其他对象将相应做出反应。在此，发生改变的对象称为观察目标，而被通知的对象称为观察者，一个观察目标可以对应多个观察者，而且这些观察者之间没有相互联系，可以根据需要增加和删除观察者，使得系统更易于扩展，这就是观察者模式的模式动机。

## 3.2\. 模式定义

观察者模式(Observer Pattern)：定义对象间的一种一对多依赖关系，使得每当一个对象状态发生改变时，其相关依赖对象皆得到通知并被自动更新。观察者模式又叫做发布-订阅（Publish/Subscribe）模式、模型-视图（Model/View）模式、源-监听器（Source/Listener）模式或从属者（Dependents）模式。

观察者模式是一种对象行为型模式。

## 3.3\. 模式结构

观察者模式包含如下角色：

*   Subject: 目标
*   ConcreteSubject: 具体目标
*   Observer: 观察者
*   ConcreteObserver: 具体观察者

![../_images/Obeserver.jpg](img/Obeserver.jpg)

## 3.4\. 时序图

![../_images/seq_Obeserver.jpg](img/seq_Obeserver.jpg)

## 3.5\. 代码分析

|  
```
 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
```

 |  
```
#include <iostream> #include "Subject.h" #include "Obeserver.h" #include "ConcreteObeserver.h" #include "ConcreteSubject.h" using namespace std;

int main(int argc, char *argv[])
{
 Subject * subject = new ConcreteSubject();  Obeserver * objA = new ConcreteObeserver("A");  Obeserver * objB = new ConcreteObeserver("B");  subject->attach(objA);  subject->attach(objB);    subject->setState(1);  subject->notify();    cout << "--------------------" << endl;  subject->detach(objB);  subject->setState(2);  subject->notify(); 	
	delete subject;
	delete objA;
	delete objB;

	return 0;
} 
```

 |

|  
```
 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
```

 |  
```
///////////////////////////////////////////////////////////
//  Subject.h
//  Implementation of the Class Subject
//  Created on:      07-十月-2014 23:00:10
//  Original author: cl
///////////////////////////////////////////////////////////

#if !defined(EA_61998456_1B61_49f4_B3EA_9D28EEBC9649__INCLUDED_)
#define EA_61998456_1B61_49f4_B3EA_9D28EEBC9649__INCLUDED_

#include "Obeserver.h" #include <vector> using namespace std;

class Subject
{

public:
	Subject();
	virtual ~Subject();
	Obeserver *m_Obeserver;

 void attach(Obeserver * pObeserver);  void detach(Obeserver * pObeserver);  void notify(); 		
	virtual int getState() = 0;
	virtual void setState(int i)= 0;

private:
 vector<Obeserver*> m_vtObj; 
};
#endif // !defined(EA_61998456_1B61_49f4_B3EA_9D28EEBC9649__INCLUDED_) 
```

 |

|  
```
 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
```

 |  
```
///////////////////////////////////////////////////////////
//  Subject.cpp
//  Implementation of the Class Subject
//  Created on:      07-十月-2014 23:00:10
//  Original author: cl
///////////////////////////////////////////////////////////

#include "Subject.h" Subject::Subject(){

}

Subject::~Subject(){

}

void Subject::attach(Obeserver * pObeserver){  m_vtObj.push_back(pObeserver); }   void Subject::detach(Obeserver * pObeserver){  for(vector<Obeserver*>::iterator itr = m_vtObj.begin();  itr != m_vtObj.end(); itr++)  {  if(*itr == pObeserver)  {  m_vtObj.erase(itr);  return;  }  } }   void Subject::notify(){  for(vector<Obeserver*>::iterator itr = m_vtObj.begin();  itr != m_vtObj.end();  itr++)  {  (*itr)->update(this);  } } 
```

 |

|  
```
 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
16
17
18
19
20
21
```

 |  
```
///////////////////////////////////////////////////////////
//  Obeserver.h
//  Implementation of the Class Obeserver
//  Created on:      07-十月-2014 23:00:10
//  Original author: cl
///////////////////////////////////////////////////////////

#if !defined(EA_2C7362B2_0B22_4168_8690_F9C7B76C343F__INCLUDED_)
#define EA_2C7362B2_0B22_4168_8690_F9C7B76C343F__INCLUDED_

class Subject; 
class Obeserver
{

public:
	Obeserver();
	virtual ~Obeserver();
	virtual void update(Subject * sub) = 0;
};
#endif // !defined(EA_2C7362B2_0B22_4168_8690_F9C7B76C343F__INCLUDED_) 
```

 |

|  
```
 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
```

 |  
```
///////////////////////////////////////////////////////////
//  ConcreteObeserver.h
//  Implementation of the Class ConcreteObeserver
//  Created on:      07-十月-2014 23:00:09
//  Original author: cl
///////////////////////////////////////////////////////////

#if !defined(EA_7B020534_BFEA_4c9e_8E4C_34DCE001E9B1__INCLUDED_)
#define EA_7B020534_BFEA_4c9e_8E4C_34DCE001E9B1__INCLUDED_
#include "Obeserver.h" #include <string> using namespace std;

class ConcreteObeserver : public Obeserver
{

public:
	ConcreteObeserver(string name);
	virtual ~ConcreteObeserver();
	virtual void update(Subject * sub);

private:
	string m_objName;
	int m_obeserverState;
};
#endif // !defined(EA_7B020534_BFEA_4c9e_8E4C_34DCE001E9B1__INCLUDED_) 
```

 |

|  
```
 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
```

 |  
```
///////////////////////////////////////////////////////////
//  ConcreteObeserver.cpp
//  Implementation of the Class ConcreteObeserver
//  Created on:      07-十月-2014 23:00:09
//  Original author: cl
///////////////////////////////////////////////////////////

#include "ConcreteObeserver.h" #include <iostream> #include <vector> #include "Subject.h" using namespace std;

ConcreteObeserver::ConcreteObeserver(string name){
	m_objName = name;
}

ConcreteObeserver::~ConcreteObeserver(){

}

void ConcreteObeserver::update(Subject * sub){  m_obeserverState = sub->getState();  cout << "update oberserver[" << m_objName << "] state:" << m_obeserverState << endl; } 
```

 |

运行结果：

![../_images/Obeserver_run.jpg](img/Obeserver_run.jpg)

## 3.6\. 模式分析

*   观察者模式描述了如何建立对象与对象之间的依赖关系，如何构造满足这种需求的系统。
*   这一模式中的关键对象是观察目标和观察者，一个目标可以有任意数目的与之相依赖的观察者，一旦目标的状态发生改变，所有的观察者都将得到通知。
*   作为对这个通知的响应，每个观察者都将即时更新自己的状态，以与目标状态同步，这种交互也称为发布-订阅(publishsubscribe)。目标是通知的发布者，它发出通知时并不需要知道谁是它的观察者，可以有任意数目的观察者订阅它并接收通

知。

## 3.7\. 实例

## 3.8\. 优点

观察者模式的优点

*   观察者模式可以实现表示层和数据逻辑层的分离，并定义了稳定的消息更新传递机制，抽象了更新接口，使得可以有各种各样不同的表示层作为具体观察者角色。
*   观察者模式在观察目标和观察者之间建立一个抽象的耦合。
*   观察者模式支持广播通信。
*   观察者模式符合“开闭原则”的要求。

## 3.9\. 缺点

观察者模式的缺点

*   如果一个观察目标对象有很多直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间。
*   如果在观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行循环调用，可能导致系统崩溃。
*   观察者模式没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化。

## 3.10\. 适用环境

在以下情况下可以使用观察者模式：

*   一个抽象模型有两个方面，其中一个方面依赖于另一个方面。将这些方面封装在独立的对象中使它们可以各自独立地改变和复用。
*   一个对象的改变将导致其他一个或多个对象也发生改变，而不知道具体有多少对象将发生改变，可以降低对象之间的耦合度。
*   一个对象必须通知其他对象，而并不知道这些对象是谁。
*   需要在系统中创建一个触发链，A 对象的行为将影响 B 对象，B 对象的行为将影响 C 对象……，可以使用观察者模式创建一种链式触发机制。

## 3.11\. 模式应用

观察者模式在软件开发中应用非常广泛，如某电子商务网站可以在执行发送操作后给用户多个发送商品打折信息，某团队战斗游戏中某队友牺牲将给所有成员提示等等，凡是涉及到一对一或者一对多的对象交互场景都可以使用观察者模式。

## 3.12\. 模式扩展

MVC 模式

*   MVC 模式是一种架构模式，它包含三个角色：模型(Model)，视图(View)和控制器(Controller)。观察者模式可以用来实现 MVC 模式，观察者模式中的观察目标就是 MVC 模式中的模型(Model)，而观察者就是 MVC 中的视图(View)，控制器(Controller)充当两者之间的中介者(Mediator)。当模型层的数据发生改变时，视图层将自动改变其显示内容。

## 3.13\. 总结

*   观察者模式定义对象间的一种一对多依赖关系，使得每当一个对象状态发生改变时，其相关依赖对象皆得到通知并被自动更新。观察者模式又叫做发布-订阅模式、模型-视图模式、源-监听器模式或从属者模式。观察者模式是一种对象行为型模式。
*   观察者模式包含四个角色：目标又称为主题，它是指被观察的对象；具体目标是目标类的子类，通常它包含有经常发生改变的数据，当它的状态发生改变时，向它的各个观察者发出通知；观察者将对观察目标的改变做出反应；在具体观察者中维护一个指向具体目标对象的引用，它存储具体观察者的有关状态，这些状态需要和具体目标的状态保持一致。
*   观察者模式定义了一种一对多的依赖关系，让多个观察者对象同时监听某一个目标对象，当这个目标对象的状态发生变化时，会通知所有观察者对象，使它们能够自动更新。
*   观察者模式的主要优点在于可以实现表示层和数据逻辑层的分离，并在观察目标和观察者之间建立一个抽象的耦合，支持广播通信；其主要缺点在于如果一个观察目标对象有很多直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间，而且如果在观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行循环调用，可能导致系统崩溃。
*   观察者模式适用情况包括：一个抽象模型有两个方面，其中一个方面依赖于另一个方面；一个对象的改变将导致其他一个或多个对象也发生改变，而不知道具体有多少对象将发生改变；一个对象必须通知其他对象，而并不知道这些对象是谁；需要在系统中创建一个触发链。
*   在 JDK 的 java.util 包中，提供了 Observable 类以及 Observer 接口，它们构成了 Java 语言对观察者模式的支持。 © 版权所有 2014, Colin http://blog.me115.com. 由 [Sphinx](http://sphinx-doc.org/) 1.3.5 创建。