---
layout: post
title:  "开源许可扫盲"
date:   2016-01-22 22:11:52
categories: license, open source
comments: true
thread: 20160122221152555

---

开源许可(open source license), 也就是我们经常说的*开源协议*(非专业叫法,下面都会以许可称呼),虽然有很多耳熟能详的英文缩写,然而这些许可有哪些约束,能赋予我们哪些权利呢？

下面我们就细数一下几种开源许可:
 BSD、Apache Licence、GPL V2 、GPL V3 、LGPL、MIT

 **基本概念**

*Contributors 和 Recipients*
 Contributors（贡献者） ——指的是对某个开源软件或项目提供了代码（包括最初的或者修改过）的人或实体（退队、公司、组织等）。
 按照贡献的先后可分为"创始人"（an initial Contributor）和"参与者"（subsequent Contributors）。

 Recipients（获取者） ——指的是开源软件或项目的使用者。
 显然，subsequent Contributors也属于Recipients之列。

 *Source Code 和 Object Code*
 Source Code ——指的是由各种语言写成的源代码 。
 Object Code ——指的是Source Code经过编译后，生成的类似“类库”一样的，提供了各种接口供他人使用的目标代码 （就如,DLL、JAR等）。

 *Derivative Module 和 Separate Module*
 Derivative Module（衍生模块） ——指的是，依托或包含“最初的”或者“从别人处获取的”开源代码而产生的代码，是对“源代码模块”的增强、改善和延续。
 Separate Module（独立模块） ——指的是，参考或借助“源代码”开发出来的独立的，不包含、不依赖于原“源代码模块”的功能模块。


---

## 一. BSD

### Brief Introduction
BSD是Berkly Software Distribution.

