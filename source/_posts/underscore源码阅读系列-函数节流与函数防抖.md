---
title: underscore(1.9.0)源码阅读系列-函数节流与函数防抖
date: 2018-06-06 22:10:22
tags:
---

### 节流和防抖的应用场景辨析
　　在开始这个这个部分前，我想先提问两个问题，看看你是否曾对节流和防抖有一定的概念了解？
- 问题一：当我们改变浏览器窗口大小时，如果我们在 resize 事件中进行了大量的运算，会导致浏览器非常的卡顿，解决这种问题的方案是节流还是防抖？
- 问题二：当用户在输入框中输入文字时，我们需要根据用户输入从远程服务器拉取数据，如果不做限制，请求数量在短时间内会骤增，那么我们是通过节流还是防抖来解决这个问题？
　　这部分的标题是应用场景辨析，之所以需要辨析，是因为它们的应用场景往往十分类似（都是为了防止触发频繁的回调），很多有经验的前端工作者也未必能够明确它们之间到底有什么细微的差别。废话不多说，上面两个问题的答案分别是节流和防抖，我们就从这两个场景来进入节流和防抖的世界。
　　函数节流旨在进行频率控制，即在一段时间内，一段代码函数只能执行一次。套用到 resize 的场景中 - resize 事件是不断被触发的，但是该事件的回调函数是以某种频率执行的，它保证了程序的执行不是没有间断地连续重复执行。
　　函数防抖旨在进行空闲控制，只有当调用动作 n 毫秒后，才会执行代码函数，如果这 n 毫秒内又调用此动作则将重新计算执行时间。套用到输入框的场景中 - 用户输入文字 n 毫秒后，才去服务器拉取数据，如果在这 n 毫秒中用户又进行了输入操作，则重新开始计时，以此往复。

### underscore 中的函数节流
　　在 underscore 源码中对于函数节流的实现还包括一些高级功能，如设置头执行和尾执行，这与函数节流本身的思想不相关，所以在代码中也不提及这部分的实现。以下是节流的核心实现及注释：
```
_.throttle = function(func, wait) {
    var context, args, result;
    // previous 用于存储上次执行时间
    var previous = 0;

    // 该函数被返回，因此实际执行的是该函数
    var throttled = function() {
      // 当前系统时间
      var now = _.now();
      // 上次执行到本次执行还剩余的毫秒数
      var remaining = wait - (now - previous);
      context = this;
      args = arguments;
      // 如果剩余的毫秒数 <= 0 或者是 调整系统时间到更早的时间的话
      if (remaining <= 0 || remaining > wait) {
        // 设置上次执行时间为当前时间
        previous = now;
        // 执行函数并返回结果
        result = func.apply(context, args);
      }
      return result;
    };
    return throttled;
  };
```
　　emmm..对源代码删删减减之后，得到了如上的非常简单的版本。它的思路就是每次执行前先判断当前系统时间 - 上次执行时间是否已经超过了设置的间隔 wait 毫秒数，如果超过了（>=）就执行函数并返回结果，同时重置上次执行时间；否则直接返回上次的执行结果。

### underscore 中的函数防抖
　　类似于函数节流，源码中函数防抖也有高级功能实现，在下面的源码讲解时也不会提及。以下是防抖的核心实现及注释：
```
  _.debounce = function(func, wait) {
    var timeout, result;

    // later 函数的执行时机是上一次 debounced 函数被执行 wait 毫秒后
    // 它的职责是清空定时器并且执行 func 函数得到 result
    var later = function(context, args) {
      timeout = null;
      if (args) result = func.apply(context, args);
    };

    // 该函数被返回，因此实际执行的是该函数
    var debounced = restArguments(function(args) {
      // 如果在执行 debounced 函数时
      // 上次设置的 timeout 还未被设为 null，则意味着还未到下次执行的时机
      // 也就是说，本次执行该函数和上次执行的间隔时间还未到达 wait 毫秒
      // 那么就清空该计时器
      if (timeout) clearTimeout(timeout);
      // 然后重新设置定时器，在 wait 毫秒后执行 later 函数
      timeout = _.delay(later, wait, this, args);
      // 返回 result
      return result;
    });

    return debounced;
  };
```

### 总结
　　其实在 underscore 中函数节流和防抖的实现也并无特别之处，之前我写的一篇文章[JavaScript高级程序回顾(六)——高级技巧](http://zhuyali.com.cn/2018/01/15/%E9%AB%98%E7%BA%A7%E6%8A%80%E5%B7%A7/)中也有讲到过节流和防抖，其中防抖的实现方案与 underscore 中没有什么差别，节流的实现方案就稍有不同，好奇的同学可以去看一看，看看哪一种实现更符合你的心意。