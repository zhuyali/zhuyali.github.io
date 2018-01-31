---
title: CSS魔法篇之清除浮动
date: 2018-01-31 19:59:26
tags:
---

### BFC
　　BFC，全称为块级格式化上下文。对内，它是一个独立的渲染区域，只有块级框参与，它规定了内部的块级框如何布局，与这个区域的外部毫不相干；对外，它仍处在流布局中，与其它元素的布局方式无异。通常来说，我们所看到的 HTML 页面就是一个 BFC，该 BFC 由根元素创建，对页面中的所有块级框作用。块级格式化上下文的创建方式诸多，详见：[块格式化上下文](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context)。

#### BFC 的特性
- 由于 BFC 是针对内部的块级框作用的，而块级框本身就具有每个元素会换行的特性，因而可以知道，在 BFC 中，框会一个接一个地被垂直放置，它们的起点是包含块的顶部
- 两个兄弟框之间的垂直距离取决于 margin 属性，处于同一个 BFC 中相邻的块级元素会发生外边距合并
- 在 BFC 中，每一个块框的左边距会与包含块的左边接触，即使存在浮动也是如此
- 计算 BFC 的高度时，浮动元素也会参与计算
- BFC 的区域不会与浮动元素重叠

### 边界塌陷
　　在 CSS 中，有一个常见的问题就是当子元素设为浮动后，父元素的高度也随之变为 0，但这往往并不是我们想要的结果。
HTML 代码如下：
```
<div class="parent">
  <div class="child">child</div>
</div>
```
CSS 代码如下：
```
.child {
  float: left;
}
```

### 清除浮动
　　为了解决边界塌陷的问题，可以借助上述提到的*在计算 BFC 的高度时，浮动元素也会参与计算*的思想。利用 BFC 的解决方案有如下：
1. 使用 overflow 属性
   ```
   .parent {
     overflow: hidden;
   }
   ```
   通过将父元素的 overflow 属性设为非 visible 的值，即可创建一个 BFC 包含子浮动元素。
2. 浮动父元素
   ```
   .parent {
     float: left;
   }
   ```
   通过浮动父元素，能够创建新的 BFC。但是此方案的隐患是可能造成新的浮动问题。

　　除了利用 BFC 的思想之外，还有一些其它解决方案：
1. 直接设置父元素高度
   ```
   .parent {
     height: 100px;
   }
   ```
   该方法简单粗暴，适合于子元素高度固定且已知的情况。只有当子元素高度<=100时，上面的代码才能完全包含子元素。
2. 结尾处加空 div 或者空 br 标签，设置其 CSS 属性 clear
   修改 HTML 代码为：
   ```
   <div class="parent">
     <div class="child">child</div>
     <!--div class="clearfloat"></div--> //方案1：添加空 div 标签
     <!--br class="clearfloat" /--> //方案2：添加空 br 标签
   </div>
   ```
   CSS 代码为：
   ```
   .clearfloat {
     clear: both;
   }
   ```
   该方法利用了 clear 属性。根据 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/clear)，clear 属性指定一个元素是否可以在它之前的浮动元素旁边，或者必须向下移动。在本例中使用 `clear: both`使得该元素左右都不能有浮动的元素，那么它就会向下移动，来撑开父元素。
3. 使用伪元素 ::after
   ```
   .parent::after {
     display: block;
     clear: both;
     content: '';
   }
   ```
   与上面的方案类似，该元素不过是拿 ::after 伪元素替代了`<div class="clearfloat"></div>`。这里需要注意的一点是伪元素默认是行内元素，所以这里通过 display 属性将其改为块级元素。

### 参考文献
1. [详说清除浮动](http://kayosite.com/remove-floating-style-in-detail.html)
2. [常规流之块级格式化上下文（Block Formatting Contexts）](https://www.cnblogs.com/rexD/p/4597380.html)
