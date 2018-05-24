---
title: underscore(1.9.0)源码阅读系列-硬绑定函数背后的故事
date: 2018-05-24 19:38:23
tags:
---

### 硬绑定函数

#### 绑定丢失问题
　　在之前的文章[飘忽不定的 this](http://zhuyali.com.cn/2018/01/11/%E9%A3%98%E5%BF%BD%E4%B8%8D%E5%AE%9A%E7%9A%84-this/)中详细介绍过 this 的绑定规则，并且在*一个栗子*部分分析了绑定丢失的问题，这里重新回顾一下。先看以下的代码：
```
function foo() {
    console.log(this.a);
}
var obj = {
    a: 2,
    foo: foo
};
var bar = obj.foo;
var a = 'oops, global';
obj.foo(); // 2
bar(); // 'oops, global'
```
　　obj.foo() 应用了隐式绑定的规则，所以 foo 中的 this 指向了 obj 对象。而 bar 虽然是 obj.foo 的一个引用，但是实际上，他引用的是 foo 函数本身，因此此时的 bar() 其实是一个不带任何修饰的函数调用，跟直接调用 foo() 没什么差别，因此使用了默认绑定，其中的 this 指向了全局对象。
　　之所以重温这部分，是为了引出后文。JavaScript 原生提供了可以解决绑定丢失问题的方法，那就是 Function.proptotype.bind。

#### Function.prototype.bind
　　[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)上对于 bind 的方法的介绍是：bind() 方法创建一个新的函数，当被调用时，将其 this 关键字设置为提供的值，同时也接受预设的参数提供给原函数。从定义上来看，bind 函数可以绑定一个函数的 this 使其不动态的变化，我们来使用 bind 函数解决上面的绑定丢失的问题。
```
function foo() {
    console.log(this.a);
}
var obj = {
    a: 2,
    foo: foo
};
var bar = obj.foo.bind(obj);
var a = 'oops, global';
obj.foo(); // 2
bar(); // 2
```
　　bind 函数会返回一个硬编码的函数，它在原始函数的基础上绑定了函数的上下文 this，这种机制被称为硬绑定，是显示绑定的一种。

注：但是 bind 绑定的 this 也不是不会变的，当可以应用多条绑定规则时，this 绑定的优先级为：new 绑定 > 显示绑定 > 隐式绑定 > 默认绑定。

### underscore 中的硬绑定函数
　　在 underscore 中，实现了一个类似于原生 Function.prototype.bind 的函数 _.bind，不过原生的 bind() 函数在 ECMA-262 第五版才被加入，可能无法在所有的浏览器上运行；相较来说，_.bind 可以兼容各个浏览器。下面简单介绍一下 _.bind：
　　_.bind(function, object, *arguments) 绑定函数 function 到对象 object 上，也就是无论何时调用函数，函数里的 this 都指向这个 object，任意可选参数 *arguments 的目的是函数柯里化，与本节内容关系不大，暂且不聊。它的用法为：
```
var func = function(greeting){ return greeting + ': ' + this.name };
func = _.bind(func, {name: 'moe'}, 'hi');
func();
=> 'hi: moe'
```

### _.bind 的具体实现
```
_.bind = restArguments(function(func, context, args) {
  if (!_.isFunction(func)) throw new TypeError('Bind must be called on a function');
  var bound = restArguments(function(callArgs) {
    return executeBound(func, bound, context, this, args.concat(callArgs));
  });
  return bound;
});
```
　　上面的代码中，restArguments 是为了格式化参数的，它的返回值是一个函数。以 _.bind 为例，如果我们调用了 _.bind(bindFunc, bindContext, arg1, arg2...)，实际上会调用传入 restArguments 的第一个函数参数，这个函数中的参数会被进行格式化：
```
_.bind = restArguments(function(func, context, args) {
  // 进入该函数之后
  // func 引用的是 bindFunc
  // context 引用的是 bindContext
  // args 是一个数组，形式为[arg1, arg2...]
  // restArguments 把最后一个参数作为数组形式，可以容纳任意数量的参数
  // 可以得出结论：restArguments 适合那些传任意多参数的情况
});
```
　　刚刚提到，调用 _.bind 时实际上会调用传入 restArguments 的内部匿名函数，所以我们来分析这个内部匿名函数：
```
function(func, context, args) {
  // 首先进行参数有效性判断，func 必须是一个函数
  if (!_.isFunction(func)) throw new TypeError('Bind must be called on a function');
  // args 是柯里化参数，是个数组
  var bound = restArguments(function(callArgs) {
    return executeBound(func, bound, context, this, args.concat(callArgs));
  });
  return bound;
}
```
　　这个函数的返回值是 bound 函数，也就是说我们在调用由 _.bind 生成的函数时，实际上是调用了 bound 函数。这个 bound 函数也是由 restArguments 生成的，同理，在调用 bound 函数时实际调用了传入 restArguments 的内部匿名函数，并且该函数的参数被格式化成了一个数组，而该匿名函数的返回值是 `executeBound(func, bound, context, this, args.concat(callArgs))`，这个返回值也就是我们最后调用 _.bind 生成的函数时得到的结果。
　　至此，我们先整理一下思路，调用 _.bind 返回了函数 bound，调用 bound 返回了 executeBound(func, bound, context, this, args.concat(callArgs))。接下来我们把重点放在 executeBound 函数上。

### 核心函数 executeBound
　　在继续之前，想先强调至关重要的一点：调用 _.bind 返回的是一个函数，是函数就意味着它既可以被普通调用，也可以被 new 调用，因此上面提到的 bound 函数，既可以 bound(...)，也可以 new bound(...)。明确这一点之后我们来看 executeBound 的源码。
```
var executeBound = function(sourceFunc, boundFunc, context, callingContext, args) {
  if (!(callingContext instanceof boundFunc)) {
      return sourceFunc.apply(context, args);
  }
  // 使用 bound 实例化出来的对象
  // 接下来经历 new 过程时发生的事情
  // 1.创建一个继承自 sourceFunc 的对象
  // 2.绑定新创建的对象到 this
  var self = baseCreate(sourceFunc.prototype);
  var result = sourceFunc.apply(self, args);
  // 如果 sourceFunc 没有返回一个对象的话，默认就返回继承自 sourceFunc 的对象
  if (_.isObject(result)) return result;
  return self;
};
```
　　executeBound 接受五个参数：sourceFunc 代表原始被 bind 的函数，boundFunc 就是 _.bind 的返回函数 bound，context 代表传给原始函数的上下文，callingContext 代表调用 bound 函数时候的上下文，args 代表要传入 sourceFunc 的参数。

#### 普通调用
　　普通调用下 !(callingContext instanceof boundFunc) 条件永远成立，所以会直接进入分支，返回 sourceFunc.apply(context, args)，这里会绑定 sourceFunc 的上下文到 context 并执行 sourceFunc(args[0], args[1]...)。

#### new 调用
　　new 调用会创建一个 bound 的实例，并且把 this 绑定到这个实例上，此时 !(callingContext instanceof boundFunc) 条件不成立，会继续向下执行，构造一个原始函数的实例。因为这里其实是在模仿真实的 new 过程，不妨先去看看之前我写的一篇文章[面向对象的JS](http://zhuyali.com.cn/2018/01/12/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%9A%84JS/)中构造器方式构造对象的部分。首先会创建一个 sourceFunc 的实例 self，这里的 baseCreate 等价于 Object.create(sourceFunc.prototype)，创建出的实例拥有 sourceFunc.prototype 上的所有属性和方法；然后接下来绑定 sourceFunc 的 this 到创建出的实例上并调用 sourceFunc 得到 result；这个得到的 result 完全取决于 sourceFunc 的返回值，可能是任何类型，但是当且仅当在 result 是个对象的时候返回 result，否则返回的是创建出的那个实例。

### 参考资料
你不知道的JavaScript（上卷）