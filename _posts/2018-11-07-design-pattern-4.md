---
layout: post
title:  "第四章 设计模式-结构型模式"
date:   2018-11-07 22:45:52
categories: 设计模式
published: true
comments: true
thread: 20181107224555555
---
第四章 结构型模式
---
结构型模式涉及到如何组合类和对象以获得更大的结构。

  结构型类模式采用继承机制来 组合接口或实现。一个简单的例子是采用多重继承方法将两个以上的类组合成一个类，结果 这个类包含了所有父类的性质。 Adapter(4.1)模式。一般来说，适配器使得一个接口(adaptee的接口)与其他接口兼容，从而给出了多个不同接口的统一抽象。

  结构型对象模式不是对接口和实现进行组合，而是描述了如何对一些对象进行组合，从 而实现新功能的一些方法。

## 一、ADAPTER(适配器) — 类对象结构型模式
1. 意图
将一个类的接口转换成客户希望的另外一个接口。 而不能一起工作的那些类可以一起工作。

2. 别名
包装器 Wrapper

3. 动机
为复用而设计的工具箱类不能够被复用的原因仅仅是因为它的接口与专业应用领域所需要的接口不匹配。

4. 适用性
5. 结构
6. 参与者
7. 协作
8. 效果
9. 实现
10. 代码示例
11. 已知应用
12. 相关模式

## 二、BRIDGE(桥接)— 对象结构型模式
1. 意图
将抽象部分与它的实现部分分离，使它们都可以独立地变化。

2. 别名
Handle/Body

3. 动机
当一个抽象可能有多个实现时，通常用继承来协调它们。抽象类定义对该抽象的接口，而具体的子类则用不同方式加以实现。但是此方法有时不够灵活。继承机制将抽象部分与它 的实现部分固定在一起，使得难以对抽象部分和实现部分独立地进行修改、扩充和重用。

4. 适用性
    - 不希望在抽象和它的实现部分之间有一个固定的绑定关系。例如这种情况可能是因为，在程序运行时刻实现部分应可以被选择或者切换。
    - 类的抽象以及它的实现都应该可以通过生成子类的方法加以扩充。这时Bridge模式使你可以对不同的抽象接口和实现部分进行组合，并分别对它们进行扩充。
    -

5. 结构
```plantuml
Abstraction o-right- Implementor

abstract class Abstraction {
  + void Operation()
  ---
  Implementor imp;
}
note bottom of Abstraction
imp --> OperationImpl();
operation() 依靠各种具体实现 OperationImpl（）实现
end note

class RefinedAbstraction extends Abstraction {
}

interface Implementor {
  + void OperationImpl();
}

class ConcreteImplementorA implements Implementor {
}

class ConcreteImplementorB implements Implementor {
}

```

6. SOLID
 - 单一职责
 - 开闭原则
 - 接口分离


## 三、Composite(组合)— 对象结构型模式
1. 意图
对象组合成树形结构以表示“部分-整体”的层次结构。Composite使得用户对单个对象和组合对象的使用具有一致性。类似于递归。

## 四、DECORATOR(装饰)— 对象结构型模式
1. 意图
动态地给一个对象添加一些额外的职责。就增加功能来说，更为灵活。Decorator模式相比生成子类
2. 别名
包装器 Wrapper
3. 动机
有时我们希望给某个对象而不是整个类添加一些功能。例如，一个图形用户界面工具箱允许你对任意一个用户界面组件添加一些特性，例如边框，或是一些行为，例如窗口滚动。 `使用继承机制是添加功能的一种有效途径`，从其他类继承过来的边框特性可以被多个子类的实例所使用。但这种方法不够灵活，因为边框的选择是静态的，用户不能控制对组件加边框的方式和时机。
`一种较为灵活的方式是将组件嵌入另一个对象中，由这个对象添加边框`。我们称这个嵌入的对象为 装饰 。这个装饰与它所装饰的组件接口一致，因此它对使用该组件的客户透明。 它将客户请求转发给该组件，并且可能在转发前后执行一些额外的动作(例如画一个边框)。 透明性使得你可以递归的嵌套多个装饰，从而可以添加任意多的功能，如下图所示。

4. 适用性
    - 在不影响其他对象的情况下，以动态、透明的方式给单个对象添加职责
    - 从对象中分离出可以部分职责
    - 子类扩展不可行时。导致两种情况：扩展可行单会类爆炸；扩展不可行

5. 结构
```plantuml
Decorator o-left- Component
abstract class Component {
  operation()
}

class ConcreteComponent extends Component {
}

abstract class Decorator {
  operation()
  ---
  Component component;
}

class ConcreteDecoratorA extends Decorator {
  addedState;
}

class ConcreteDecoratorB extends Decorator {
  addedBehavior();
}

note right of Decorator
component->operation()
end note

note right of ConcreteDecoratorB
Decorator:operation()
addedBehavior();
end note

```
6. 实现
改变对象外壳与改变对象内核我们可以将Decorator看作一个对象的外壳，它可以改变这个对象的行为。另外一种方法是改变对象的内核。例如，Strategy(5.9)模式就是一个用于改变内核的很好的模式。

