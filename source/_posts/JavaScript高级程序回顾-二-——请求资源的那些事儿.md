---
title: JavaScript高级程序回顾(二)——请求资源的那些事儿
date: 2018-01-09 11:54:15
tags:
---

### 以不变应万变的 XMLHttpRequest
　　在这个横行使用 Ajax 的时代，很多人对待 XMLHttpRequest(简称 XHR)的态度都是：“哦，我知道这个东西”，“那你来使用它简单实现一下 Ajax 吧”，“emmm...”，不了了之。好，废话不多说，下面我们一起揭开 XHR 并不神秘的面纱吧。
　　XHR 为向服务端发送请求和解析服务端响应提供了流畅的接口，能够以异步方式从服务端取得更多信息，是当今一系列异步请求的鼻祖。它最初是由微软发明的，后来被各大浏览器原生支持。

#### XHR 的基本用法
```
var xhr = new XMLHttpRequest();//创建 XHR 对象
xhr.open('get', 'example.txt', false);//启动 XHR 请求，此时并没有真正发送请求
xhr.send(null);//发送请求
```
　　上面的代码就完成了一个基本的 HTTP 请求，这里白话解释一下上面做了什么：使用 GET 方法，以同步的方式获取“当前页面/example.txt”资源。API 不深入解释，可以参考 [MDN 相关文档](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)。在我的解释中，不知大家有没有关注到“同步”两个字，我想这是我强调的关键：open 方法的第三个布尔值代表是否已同步的方式获取资源，有同步当然就有异步啦，这是令人激动的地方，所以说 XHR 是异步请求的鼻祖嘛，因为浏览器给我们原生提供了异步请求的方法，后续无论做多少封装，也万变不离其宗。

### Ajax
　　Ajax 的全称是 Asynchroronous JavaScript + XML(异步的 JavaScript 和 XML)，是一种无需重新加载整个网页的情况下，能够部分更新网页的技术。它基于 XHR，下面原生实现一版阉割版 Ajax ~
```
function ajax(options) {
  options = options || {};
  options.type = (options.type || 'GET').toUpperCase();
  var params = formatParams(options.data);
  var xhr = new XMLHttpRequest();
  xhr.onreadystatechange = function() {
    if (xhr.readyState === 4 && xhr.status >= 200 && xhr.status < 300 || xhr.status === 304) {
      options.success && options.success(xhr.responseText, xhr.responseXML);
    } else {
      options.fail && options.fail(status);
    }
  };
  if (options.type === 'GET') {
    xhr.open('get', `${options.url}?${params}`, true);
    xhr.send(null);
  } else if (options.type === 'POST') {
    xhr.open('post', options.url, true);
    xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');//模仿表单提交
    xhr.send(params);
  }
}

//格式化参数
function formatParams(data) {
  let result = [];
  for (let key in data) {
    result.push(`${encodeURIComponent(key)}=${encodeURIComponent(data[key])}`)
  }
  return result.join('&');
}
```
　　调用方式为：
```
ajax({
  url: '/test',
  type: 'POST',
  data: { name: 'Julia', age: 23 },
  success: function (responseText, responseXML) {
    //此处放成功后执行的代码
  },
  fail: function (status) {
    //此处放失败后执行的代码
  }
});
```

### 新兴的 Fetch
　　在网上看到一句话“Ajax 已死，fetch 永生”，颇有标题党的意思。但是仔细想了想，不无道理。Ajax 过去再怎么辉煌，也离不开“封装”二字，尽管对 XHR 的封装已经做得非常好。而新兴的 fetch 既是浏览器的亲生骨肉，又能提供类似于 Ajax 的实现。所以，Ajax 被替代是指日可待的事情了。不过这里我就不太多赘述关于 fetch 的 API 啦（博主赶紧跑去学习学习...）。

### 跨域技术
　　跨域实在是很让人头疼的一件事，好在现在有不少流行的跨域技术。以我的角度来看，我更愿意把各样的跨域技术分为两类：需要服务端支持的和不需要服务端支持的。为什么这么分类呢？是因为跨域也要分跨谁的域，跨自家公司的其它域的时候，如果能在服务端做点手脚能省不少事；要是想跨别人的域呢，那就只能前端在浏览器里做点手脚啦，而且还不一定能得到自己想要的跨域结果~