目前分为 [BSD 3-Clause](http://www.opensource.org/licenses/BSD-3-Clause) 和[BSD 2-Clause](https://opensource.org/licenses/BSD-2-Clause) 。顾名思义，3-Clause包含3个条款，2-Clause只有两个。


### Aim at
BSD 鼓励代码共享的同时，要求尊重代码作者的著作权

### CAN DO

- 自由的使用
- 修改源代码，
- 修改后的代码作为开源或者专有软件再发布。

### RESTRICT(以下味BSD 3, BSD 2不包含第三条)
- 再发布的产品中包含源代码，则在源代码中必须带有原来代码中的BSD协议。
- 如果再发布的只是二进制类库/软件，则需要在类库/软件的文档和版权声明中包含原来代码中的BSD协议。
- 不可以用开源代码的作者/机构名字和原来产品的名字做市场推广。

### Scenario
BSD由于允许使用者修改和重新发布代码，也允许使用或在BSD代码上开发商业软件发布和销售，因此是对 商业集成很友好的协议。
而很多的公司企业在选用开源产品的时候都首选BSD协议，因为可以完全控制这些第三方的代码，在必要的时候可以修改或者二次开发。

---

## 二. [Apache](http://www.opensource.org/licenses/Apache-2.0)

### Brief Introduction
Apache Licence 2.0

- Apache License, Version 2.0
- Apache License, Version 1.1
- Apache License, Version 1.0

Apache Licence是著名的非盈利开源组织Apache采用的协议。

### Aim at
鼓励代码共享和尊重原作者的著作权

### CAN DO

- 自由的使用
- 修改源代码，
- 修改后的代码作为开源或者专有软件再发布。


### RESTRICT
- 需要给代码的用户一份Apache Licence
- 如果你修改了代码，需要在被修改的文件中说明。
- 在延伸的代码中（修改和有源代码衍生的代码中）需要带有原来代码中的协议，商标，专利声明和其他原来作者规定需要包含的说明。
- 如果再发布的产品中包含一个Notice文件，则在Notice文件中需要带有Apache Licence。你可以在Notice中增加自己的许可，但不可以表现为对Apache Licence构成更改。

### Scenario
Apache Licence也是对商业应用友好的许可。使用者也可以在需要的时候修改代码来满足需要并作为开源或商业产品发布/销售。

---

## 三. [GPL](http://www.opensource.org/licenses/gpl-license)

### Brief Introduction
GNU General Public License

- GPL
- V2
- V3

GPL v2的相关规定：
只要这种修改文本在整体上或者其某个部分来源于遵循GPL的程序，该修改文本的整体就必须按照GPL流通，不仅该修改文本的源码必须向社会公开，
而且对于这种修改文本的流通不准许附加修改者自己作出的限制。

GPL v3的修订草案中，不仅要求用户公布修改的源代码，还要求公布相关硬件.

### Aim at
GPL的出发点是代码的开源/免费使用和引用/修改/衍生代码的开源/免费使用，但不允许修改后和衍生的代码做为闭源的商业软件发布和销售。

### CAN DO

- 自由的使用
- 修改源代码，
- 修改后的代码作为开源/免费再使用。

### RESTRICT
- 只要在一个软件中使用(”使用”指类库引用，修改后的代码或者衍生代码)GPL 协议的产品，则该软件产品必须也采用GPL协议，既必须也是开源和免费。
- 不允许修改后和衍生的代码做为闭源的商业软件发布和销售

### Scenario
由于GPL严格要求使用了GPL类库的软件产品必须使用GPL协议，对于使用GPL协议的开源代码，商业软件或者对代码有保密要求的部门就不适合集成/采用作为类库和二次开发的基础。

---

## 四. [LGPL](http://www.opensource.org/licenses/lgpl-license)

### Brief Introduction
GNU Lesser General Public License

LGPL 是GPL的一个为主要为类库使用设计的开源协议。

### Aim at
LGPL 允许商业软件通过类库引用（link）方式使用LGPL类库而不需要开源商业软件的代码。

### CAN DO

- 自由的使用
- 修改源代码，
- 修改后的代码作为开源或者专有软件再发布。

### RESTRICT
- 但是如果修改LGPL协议的代码或者衍生，则所有修改的代码，涉及修改部分的额外代码和衍生的代码都必须采用LGPL协议

### Scenario
LGPL协议的开源 代码很适合作为第三方类库被商业软件引用，但不适合希望以LGPL协议代码为基础，通过修改和衍生的方式做二次开发的商业软件采用。

---

## 五. [MIT](http://www.opensource.org/licenses/MIT)

### Brief Introduction
MIT协议又称麻省理工学院许可证，最初由麻省理工学院开发. MIT是和BSD一样宽范的许可协议，作者只想保留版权。

### Aim at
作者只想保留版权，而无任何其它的限制。

### CAN DO

- 被授权人有权利使用、复制、修改、合并、出版发行、散布、再授权及贩售软件及软件的副本。
- 被授权人可根据程式的需要修改授权条款为适当的内容。

### RESTRICT
- 必须在你的发行版里包含原许可协议的声明，无论你是以二进制发布的还是以源代码发布的

---

## 总结
> - BSD2：修改版本必须保持其原始版权声明。
- BSD3：修改版本必须保持其原始版权声明。未经许可不得使用原作者或公司的名字做宣传。
- Apache：修改版本必须保持其原始版权声明；修改过的文件要标明改动。
- GPL：只要你用了任何该协议的库、甚至是一段代码，那么你的整个程序，不管以何种方式链接，都必须全部使用GPL协议、并遵循该协议开源。商业软件公司一般禁用GPL代码，但可以使用GPL的可执行文件和应用程序。
- LGPL：就是GPL针对动态链接库放松要求了的版本，即允许非LGPL的代码动态链接到LGPL的模块。注意：不可以静态链接，否则你的代码也必须用LGPL协议开源。
- MIT：修改版本必须保持其原始版权声明。


版权有利有弊,总体上讲还是利大于弊.

- 弊: 阻碍技术的传播，或增加技术传播的成本。
- 利: 尊重原创者, 提升行业竞争力.

在工作当中我们常常会采用／引用一些开源框架或第三方类库，大大的缩短我们的开发周期，甚至提升软件的性能。本章内容阅读完毕后，希望大家在需要引用的场景下做出最正确的选择。


---

## 参考资料
* [百度百科](http://baike.baidu.com/link?url=tNfyS5cVC9Syf-AsDth8mtzVhVKMIdpAhh_d2LUjn5_fAj2Z6a0r_PZCrqP8RwsSzC3lty2JS_1T9TEKsPQkNq)
* [常见开源协议大白话说明](http://blog.csdn.net/nightmare/article/details/12405109)
