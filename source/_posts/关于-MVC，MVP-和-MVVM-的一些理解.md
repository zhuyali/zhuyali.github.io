---
title: 关于 MVC，MVP 和 MVVM 的一些理解
date: 2017-11-27 14:50:57
tags:
---

MVC，是我（自以为）很熟悉的一个编程模式，因为从大学以来搭建的各个项目来看，几乎每个项目都使用了 MVC 的思想。然而，在前一段时间的研究生课程中，旁边的同学问我 “MVC 是什么？” 的问题时，我竟一时语塞，答案也只能简单的介绍 M-V-C 三个字母分别代表什么，而不能做出更加深层次的回答。所以，就有了这篇文章。不过，这里的理解整体上还是比较浅显，大神们可以自动略过...

### 什么是 MVC?

M-Model，V-View，C-Controller 分别代表 模型-视图-控制器。

- 模型，负责程序中的数据逻辑（既包括数据，也包括处理数据的逻辑），这是整个应用程序的核心。当我们面对一个问题时，对问题的本质的描述就是模型，而解决问题就是给问题建立模型，模型建立好了，问题的处理自然也会变得清晰。

- 视图，是用户能够看到并与之交互的界面，我们每天浏览的网页就是视图。视图的职责比较单一，就是负责展示，至于展示的内容，还是由模型来决定的（所以说模型是核心嘛）。

- 控制器，是模型和视图之间的桥梁。早起的 MVC 中，视图通过控制器来通知模型变更状态，模型履行了自己的职责后再通知视图来改变状态，简单来说，控制器和视图分别是模型的输入和输出。

早期的 MVC 模式如图：

![](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015020105.png)

### 什么是 MVP?

MVP 是 MVC 的一个变种。从上面的图中我们可以看出，在 MVC 中，Model 不依赖于 View，但是 View 是依赖于 Model 的，切断 Model 与 View 的联系，能够使得三者的关系更加清晰，职责也更加单一，因而这里对 Controller 做了改进。Controller 全权负责 Model 与 View 之间的通信而无需它们之间单独通信：Controller 将消息传递给 Model，Model 处理完数据后将数据的变化通知给 Controller，再由Controller 来通知 View 来变化视图，简单来说，它们之间的关系为  View <-> Controller <-> Model，以上说的 Controller 就是 MVP 中的 P-Presenter。这好像才是平时我熟悉的 MVC 模式，如图：

![](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015020109.png)

### 什么是 MVVM?

MVVM 也是 MVC 的一个变种。MVP 中的 P 封装了 Model 和 View 之间的所有通信，那么能不能把这个通信自动化？也就是说，当 Model 变化 时，View 能够识别变化做出相应改变；当 View 变化时，Model 也能够识别变化做出相应改变。MVVM 就解决了这个问题。MVVM 与 MVP 十分相似，除了你需要为 View 量身定制一个 model，这个 model 就是 ViewModel，它实现了 Model 与 View 之间的双向数据绑定，从而能够达到 View 与 Model 之间的自动双向同步。MVVM 结构与 MVP 类似：

![](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015020110.png)


### 参考文献：
1. [MVC，MVP 和 MVVM 的图示 - 阮一峰](http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html)
2. [从 Script 到 Code Blocks、Code Behind 到 MVC、MVP、MVVM](http://www.cnblogs.com/indream/p/3602348.html)
3. [mvc、mvp、mvvm的架构简单解读](https://zhuanlan.zhihu.com/p/26287306)