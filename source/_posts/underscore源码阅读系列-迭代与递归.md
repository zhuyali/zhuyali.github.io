---
title: underscore(1.9.0)源码阅读系列-迭代与递归
date: 2018-05-29 16:20:21
tags:
---

　　关于迭代和递归是什么，概念性的东西我不多介绍，举个例子（参考知乎上[「递归」和「迭代」有哪些区别？](https://www.zhihu.com/question/20278387)的回答)，比如你要给某小孩买玩具，迭代是你挑了一件觉得不行，又挑了一件又不行，如此这般，直到找到合适的玩具。而递归是你自己不太了解小孩子的需求，为了缩小范围，你让你的儿子去挑选，儿子虽然比你强点，但依然不太了解小孩子的需求，然后他又让你孙子去挑选，如此这般，直到找到合适的玩具。
　　underscore 中使用了大量的迭代和递归，这里粗浅罗列一下，使用到迭代的方法有 _.property、 _.reduce、 _.compose、 _.has、 _.result、 _.propertyOf 以及 _.min 和 _.max 等，使用到递归的方法有 _.flatten 和 _.isEqual。其实递归和迭代在平时的编码中也都会用到很多，我想大家对它的掌握程度都不错，这里之所以要专门写一篇文章来介绍递归和迭代，主要是因为我在看源码的过程中，多次看到了递归和迭代的使用，但却很少注意到这两种用法，所以就有了以下的“温故知新”。

### underscore 中的迭代 - _.reduce
　　用到迭代的函数千千万，这里就只聊聊经典的 _.reduce。reduce 函数应该是每一个 JavaScript 开发者都熟悉的函数，因为 Array 原生提供了 Array.prototype.reduce(callback[, initialValue]) 函数，这个函数接收两个参数，第一个参数 callback 是一个函数，接收 (accumulator, currentValue, [currentIndex], [array])，其中后面三个参数语义比较明显，这里不做解释。第一个参数的字面意思是累加器，它是上一个数组元素执行 callback 的返回结果，而第一个元素没有上一个参数，怎么办呢？这时候就可以可选的传入第二个参数 initialValue，如果传了该参数，那么 accumulator 的值是 initialValue，currentValue 是数组中下标为 0 的元素；如果没有传该参数，那么 accumulator 的值是数组中下标为 0 的元素，currentValue 是数组中下标为 1 的元素。根据上面的描述，可以知道上一次 callback 的值是下一次 callback 的第一个参数，这...？很明显就是用了迭代吧~ 好了，接下来我们看看具体实现，在 underscore 中，reduce 家族(_.reduce 以及 _.reduceRight) 底层都是通过函数 createReduce(dir) 来实现的，其中参数 dir 代表方向。
```
var createReduce = function(dir) {
  var reducer = function(obj, iteratee, memo, initial) {
    var keys = !isArrayLike(obj) && _.keys(obj),
        length = (keys || obj).length,
        index = dir > 0 ? 0 : length - 1;
    // 如果没有传初始值
    if (!initial) {
      // 初始值就为 obj 中的第一个元素(_.reduce)或最后一个元素(_.reduceRight)
      memo = obj[keys ? keys[index] : index];
      // 索引置为下一项的索引(_.reduce) 或前一项的索引(_.reduceRight)
      index += dir;
    }
    for (; index >= 0 && index < length; index += dir) {
      // 当前元素的 key
      var currentKey = keys ? keys[index] : index;
      // 用上一次迭代的返回值，作为本次迭代的第一个参数
      memo = iteratee(memo, obj[currentKey], currentKey, obj);
    }
    return memo;
  };

  return function(obj, iteratee, memo, context) {
    // initial 代表有没有传入 memo 值
    var initial = arguments.length >= 3;
    // optimizeCb 负责绑定 iteratee 的上下文，同时规范了 iteratee 的参数为(accumulator, value, index, collection)
    return reducer(obj, optimizeCb(iteratee, context, 4), memo, initial);
  };
};
```
　　哈，源码贴出来了，注释也写得比较明白，非常简单，大家就自行理解吧。

### 扩展: 长度判断的方法
　　不过呢，简单的代码中，也少不了有疑惑的内容，在第一次看源码的过程中，我对下面几行代码看了好几遍才理清，所以这里也简单提一下。
```
var keys = !isArrayLike(obj) && _.keys(obj),
    length = (keys || obj).length;
var currentKey = keys ? keys[index] : index;
```
　　因为 _.reduce 方法是放在 Collections 类目下面，这代表这个方法不一定只针对数组可用，还对对象可用。然后分析上面的代码：首先 `isArrayLike(obj)` 判断传入的是不是一个类数组对象，本质上是判断 obj 是否具有 length 属性。如果判断为 true，根据 && 的截断机制，此时 keys 的值为 false，length 就取 obj.length，而 currentKey 就取索引值 index，看到这里我想很多人也猜到了，这是针对数组的判断方式。相反，如果判断为 false，就取到 obj 的键数组，length 值为 keys.length，对应的 currentKey 为 keys[index]，这是针对对象的判断方式。以上，看似好像都没有什么问题，但是我却不这么认为，比如有个对象是 `var obj = {x: 1, y: 2, length: 3}`，我们可以一眼看出这是一个对象，但是它却符合 isArrayLike 的标准，使得最后以处理数组的方式来处理这个对象，这显然是有问题的。对此，我有一个想法，那就是将 `isArrayLike(obj)` 替换为判断条件 `isArrayLike(obj) && (_.isArray(obj) || _.isArguments(value))`，这样能够保证不会错误筛选出具有 length 属性的对象。其实在 underscore 源码中就有做这样处理的地方，具体可以参见 _.flatten 的具体实现。

### underscore 中的递归 - _.flatten
　　underscore 中有两个经典的使用递归的函数：_.flatten 和 _.isEqual。_.isEqual 后面会单独有一篇文章进行讲解，所以这里我们以较为简单的 _.flatten 函数为例。_.flatten(array, [shallow]) 是针对数组的平铺函数，可以通过 shallow 来决定是深平铺还是浅平铺。举个例子来说：
```
[[[1, 2], [1, 2, 3]], [1, 2]] => [1, 2, 1, 2, 3, 1, 2]
[[[1, 2], [1, 2, 3]], [1, 2]] => [[1, 2], [1, 2, 3], 1, 2]
```
　　以上两种都是平铺数组，第一种是把所有的嵌套数组层层展开，也就是深平铺；第二种只展开了第一层嵌套数组，没有递归的展开下去，是浅平铺。接下来我们又要深入源码啦~ 我们先来看看 flatten 的调用形式：
```
var flatten = function(input, shallow, strict, output) {
  ...
}
```
　　这个 flatten 函数是 _.flatten 的核心函数。它接受四个参数：input 代表要展开的数组；shallow 决定了是否深度展开；strict 决定是否把非数组元素过滤掉；output 代表传入 flatten 的初始结果。下面举两个例子来分别说明一下最后两个参数：
```
// 第三个参数的 demo：因为 5 和 6 是非数组元素，所以在结果中直接过滤掉了
// 推论：如果 strict 为 true 并且 shallow 为 false，那么最终的返回结果一定为 output || []
var ans = flatten([5, 6, [1, 2], [3, 4]], true, true);
console.log(ans); // => [1, 2, 3, 4]
// 第四个参数的 demo
var ans = flatten([5, 6, [1, 2], [3, 4]], true, false, [7, 8]);
console.log(ans); // => [7, 8, 5, 6, 1, 2, 3, 4]
```
　　四个参数都理解之后，我们直接来看源码，加了非常多注释。
```  
var flatten = function(input, shallow, strict, output) {
  // output 数组保存结果，默认是 []
  // flatten 的返回结果为 output
  output = output || [];
  // idx 代表下一个添加的元素的下标
  var idx = output.length;
  // 遍历要展开的数组
  for (var i = 0, length = getLength(input); i < length; i++) {
    var value = input[i];
    // 如果 value 是数组或者 arguments
    if (isArrayLike(value) && (_.isArray(value) || _.isArguments(value))) {
      // 如果是浅平铺
      if (shallow) {
        var j = 0, len = value.length;
        // 直接遍历 value
        // 并且进行追加
        while (j < len) output[idx++] = value[j++];
      } 
      // 如果是深平铺
      else {
        // 递归遍历 value
        // 因为 output 是引用类型，所以对它的修改可以直接反应到外面
        // 即里面如果对 output 有改动，那么也会反应到外面的 output 上来
        flatten(value, shallow, strict, output);
        // 递归完之后，重新获取下一个添加的元素的下标
        idx = output.length;
      }
    } 
    // 如果 value 是个非 arguments 及 非数组
    // 并且如果 strict 为 false，会进入该分支
    // 意味着要追加非数组元素到结果中
    else if (!strict) {
      // 结果追加到 output 后面
      output[idx++] = value;
    }
    // 对于 output 的赋值操作 output[idx++] 仅仅发生在两处
    // 当 shallow 为 false 且 strict 为 true 的时候
    // 永远不会执行任意一处的代码
    // 所以返回的 output 永远为 output || []
  }
  return output;
};
```
　　总的来说，当进行浅平铺时，只展开第一层；当进行深平铺时，就要持续递归地调用 flatten，直到不能展开为止。
