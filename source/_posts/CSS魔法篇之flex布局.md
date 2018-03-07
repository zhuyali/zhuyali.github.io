---
title: CSS魔法篇之flex布局
date: 2018-03-07 11:40:26
tags:
---

　　Flex 基础布局可以参考阮老师的文章，写的非常的详尽，并且配备了大量的示例。这里主要聊一聊 flex 的一些注意点。
### 轴对齐
　　在接下来的描述中，默认主轴为水平轴，交叉轴为垂直轴，并且书写模式为从左到右。与主轴对齐相关的属性是 justify-content，该属性设置于 flex 容器上。与交叉轴对齐相关的属性有 align-items，align-content 和 align-self，其中前两个设置于 flex 容器上，最后一个设置于 flex 项目上。justify-content 和 align-content 是对应关系，前者对应于主轴每列元素间的空间分配，后者对应于交叉轴每行元素间的空间分配，它们可取的属性值也基本相同，包括 flex-start(轴起始位置)，flex-end(轴终止位置)，center(居中)，space-between(项目之间平均留白)，space-around(项目前后平均留白)，还有一个属性值 stretch 是 align-content 特有的。
　　其实不难发现，justify-content 是将 flex 项目当成一个组来做对齐处理，那么如果单个 flex 项目或者多个 flex 项目想和其它 flex 项目分离的话，可以通过 margin 来实现。考虑下面图片的情况，我有三个项目在左边两个在右边，就可以通过给项目 d 设置 `margin-left: auto;`来实现该效果。它利用的原理是水平方向上的 auto margin 会尽量占据多余的空间以撑开包含块的宽度。
![](https://mdn.mozillademos.org/files/15633/align7.png)

### 冲突的元素设置
1. 在 flex 项目上设置`float: left/right;`没有效果
2. 在 flex 项目上设置`display: inline-block;`没有效果
3. 在 flex 项目上设置`display: table-;`没有效果

### 项目尺寸和伸缩性
　　flex 项目最终展现出的尺寸大小与三个属性有关：flex-grow，flex-shrink 和 flex-basis。flex 项目之所以能够自由的伸缩依赖于 flex 容器的正负自由空间，可以看下图理解：
![](https://mdn.mozillademos.org/files/15654/Basics7.png)　　在上图中，容器为 500 像素宽，每个项目为 100 像素宽，剩余了 200 像素的 positive free space。
![](https://mdn.mozillademos.org/files/15655/ratios1.png)　　在上图中，容器为 500 像素宽，每个项目为 200 像素宽，那么项目的总宽度超过了容器宽度，就有了 100 像素的 negative free space。
　　flex-grow 定义了项目如何扩张，它会按照比例分配总的可用空间给各个项目。在以下两张图代表的例子中，设置各个项目的`flex-basis: auto;`让它们自动调整大小，此时总的可用空间就是容器的宽度减去各个项目的宽度，也就是图中的阴影区域。如果我们相对这篇阴影区域在每个项目之间进行平均分配，就设置每个项目的`flex-grow: 1;`即可。其实对于 flex-grow 的设置不一定非要是整数，设置该值的主要用途是用作比例，因而可以任意设置该值，只要保证它们的比例为 1：1：1 即可。（最后在强调一点，flex-grow 的比例并不是项目的大小比例，而是对于剩余的空间分配的比例）
![](https://mdn.mozillademos.org/files/15656/ratios2.png)![](https://mdn.mozillademos.org/files/15657/ratios3.png)　　flex-shrink 定义了项目如何收缩，用途类似于 flex-grow，这里不详细说明。
　　flex-basis 定义了项目的基础尺寸，它的初始值为 auto，如果你给项目设置了宽度，那么基础尺寸就为该宽度；如果项目大小没有设置，那么基础尺寸就会恰好包含其内容。空间分配时，如果你想要完全忽略该项目的尺寸，可以设置该属性值为 0，这相当于告诉 flexbox 该项目的所有空间都可以抢占，并按比例分享。

### 参考资料
1. [Flex 布局教程：语法篇](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)
2. [MDN Flex](https://developer.mozilla.org/en-US/docs/Glossary/Flex)