---
title: CSS魔法篇之元素居中
date: 2017-11-15 18:41:31
tags:
---

### 水平居中

#### inline 或者 inline-* 元素

其父元素为块级元素，使用 text-align: center; 属性居中。

HTML代码如下：
```
<div class="parent">
  <span class="child">child</span>
</div>
```

CSS代码如下：
```
.parent {
  text-align: center;
}
```

#### 块级元素

一个块级元素包含在另一个块级元素中，通过设定子块级元素的 width，margin-left: auto; 和 margin-right: auto; 来进行元素居中。原理是：块级元素所有的水平属性加在一起必须为元素包含块（这里就是指父元素）的宽度。默认情况下元素宽度为 auto，margin 为 0，块级子元素的宽度会充满其父元素。当设置 margin-left 和 margin-right 为 auto 时，再加上默认的 width 为 auto，此时元素与默认情况下表现一致。当同时设定了元素宽度为特定值和左右外边距为 auto 时，元素的左右外边距会设置为相同的长度以填充满父元素的宽度，就会使得元素在其父元素居中。（水平属性有：margin-left，border-left，padding-left，width，padding-right，border-right，margin-right）

HTML代码如下：
```
<div class="parent">
  <div class="child">child</div>
</div>
```

CSS代码如下：
```
.child {
  width: 50px;
  margin: 0 auto;
}
```

#### 多个块级元素

HTML代码如下：
```
<div class="parent">
  <div class="child">child1</div>
  <div class="child">child2</div>
  <div class="child">child3</div>
</div>
```

- ##### 多个块级元素在一行居中

  - 通过调节 display 进行居中

    大家都知道，默认情况下块级元素的前后都是要换行的。这里我们要将多个块级元素放在一行，所以要设置块级元素的 display: inline-block; 然后通过设置其父元素的 text-align: center; 就可以完成居中。

    CSS代码如下：
    ```
    .parent {
      text-align: center;
    }
    .child {
      display: inline-block;
    }
    ```

  - 使用 flex 布局
    
    CSS代码如下：
    ```
    .parent {
      display: flex;
      justify-content: center;
    }
    ```

- ##### 多个块级元素在不同行分别居中

  与上述块级元素的缩进同理

    CSS代码如下：
    ```
    .child {
      width: 50px;
      margin: 0 auto;
    }
    ```

### 垂直居中

#### inline 或者 inline-* 元素

将元素的行高设置为与父元素的高度一致。原理是 lineheight - fontsize 能够得到行间距，行间距会分为两半分别应用到内容区的顶部和底部，从而使得元素居中。

HTML代码如下：
```
<div class="parent">
  <span class="child">child</span>
</div>
```

CSS代码如下：
```
.parent {
  width: 100px;
  height: 100px;
  background: red;
  
}
.child {
  line-height: 100px;
}
```

#### 块级元素

HTML代码如下：
```
<div class="parent">
  <div class="child">child</div>
</div>
```

- 元素高度已知

  将父元素设为相对定位，将子元素设为绝对定位，这是因为绝对定位元素的包含块是最近的 position 值不为 static 的祖先元素。设置完元素的定位之后，就可以保证子元素以父元素为基准进行定位的，此时将子元素下移父元素高度的 50% 再上移自身高度的一半，就能达到居中的效果。

  CSS代码如下：
  ```
  .parent {
    position: relative;
    height: 100px;
  }
  .child {
    position: absolute;;
    height: 30px;
    top: 50%;
    margin-top: -15px;
  }
  ```

- 元素高度未知

  将父元素设为相对定位，将子元素设为绝对定位，这是因为绝对定位元素的包含块是最近的 position 值不为 static 的祖先元素。设置完元素的定位之后，就可以保证子元素以父元素为基准进行定位的，此时将子元素下移父元素高度的 50% 再上移自身高度的 50%，就能达到居中的效果。

  CSS代码如下：
  ```
  .parent {
    position: relative;
    height: 100px;
  }
  .child {
    position: absolute;;
    height: 30px;
    top: 50%;
    transform: translateY(-50%);
  }
  ```

- 使用 flex 布局

  CSS代码如下：
  ```
  .parent {
    display: flex;
    height: 100px;
    align-items: center;
  }
  ```

### 水平居中 + 垂直居中

HTML代码如下：
```
<div class="parent">
  <div class="child">child</div>
</div>
```

- 元素宽度和高度都已知

  将元素相对父元素向下移动父元素高度的 50%，向右移动父元素宽度的 50%，然后再将元素向上移自身高度的一半（本例中是 15px），向左移自身宽度的一半（本例中是 25px）。

  CSS代码如下：
  ```
  .parent {
    position: relative;
    height: 100px;
  }
  .child {
    position: absolute;
    width: 50px;
    height: 30px;
    left: 50%;
    top: 50%;
    margin-top: -15px;
    margin-left: -25px;
  }
```

- 元素宽度或高度未知

  将元素相对父元素向下移动父元素高度的 50%，向右移动父元素宽度的 50%，然后再将元素向上移自身高度的 50%，向左移自身宽度的 50%。

  CSS代码如下：
  ```
  .parent {
    position: relative;
    height: 100px;
  }
  .child {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
  }
  ```

- 使用 flex 布局

  CSS代码如下：
  ```
  .parent {
    height: 100px;
    width: 100px;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  ```