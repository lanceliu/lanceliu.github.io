---
layout: post
title:  "第五章 设计模式-行为型模式"
date:   2018-11-12 22:45:52
categories: 设计模式
published: true
comments: true
thread: 20181112224555555
---
第五章 行为型模式
---
行为模式涉及到算法和对象间职责的分配。行为模式不仅描述对象或类的模式，还描述它们之间的通信模式。
    - 行为类模式使用继承机制在类间分派行为
      - TemplateMethod
        - 模板方法是一个算法的抽象定义，它逐步地定义该算法， 每一步调用一个抽象操作或一个原语操作，子类定义抽象操作以具体实现该算法
      - Interpreter
        - 将一个文法表示为一个类层次，并实现一个解释器作为这些类的实例上的一个操作


    - 行为对象模式使用对象复合而不是继承
      行为对象模式描述了一组对等的对象怎样 相互协作以完成其中任一个对象都无法单独完成的任务。这里一个重要的问题是对等的对象 如何互相了解对方。
      - Mediator
        - 在对等对象间引入一个 m e d i a t o r对象以避免这种情况的出现。 m e d i a t o r 提供了松耦合所需的间接性
      - Chain of Responsibility
        - 通过一条候选对象链 隐式的向一个对象发送请求。根据运行时刻情况任一候选者都可以响应相应的请求。候选者的数目是任意的， 你可以在运行时刻决定哪些候选者参与到链中
      - Observer - 定义并保持对象间的依赖关系
        - 典型的Observer的例子是Smalltalk中的模型/视图/控制器，其中一旦模型的状态发生变化，模型的所有视图都会得到通知。
      - 将行为封装在一个对象中并将请求指派给它
        - Strategy(5.9)
          模式将算法封装在对象中，这样可以方便地指定和改变一个对象所使用的算法
        - Command
          将请求封装在对象中，这样它就可作为参数来传递，也可以被存储在历史列表里，或者以其 他方式使用
        - State
          封装一个对象的状态，使得当这个对象的状态对象变化时，该对象可改变它的行为
        - Visitor
          封装分布于多个类之间的行为
        - Iterator
          抽象了访问和遍历一个集合中的对象的方式

## 一、CHAIN OF RESPONSIBILITY(职责链)— 对象行为型模式
1. 意图
   使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这 些对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。

2. 动机
   略
3. 适用性
   - 有多个的对象可以处理一个请求，哪个对象处理该请求运行时刻自动确定
   - 你想在不明确指定接收者的情况下，向多个对象中的一个提交一个请求
   - 可处理一个请求的对象集合应被动态指定
4. 结构
   ```plantuml
   Client -right-> Handler
   Handler  -->  Handler

   abstract class Handler {
     +handleRequest();

     Handler successor
   }

   class ConcreteHandler1 extends Handler {
   }
   class ConcreteHandler2 extends Handler {     
   }
   ```
5. 相关模式
职责链常与 C o m p o s i t e( 4 . 3 )一起使用。这种情况下，一个构件的父构件可作为它的后继

## 二、COMMAND(命令)— 对象行为型模式
1. 意图
将一个请求封装为一个对象，从而使你可用不同的请求对客户进行参数化;对请求排队或记录请求日志，以及支持可撤消的操作。

2. 别名
动作(Action)，事务(Transaction)

3. 动机
有时必须向某对象提交请求，但并不知道关于被请求的操作或请求的接受者的任何信息。

4. 适用性
  - 抽象出待执行的动作以参数化某对象，是回调机制的一个面向对象的替代品。
  - 支持取消/重做操作
5. 结构
```plantuml
Invoker o-right- Command
Client -right-> Receiver
ConcreteCommand <-right- Receiver

interface Command {
  +execute();
}

class ConcreteCommand implements Command {
  state
}

class Receiver{
  +action()
}
note right of ConcreteCommand
receiver->action();
end note
```
6. 参与者
Command
- 声明执行操作的接口。
•ConcreteCommand(PasteCommand，OpenCommand)
- 将一个接收者对象绑定于一个动作。
- 调用接收者相应的操作，以实现Execute。
•Client(Appliction)
- 创建一个具体命令对象并设定它的接收者。
•Invoker(MenuItem)
- 要求该命令执行这个请求。
•Receiver(Document，Application)
- 知道如何实施与执行一个请求相关的操作。任何类都可能作为一个接收者。