由于Decorator模式仅从外部改变组件，因此组件无需对它的装饰有任何了解;也就是说，这些装饰对该组件是透明的。

在Strategy模式中，component组件本身知道可能进行哪些扩充，因此它必须引用并维护相应的策略。
基于Strategy的方法可能需要修改component组件以适应新的扩充。另一方面，一个策略可以有自己特定的接口，而装饰的接口则必须与组件的接口一致。

7. 相关模式
Adapter(4.1)模式:Decorator模式不同于Adapter模式，因为装饰仅改变对象的职责而不改变它的接口;而适配器将给对象一个全新的接口。

Composite(4.3)模式:可以将装饰视为一个退化的、仅有一个组件的组合。然而，装饰仅给对象添加一些额外的职责—它的目的不在于对象聚集。

Strategy(5.9)模式:用一个装饰你可以改变对象的外表;而Strategy模式使得你可以改变对象的内核。这是改变对象的两种途径。

8. SOLID
 - 单一职责：修饰功能从抽象类中剥离出来
 - 开闭原则：修饰可以扩展
 - 里氏代换原则：修饰器出现的地方也可用抽象类代替
 - 依赖倒置：有多种修饰时，依赖于修饰器的抽象，不依赖实现


## 五、FACADE(外观)— 对象结构型模式
1. 动机
将一个系统划分成为若干个子系统有利于降低系统的复杂性。一个常见的设计目标是使子系统间的通信和相互依赖关系达到最小。达到该目标的途径之一是就是引入一个外观(facade)对象，它为子系统中较一般的设施提供了一个单一而简单的界面。

2. SOLID
  - 封装

## 六、FLYWEIGHT(享元)— 对象结构型模式
1. 意图
运用共享技术有效地支持大量细粒度的对象。

2. 动机
有些应用程序得益于在其整个设计过程中采用对象技术，但简单化的实现代价极大。

3. 结构
```plantuml
FlyweightFactory o-right- Flyweight
FlyweightFactory <-up- Client
ConcreteFlyweight <-up- Client
UnsharedConcreteFlyweight <-up- Client

class FlyweightFactory {
  + getFlyweight(key)
}

abstract class Flyweight {
  +operation()
}

class ConcreteFlyweight extends Flyweight {

}

class UnsharedConcreteFlyweight extends Flyweight {

}
```

4. 实现
对象信息分两类，内部状态和外部状态
    - 内部状态是对象可共享出来的信息，存储在享元对象内部并且不会随环境改变而改变，如我们例子中的id、postAddress等，它们可以作为一个对象的动态附加信息，不必直接储存在具体某个对象中，属于可以共享的部分。
    - 外部状态是对象得以依赖的一个标记，是随环境改变而改变的、不可以共享的状态，如我们例子中的考试科目+考试地点复合字符串，它是一批对象的统一标识，是唯一的一个索引值

5. 协作
    - Flyweight——抽象享元角色
      它简单地说就是一个产品的抽象类，同时定义出对象的外部状态和内部状态的接口或实现。
    - ConcreteFlyweight——具体享元角色
    具体的一个产品类，实现抽象角色定义的业务。该角色中需要注意的是内部状态处理应该与环境无关，不应该出现一个操作改变了内部状态，同时修改了外部状态，这是绝对不允许的。
    - unsharedConcreteFlyweight——不可共享的享元角色
    不存在外部状态或者安全要求（如线程安全）不能够使用共享技术的对象，该对象一般不会出现在享元工厂中
    - FlyweightFactory——享元工厂
    职责非常简单，就是构造一个池容器，同时提供从池中获得对象的方法。


## 七、PROXY(代理)— 对象结构型模式
1. 相关模式
Adapter(4.1):适配器Adapter为它所适配的对象提供了一个不同的接口。相反，代理提供了与它的实体相同的接口。然而，用于访问保护的代理可能会拒绝执行实体会执行的操作，因此，它的接口实际上可能只是实体接口的一个子集。
Decorator(4.4):尽管decorator的实现部分与代理相似，但decorator的目的不一样。Decorator为对象添加一个或多个功能，而代理则控制对对象的访问。
代理的实现与decorator的实现类似，但是在相似的程度上有所差别。ProtectionProxy的实现可能与decorator的实现差不多。另一方面，RemoteProxy不包含对实体的直接引用，而只是一个间接引用，如“主机ID，主机上的局部地址。”VirtualProxy开始的时候使用一个间接引用，例如一个文件名，但最终将获取并使用一个直接引用。
