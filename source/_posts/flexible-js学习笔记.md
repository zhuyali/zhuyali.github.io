---
title: 移动端小白的flexible.js学习笔记
date: 2018-06-22 17:20:35
tags:
---

### 预备知识
　　一直以来都没有尝试过移动端的开发，但是却对 REM 适配方案很感兴趣。最近趁着开发微信小程序的契机，顺便学习了一下手淘前端团队的可伸缩布局方案 [flexible](https://github.com/amfe/lib-flexible)。作为一个移动端开发小白，学习适配方案遇到的最大的困难可能是各种各样的换算关系和公式，网上大部分文章都会介绍非常多的专业词汇，我认为有些对于前端开发者来说可以是无感知的，所以这里仅仅介绍一些掌握 REM 适配方案的必要知识。

##### REM
　　REM 适配方案的核心，不必多说，就是 REM。REM 是什么呢？没你想的那么复杂，它就是我们见过甚至平时都会用到的 CSS 相对长度单位，官方解释是相对于根元素 font-size 计算值的倍数。也就是说，如果设置根元素 font-size: 12px; 那么 1rem 的值为 12px，2rem 的值为 24px，以此类推。它是一个相对单位，意味着只要我们改变了根元素 font-size 的值，那么所有使用 rem 单位的地方值的大小都会相应的改变，emmm...这么说来好像确实是实现适配方案的一个不错的选择哦~

##### 设备像素比 dpr
　　dpr = 物理像素 / 设备独立像素。物理像素是显示器上一个个的点，我们平常所说的 1920 × 1080 像素分辨率就是用的物理像素。设备独立像素是与设备无关的逻辑像素，可以理解为程序中可以使用的像素，CSS 中用到的像素就是设备独立像素。
    <img src="https://haitao.nos.netease.com/73d06363-076e-40b0-ae7b-b0be8e851436.jpg" width="70%;"/>
##### 图片缩放比例
　　假设我们有一个 width: 200px; height: 300px; 的 img 标签，在 dpr 为 1:1 的屏幕设备上，它所对应的物理像素是 200 × 300 个；而在 dpr 为 2:1 的屏幕上，它所对应的物理像素是 200 × 300 × 4 个，也就是说此时的物理像素宽为 200px × dpr，高为 300px × dpr。如果图片大小为 200px × 300px，在 dpr 为 1 的屏幕上显示正常，可是到了 dpr 为 2 的屏幕上，就无法高清显示，原因是因为位图像素（注：一个位图像素是栅格图像（如：png、jpg、gif等）最小的数据单元。它包含当前像素的显示位置、颜色值、透明度等信息，是不可以进一步分割的）不可拆分，所以就只能就近取色，从而导致图片模糊。解决这个问题的一个比较好的方案就是提供 dpr 倍图片，接着上面的例子说，就是需要提供 400px × 600px 的图片。理解了这一点后，就明白为什么视觉稿的画布大小要 ×2 了。

### 源码解读
```
(function flexible (window, document) {
  var docEl = document.documentElement
  // 获取设备像素比 dpr，默认为 1
  var dpr = window.devicePixelRatio || 1

  // adjust body font size
  // 这里对于 body 的字体大小的设置持疑问态度
  function setBodyFontSize () {
    if (document.body) {
      document.body.style.fontSize = (12 * dpr) + 'px'
    }
    else {
      document.addEventListener('DOMContentLoaded', setBodyFontSize)
    }
  }
  setBodyFontSize();

  // set 1rem = viewWidth / 10
  // 设置根元素的字体大小为视口的 1/10
  function setRemUnit () {
    var rem = docEl.clientWidth / 10
    docEl.style.fontSize = rem + 'px'
  }

  setRemUnit()

  // reset rem unit on page resize
  // 当窗口大小变化时，调用 setRemUnit
  window.addEventListener('resize', setRemUnit)
  // 每次加载页面时触发
  window.addEventListener('pageshow', function (e) {
    // 从浏览器缓存中读取页面的话，调用 setRemUnit
    if (e.persisted) {
      setRemUnit()
    }
  })

  // detect 0.5px supports
  // 为什么不考虑设备像素比为 1 的情况，我的理解如下：
  // 如果设备像素比为 1，那么 0.5px 对应于物理设备的 0.5 像素，而物理像素应该是以 1px 为最小单位的
  // 当设备像素比 >= 2 时，才会检测是否支持 0.5px
  if (dpr >= 2) {
    var fakeBody = document.createElement('body')
    var testElement = document.createElement('div')
    testElement.style.border = '.5px solid transparent'
    fakeBody.appendChild(testElement)
    docEl.appendChild(fakeBody)
    // 如果设备支持 0.5px，那么给 docEl 添加一个 hairlines 类
    if (testElement.offsetHeight === 1) {
      docEl.classList.add('hairlines')
    }
    docEl.removeChild(fakeBody)
  }
}(window, document))
```
　　根据官方 README，这套 lib-flexible 的方案已经不再维护，建议大家使用 viewport 来代替此方案。

### 极致一像素
　　根据我们之前学到的知识，当我们在 CSS 中设置长度单位 1px 时，对应的物理像素是由 dpr 决定的。然而，设计师给我们的设计稿要求的一像素往往是真正的物理一像素，那么如何实现这种“极致一像素”呢？这时候 meta 标签就可以为我们解决这个烦恼。可以设置 meta 标签为：
```
<meta name="viewport" content="initial-scale=1/dpr, maximum-scale=1/dpr, minimum-scale=1/dpr, user-scalable=no">
```
这时候我们在 CSS 中设置 1px 就是对应于真实的物理一像素。但是，使用这种方式带来的负面影响就是会导致我们的 REM 适配方案失效，以下代码是受影响的部分及其改进方案；
```
// 这里设置了 body 的 fontSize，但是却因为缩放的原因，使其变成了 12px
document.body.style.fontSize = (12 * dpr) + 'px'
// 这里同样会受到缩放的影响
docEl.style.fontSize = rem + 'px'
// 解决方案就是设置 body 的 fontSize 为 12 * dpr * dpr，docEl 的 fontSize 为 rem * dpr
```

### 参考资料
1. [移动端高清、多屏适配方案](https://div.io/topic/1092)
2. [谈谈 rem 与 vw -- rem](https://www.jianshu.com/p/1a9b5d48afa2)