7. 协作
•Client创建一个ConcreteCommand对象并指定它的Receiver对象。
•某Invoker对象存储该ConcreteCommand对象。
•该Invoker通过调用Command对象的Execute操作来提交一个请求。若该命令是可撤消的，ConcreteCommand就在执行Excute操作之前存储当前状态以用于取消该命令。
•ConcreteCommand对象对调用它的Receiver的一些操作以执行该请求。

8. MVC模式
Model：Receiver 执行操作
View：Invoker 发起请求
Controller：Command 请求路由给Model

## 三、INTERPRETER(解释器)— 类行为型模式
1. 意图
给定一个语言，定义它的文法的一种表示，并定义一个解释器，这个解释器使用该表示 来解释语言中的句子.

2. 动机
如果一种特定类型的问题发生的频率足够高, 那么可能就值得将该问题的各个实例表述为一个简单语言中的句子。这样就可以构建一个解释器, 该解释器通过解释这些句子来解决该问题。

3. 适用性
当有一个语言需要解释执行 , 并且你可将该语言中的句子表示为一个抽象语法树时，可使用解释器模式。

解释器是一个简单语法分析工具，它最显著的优点就是扩展性，修改语法规则只要修改相应的非终结符表达式就可以了，若扩展语法，则只要增加非终结符类就可以了。

4. 结构
```plantuml
Context <-right- Client
AbstractExpression <-right- Client
AbstractExpression o-- TerminalExpression
AbstractExpression o-- NonterminalExpression

interface AbstractExpression {
  +interpret()
}
```

5. 参与者
❑AbstractExpression——抽象解释器
具体的解释任务由各个实现类完成，具体的解释器分别由TerminalExpression和NonterminalExpression完成。
❑TerminalExpression——终结符表达式
实现与文法中的元素相关联的解释操作，通常一个解释器模式中只有一个终结符表达式，但有多个实例，对应不同的终结符。具体到我们例子就是VarExpression类，表达式中的每个终结符都在栈中产生了一个VarExpression对象。
❑NonterminalExpression——非终结符表达式
文法中的每条规则对应于一个非终结表达式，具体到我们的例子就是加减法规则分别对应到AddExpression和SubExpression两个类。非终结符表达式根据逻辑的复杂程度而增加，原则上每个文法规则都对应一个非终结符表达式。
❑Context——环境角色
— 包含解释器之外的一些全局信息。具体到我们的例子中是采用HashMap代替

## 四、ITERATOR(迭代器)— 对象行为型模式
1. 意图
提供一种方法顺序访问一个聚合对象中各个元素 , 而又不需暴露该对象的内部表示

2. 动机
一个聚合对象, 如列表(list), 应该提供一种方法来让别人可以访问它的元素，而又不需暴露它的内部结构

3. 结构
```plantuml
Aggregate <-left- Client
Aggregate o-- ConcreteAggregate

Iterator <-right- Client
Iterator o-- ConcreteIterator

interface Aggregate {
  +createIterator();
}
interface Iterator {
  +first()
  +next()
  +isDone()
  +currentItem()
}
```

## 五、MEDIATOR(中介者)— 对象行为型模式
1. 意图
用一个中介对象来封装一系列的对象交互。中介者使各对象不需要显式地相互引用，从 而使其耦合松散，而且可以独立地改变它们之间的交互。
2. 动机
面向对象设计鼓励将行为分布到各个对象中。这种分布可能会导致对象间有许多连接。 在最坏的情况下 ,每一个对象都知道其他所有对象。
3. 适用性
在下列情况下使用中介者模式 :
• 一组对象以定义良好但是复杂的方式进行通信。产生的相互依赖关系结构混乱且难以理
解。
• 一个对象引用其他很多对象并且直接与这些对象通信 ,导致难以复用该对象。
• 想定制一个分布在多个类中的行为，而又不想生成太多的子类。

