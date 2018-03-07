---
title: CSS魔法篇之布局实践
date: 2018-03-07 16:50:23
tags:
---

### 两列布局
　　HTML 代码如下所示(假设它们的高度是确定的)：
```
<div class="parent">
  <div class="aside"></div>
  <div class="content"></div>
</div>
```

#### 一侧定宽，一侧自适应
　　下面的示例是左侧定宽，右侧自适应的情况。
1. 方式一：使用 flex 布局
```
.parent {
  display: flex;
}
.aside {
  width: 100px;
}
.content {
  flex: 1;
}
```
2. 方式二：使用视口单位 vw
```
.aside {
  width: 100px;
  float: left;
}
.content {
  float: left;
  width: calc(100vw - 100px);
}
```
3. 方式三：利用 BFC 特性第三条和第五条，详见[CSS魔法篇之清除浮动](https://zhuyali.github.io/2018/01/31/CSS%E9%AD%94%E6%B3%95%E7%AF%87%E4%B9%8B%E6%B8%85%E9%99%A4%E6%B5%AE%E5%8A%A8/)
```
.aside {
  width: 100px;
  float: left;
}
.content {
  overflow: hidden;
}
```
4. 利用 width 默认为 auto 结合 margin
```
.aside {
  width: 100px;
  float: left;
}
.content {
  margin-left: 100px;
}
```
5. 使用绝对定位
```
.parent {
  position: relative;
}
.aside {
  width: 100px;
}
.content {
  position: absolute;
  left: 100px;
  right: 0;
  top: 0;
}
```

### 三列布局
　　HTML 代码如下(假设它们的高度是确定的)：
```
<div class="parent">
  <div class="left"></div>
  <div class="content"></div>
  <div class="right"></div>
</div>
```

#### 两侧定宽，中间自适应
1. 使用 flex 布局
```
.parent {
  display: flex;
}
.left {
  width: 100px;
}
.content {
  flex: 1;
}
.right {
  width: 100px;
}
```
2. 绝对定位和浮动
```
.parent {
  position: relative;
}
.left {
  float: left;
  width: 100px;
}
.content {
  position: absolute;
  left: 100px;
  right: 100px;
  top: 0;
}
.right {
  float: right;
  width: 100px;
}
```
3. 利用负外边距和浮动
```
.parent {
  overflow-x: hidden;
}
.left {
  float: left;
  width: 100px;
}
.content {
  float: left;
  width: 100%;
  margin-right: -200px;
}
.right {
  float: right;
  width: 100px;
}
```

#### 三列等分
1. 等分百分比
```
.parent {
  font-size: 0; // 解决 inline-block 的间隙问题
}
.left, .content, .right {
  display: inline-block;
  width: 33.333%;
}
或者
.left, .content, .right {
  float: left;
  width: 33.333%;
}
```
2. 使用 flex 布局
```
.parent {
  display: flex;
}
.left, .content, .right {
  flex: 1;
}
```

### 九宫格布局
　　HTML 代码如下：
```
<div class="parent">
  <div class="child"></div><div class="child"></div><div class="child"></div>
  <div class="child"></div><div class="child"></div><div class="child"></div>
  <div class="child"></div><div class="child"></div><div class="child"></div>
</div>
```
1. 使用 flex 布局
```
.parent {
  display: flex;
  flex-wrap: wrap;
}
.child {
  width: 33.333%;
  height:33.333%;
}
```
2. 使用浮动实现
```
.child {
  float: left;
  width: 33.333%;
  height:33.333%;
}
```