#### 需要服务端支持的跨域

1. 跨域资源共享(CORS)
　　CORS 的基本思想是：使用自定义的 HTTP 头部让浏览器与服务端进行沟通，从而决定请求或响应是应该成功，还是应该失败。整个 CORS 通信过程，都是浏览器自动完成，不会让用户感觉到与普通请求有任何差别。
　　在使用 CORS 进行跨域时，浏览器会自动在请求头中加入 `Origin` 字段，指定本次请求来自哪个源(协议 + 域名 + 端口)，服务端会根据这个值来判断是否同意这次跨域请求。如果 `Origin` 指定的源不在许可范围内，服务端返回正常的 HTTP 回应，但是这次 HTTP 响应头中不包含 `Access-Control-Allow-Origin`，代表本次跨域请求失败，错误会被 XHR 的 onerror 回调函数捕获；如果 `Origin` 指定的源在许可范围内，就会在 HTTP 响应头中发现 `Access-Control-Allow-Origin` 字段，它的值可能是 `Origin` 传给服务端的值，也有可能是 *(代表公共资源)。
　　以上说的 CORS 请求都是阮老师 [跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html) 一文中提到的简单请求，复杂请求会在第一次发送跨域请求时多一次 HTTP 请求，但是都是浏览器自动完成的，细节请参阅阮老师的文章。

2. JSONP(JSON with padding)
　　使用 JSONP 需要知道一个前提就是，script 标签本身就可以跨域的，将一个跨域请求放到 src 属性中即可。回想一下，你页面里引的 jQuery 源码是不是请求就是百度 CDN 上的资源呀？
　　JSONP 是包含在函数调用中的 JSON，例如`callback({ "name": "Julia"})`。它由两部分组成：回调函数和数据。回调函数是当响应到来时应该在页面中调用的函数，回调函数的名字一般是在请求中指定的。而数据就是传入回调函数的 JSON 数据。
　　这里有一个典型的 JSONP 请求：`http://freegeoip.net/json?callback=handleResponse`。我们来分析一下上面这句话。首先，这是一个 JSONP 请求，代表这个请求要获取的是包含 JSONP 格式的数据；然后看到请求的格式是在 URL 后面加一些参数，也就是 GET 请求的方式，划重点啦，*JSONP 只支持 GET 方式进行请求*；再接着看，参数是 `callback=handleResponse`，语义上来理解的话，代表回调函数是 handleResponse，那么意味着浏览器中肯定存在这个回调函数，并且在要请求的资源里，肯定具有 `callback({ "name": "Julia"})` 这种形式以便于调用函数。OK，分析完了这句话，我觉得 JSONP 的道理大家也应该都懂了吧。

　　之所以将 JSONP 也归于需要服务端支持的跨域，与 CORS 的原因有很大差别。JSONP 的关键在于被请求的资源中需要包含 URL 中指定的函数并且调用它，以实现数据的跨域传输。所以，准确来说这并不是需要服务端的支持，而是需要被请求资源的支持。而 CORS 主要是通过头部信息进行跨域，所以毫无疑问需要服务端的支持。

#### 不需要服务端支持的跨域

图像 Ping
　　与 script 标签类似，浏览器中还有一个标签可以跨域，那就是 img 标签。以我自己举例来说，没有什么可用的 CDN 服务器，可是发一篇文章又想图文并茂，所以往往会在微博上传一些图片，然后获取到图片的 URL 后再把微博删了...好像跑题了，意思就是在我的博客里你看到的好多图片，我用的都是新浪微博的静态图片资源。
　　使用该方式进行跨域请求很简单，举例如下：
```
var img = new Image();
img.onload = img.onerror = function() {
  alert('Done!');
};
img.src = 'http://www.example.com/test?name=Julia';
```
　　从上面这段很短的代码中，我想也是可以得到一些信息的。第一，使用图像 Ping 方式跨域只支持 GET 请求，这是 img 标签所决定的；第二，这是一种与服务端进行简单、单向的跨域通信的一种方式，请求的数据是以查询字符串形式发送的，而响应可以是任意内容，但是很可惜，并不能访问服务端的响应文本(隐含意思就是，你别想通过这个方式获取服务端的啥信息)。

### 参考文献
[跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)
