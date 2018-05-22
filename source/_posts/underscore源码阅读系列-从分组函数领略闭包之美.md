---
title: underscore(1.9.0)源码阅读系列-从分组函数领略闭包之美
date: 2018-05-22 11:43:14
tags:
---

### underscore 中的分组函数
　　在 underscore 中，提供了四个有关分组的函数，分别是 _.groupBy、_.indexBy、_.countBy 以及 _.partition。先来看看这四个 API 的作用。
　　_.groupBy(list, iteratee, [context])：可以用来对一个集合进行分组，这里的集合不只包括数组，同时也包括对象。分组的依据是函数传入的第二个参数 iteratee，这个 iteratee 可以是一个函数，接收集合中的元素作为入参，然后返回一个值作为分组依据。也可以传入一个字符串表示元素属性，直接根据该属性进行分组。当然，iteratee 的值不仅限于上面两种，但是我相信这已经可以涵盖你大部分的应用场景了。
```
// list 为数组，iteratee 为字符串
// 根据字符串的长度进行分组
_.groupBy(['one', 'two', 'three'], 'length');
=> {3: ["one", "two"], 5: ["three"]}

// list 为对象，iteratee 为函数
// 根据值的奇偶进行分组
_.groupBy({x:1, y:2, z:4}, function(val) { return  val % 2 === 0 ? 'even': 'odd'; });
=> {even: [2, 4], odd: [1]}
```
　　接下来看 _.indexBy(list, iteratee, [context])。
```
var stooges = [{name: 'moe', age: 40}, {name: 'larry', age: 50}, {name: 'curly', age: 60}];
_.indexBy(stooges, 'age');
=> {
  "40": {name: 'moe', age: 40},
  "50": {name: 'larry', age: 50},
  "60": {name: 'curly', age: 60}
}
```
　　可以看到，_.indexBy 跟 _.groupBy 类似，但是它的返回结果中，每个 key 值只对应一个 value 而不是一个 value 数组。如果在原来的集合中，有两个元素经过 iteratee 后产生了相同的值，那么在结果对象中，后者会覆盖前者。所以在使用 indexBy 之前，最好先确认一下集合中的元素经过 iteratee 后的值是各不相同的。
　　然后看 _.countBy(list, iteratee, [context])。
```
_.countBy([1, 2, 3, 4, 5], function(num) {
  return num % 2 == 0 ? 'even': 'odd';
});
=> {odd: 3, even: 2}
```
　　该函数也类似于 _.groupBy，我们之前在 _.groupBy 中有一个根据值的奇偶进行分组的例子，可以对比 _.countBy 的例子一起看，会发现 _.countBy 与 _.groupBy 相比，唯一不同的是 _.groupBy 是统计结果而 _.countBy 是统计个数。
　　上面的三个函数从名字来看就是一家人，不过在 underscore 中还有一个用来分组的函数是 _.partition(list, iteratee) 。这个函数所做的可能更符合“分组”这个词，它把一个集合分为两个数组，其中第一个数组中的元素都满足 iteratee，第二个数组中的元素都不满足 iteratee。
```
_.partition([0, 1, 2, 3, 4, 5], function(num) {
  return num % 2 == 0 ? false: true;
});
=> [[1, 3, 5], [0, 2, 4]]
```
　　单凭感觉来看，这四个函数所做的事情很类似，如果每一个都独立实现的话，可能会有很多冗余的代码。翻一翻 underscore 的源码，果然有一个公用的分组函数为它们提供了底层支持，这个分组函数就是 group。

### 公用函数 group
　　上面四个函数的功能我们都已经了解了，这里直接贴出源码中 group 函数的实现。
```
function group(behavior, partition) {
  // 返回一个函数，这样可以继续被调用
  return function(list, iteratee, context) {
    // 根据是不是 partion 来决定返回结果的形式
    let result = partition ? [[], []] : {};
    // 根据传入的 iteratee 来决定迭代函数
    iteratee = cb(iteratee, context);
    // 遍历集合
    _.each(list, function(val, key) {
      // 分组的依据就是 key
      let key = iteratee(val, key, list);
      behavior(result, val, key);
    });
    return result;
  }
}
_.groupBy = group(groupByBehavior);
_.indexBy = group(indexByBehavior);
_.countBy = group(countByBehavior);
_.partition = group(partitionBehavior, true);
```
　　上面是对以上四个函数公有代码的实现，而各个函数的特有逻辑通过传入一个 behavior 函数来实现，behavior 函数所进行的主要工作就是对 result 对象进行操作（因为 result 是一个引用类型，所以在 behavior 函数中对它的任何修改都可以直接反映到外部的函数中）。以_.groupBy 和 _.partition 为例来继续完善上面的代码。
　　_.groupBy 中的值是一个数组，该数组存储了所有可以得到对应键的元素。它的行为可以封装如下：
```
var groupByBehavior = function(result, value, key) {
  if (_.has(result, key)) {
    result[key].push(value);
  } else {
    result[key] = [value];
  }
}
```
　　_.partition 中的返回结果是固定的形式[[], []]，其中第一个数组中存储满足迭代函数的元素，第二个数组中存储不满足迭代函数的元素。它的行为可以封装如下：
```
var partitionBehavior = function(result, value, pass) {
  result[pass ? 0 : 1].push(value);
}
```

### 闭包的应用
　　我们这篇文章的名字是*从分组函数领略闭包之美*，但是上文半字都没提到闭包，虽然略有标题党的感觉，但是我这里还是会yingche点关系到闭包上来的！闭包在之前的文章[吓唬人的闭包](http://zhuyali.com.cn/2018/01/09/%E5%90%93%E5%94%AC%E4%BA%BA%E7%9A%84%E9%97%AD%E5%8C%85/)中有细讲过，但是经过这几个月的学习，对闭包的理解又加深了一些。首先回到那个老掉牙的问题：什么是闭包？最近有一个答案我很是喜欢：当函数可以记住并访问所在的词法作用域时，就产生了闭包，即使函数在当前词法作用域之外执行。那结合我们本文中的分组函数，闭包主要体现在 group 函数中返回的函数，该函数记住了 behavior 和 partition，即使 group 函数已经执行完成，但是返回的函数仍然保留有对外部作用域的引用，这里就形成了闭包。
　　结合本文中我们遇到的场景，当需要实现几个相似的功能时，如果独立开实现会造成大量冗余的代码和重复的逻辑，而闭包就是一个解决这种问题的很好的方案，这应该也是闭包的可爱之处吧~　　

### 参考文献
1. 你不知道的JavaScript（上卷）
2. [浅谈 underscore 内部方法 group 的设计原理](https://github.com/hanzichi/underscore-analysis/issues/16)