---
layout: post
title:  "第二章 设计模式-实例"
date:   2018-10-26 22:45:52
categories: 设计模式
published: true
comments: true
thread: 20181026224555555
---
第二章 实例-设计一个文档编辑器
---
# 考虑的几个方面
  - 文档结构
    - 所有的编辑、格式安排、显示和文本分析都涉及到这种表示，存储态结构。
  - 格式化
    - 样将文本和图形安排到行和列上的?哪些对象负责执行不同的格式策略?这些策略又是怎样和内部表述相互作用，运行态结构以及生成运行态结构的对象。
  - 修饰用户界面
    - Lexi的用户界面包括滚动条、边界和用来修饰WYSIWYG文档界面的阴影。加些调色和修饰的结构和交互。
  - 支持多种视感标准
    - Lexi应不需作较大修改就能适应不同的视感标准。一种格式化的多套修饰。
  - 支持多种窗口系统
    - 不同的视感标准通常是在不同的窗口系统上实现的。Lexi的设计应尽可能的独立于窗口系统。多种格式化结构。
  - 用户操作
    - 用户通过不同的用户界面控制Lexi，包括按钮和下拉菜单。这些界面对应的功能分散在整个应用对象中。这里的难点在于提供一个统一的机制，既可以访问这些分散的功能，又可以对操作进行撤消(undo)。命令模式p结构
  - 拼写检查和连字符
    -Lexi是怎样支持像检查拼写错误和决定连字符的连字点这样的分析操作的?当我们不得不添加一个新的分析操作时，我们怎样尽量少修改相关的类? 处理流程应该是一致的，具体如何check行为应该是不同的，策略模式。

# 文档结构
设计一种结构能够表示不同的页面元素。类似于递归组合结构。一种元素可以容纳其他元素。
抽象为 Glyph 类：属性；动作；是否容器；位置信息；

# 格式化
格式化算法应该独立于存储态文档结构，经过格式化得到特殊的物理结构。
格式化算法：各个Glyph之间的关系，运行态结构;

@startuml
Composition   o--    Glyph : > children
Composition   o--   Compositor
Compositor   <|--   SimpileCompositor
Compositor   <|--   ArrayCompositor
Compositor   <|--   TxtCompositor

interface Glyph {
void Insert(Glyph, int i)
}

class Composition {
void Insert(Glyph, g, int i)
}

interface Compositor {
void compose()
}

interface Compositor {
setComposition(Composition )
}
@enduml

模式的主要参与者是Strategy对象(这些对象中封装了不同的算法)和它们的操作环境。其实Compositor就是Strategy。它们封装了不同的格式算法。Composition就是Compositor策略的环境。

# 修饰用户界面

@startuml
MonoGlyph   o--    Glyph
MonoGlyph   <|--   Border
MonoGlyph   <|--   Scroller

interface Glyph {
void Draw(Window)
}

interface MonoGlyph {
Glyph component
void Draw(Window)
}

interface Border {
void Draw(Window)
void DrawBorder(Window)
}

interface Scroller {
void Draw(Window)
void DrawScroller(Window)
}
@enduml


# 支持多种视感标准
支持多种视感障碍在于不同视感标准的差异性。目标是符合多个已存在的视觉标准，并且在新标准出现时很容易增加对新标准的支持。
1. 对象的定义
  一、抽象窗口组件
  定义窗口组件的抽象集合，包含通用的操作
  二、具体实现子类
  抽象组件对应不同视感标准的具体实现子类
2. 对象创建
根据视感标准创建出一组窗口组件。工厂类和产品类。

@startuml
GUIFactory   <|--   MotiFactory
GUIFactory   <|--   PMFactory
GUIFactory   <|--   MacFactory


interface GUIFactory {
void CreateScrollBar()
void CreateButton()
void CreateMenu()
}

class MotiFactory {
void CreateScrollBar()
void CreateButton()
void CreateMenu()
}

class PMFactory {
void CreateScrollBar()
void CreateButton()
void CreateMenu()
}

class MacFactory {
void CreateScrollBar()
void CreateButton()
void CreateMenu()
}
@enduml

# 支持多种窗口系统
视感只是众多移植问题之一。另一个移植问题就是所运行的窗口环境。
乍一看和视感解决方式一样也可以应用抽象工厂的模式。

Window封装了窗口要各个系统都要做的一些事情：
- 提供画几何图形的操作
- 能变成图标或还原成窗口
- 改变自己的大小
- 根据需要画出窗口内容

Window类的窗口功能必须跨越不同的窗口系统。
1. 功能交集。问题在于Window接口在能力上只类似于一个最小功能的窗口系统，对一些即使是大多数窗口系统都支持的高级特征，我们也无法利用。
2. 功能并集。创建一个合并了所有存在系统的功能的接口。但是这样的接口势必规模巨大，并且存在不一致的地方。此外，当某个厂商修改它的窗口系统时，我们不得不修改这个接口和Lexi，因为Lexi依赖于它。


应该对变化的概念抽象出来单独演化，并且该部分参与者是窗口系统本身，不是程序员。