5. 参与者
•Mediator(中介者，如DialogDirector)
  —中介者定义一个接口用于与各同事(Colleague)对象通信。
•ConcreteMediator(具体中介者，如FontDialogDirector)
  —具体中介者通过协调各同事对象实现协作行为。
  —了解并维护它的各个同事。
•Colleagueclass(同事类，如ListBox,EntryField)
  — 每一个同事类都知道它的中介者对象。
  — 每一个同事对象在需与其他的同事通信的时候，与它的中介者通信。

6. 协作
• 同事向一个中介者对象发送和接收请求。中介者在各同事间适当地转发请求以实现协作行为。

7. 相关模式
Facade(4.5)与中介者的不同之处在于它是对一个对象子系统进行抽象，从而提供了一个更为方便的接口。它的协议是单向的，即Facade对象对这个子系统类提出请求，但反之则不行。相反，Mediator提供了各Colleague对象不支持或不能支持的协作行为，而且协议是多向的。
Colleague可使用Observer(5.7)模式与Mediator通信

## 六、MEMENTO(备忘录)— 对象行为型模式
1. 意图
在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。 这样以后就可将该对象恢复到原先保存的状态。

2. 别名
Token

3. 适用性
• 必须保存一个对象在某一个时刻的 (部分)状态, 这样以后需要时它才能恢复到先前的状态。
• 如果一个用接口来让其它对象直接得到这些状态，将会暴露对象的实现细节并破坏对象的封装性。

4. 相关模式
Command(5.2): 命令可使用备忘录来为可撤消的操作维护状态。 Iterator(5.4): 如前所述备忘录可用于迭代 .

5. 使用场景
❑需要保存和恢复数据的相关状态场景。
❑提供一个可回滚（rollback）的操作；比如Word中的CTRL+Z组合键，IE浏览器中的后退按钮，文件管理器上的backspace键等。
❑需要监控的副本场景中。例如要监控一个对象的属性，但是监控又不应该作为系统的主业务来调用，它只是边缘应用，即使出现监控不准、错误报警也影响不大，因此一般的做法是备份一个主线程中的对象，然后由分析程序来分析。
❑数据库连接的事务管理就是用的备忘录模式，想想看，如果你要实现一个JDBC驱动，你怎么来实现事务？还不是用备忘录模式嘛！（redo、undo log）


## 七、OBSERVER(观察者)— 对象行为型模式
推拉模式：是指对于主题对象信息获取是推送，还是拉取

## 八、STATE(状态)— 对象行为型模式
1. 适用性
在下面的两种情况下均可使用State模式:
  • 一个对象的行为取决于它的状态,并且它必须在运行时刻根据状态改变它的行为。
  • 一个操作中含有庞大的多分支的条件语句，且这些分支依赖于该对象的状态。这个状态通常用一个或多个枚举常量表示。通常,有多个操作包含这一相同的条件结构。State模式将每一个条件分支放入一个独立的类中。这使得你可以根据对象自身的情况将对象的状态作为一个对象，这一对象可以不依赖于其他对象而独立变化。

## 九、STRATEGY(策略)— 对象行为型模式
1. 意图
定义一系列的算法,把它们一个个封装起来, 并且使它们可相互替换。本模式使得算法可独 立于使用它的客户而变化。

2. 适用性
当存在以下情况时使用 S t r a t e g y 模式
• 许多相关的类仅仅是行为有异。“策略”提供了一种用多个行为中的一个行为来配置一
个类的方法。
• 需要使用一个算法的不同变体。例如，你可能会定义一些反映不同的空间 /时间权衡的
算法。当这些变体实现为一个算法的类层次时 [ H O 8 7 ] ,可以使用策略模式。
• 算法使用客户不应该知道的数据。可使用策略模式以避免暴露复杂的、与算法相关的数
据结构。
• 一个类定义了多种行为 , 并且这些行为在这个类的操作中以多个条件语句的形式出现。
将相关的条件分支移入它们各自的 S t r a t e g y 类中以代替这些条件语句。
