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
