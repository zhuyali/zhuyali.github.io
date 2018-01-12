---
title: JavaScript高级程序回顾(四)——飘忽不定的 this?
date: 2018-01-11 17:43:15
tags:
---
　　this 可谓是一个很让人头疼的问题了。以我的经验来看，我刚学完 this 的时候，看待 this 好像是看待自己亲儿子；一天以后，差不多就是干儿子了；再过一天，可能是充话费送的；然后，可能就是陌生人了...虽然这么说有点夸张，但是 this 确实是需要反复学习和琢磨的。这篇文章在我头脑比较清晰的时候记录下来，以备以后的不时之需。

### 进入正题
　　this 是在函数被调用时才发生绑定的，它指向什么完全取决于函数在哪里被调用。这里需要先明确两个概念：调用位置和调用栈，看完以下的代码后，你会了然于心。
```
function baz() {
  // 当前调用栈为 baz
  console.log('baz');
  bar(); // 函数 bar 的调用位置
}
funcion bar() {
  // 当前调用栈为 baz -> bar
  console.log('bar');
  foo(); // 函数 foo 的调用位置
}
function foo() {
  // 当前调用栈为 baz -> bar -> foo
  console.log('foo');
}
baz(); // 函数 baz 的调用位置
```

### this 的绑定规则
　　接下来，我们正式开启寻秘 this 的旅程。开始之前，我觉得有必要提醒一下读者，时刻谨记“this 的指向完全取决于函数在哪里被调用”这句话，把这个思想贯穿到对 this 的思考中，可能你很快就能得到答案。

#### 默认绑定
　　思考如下代码：
```
function foo() {
  console.log(this.a);
}
var a = 2;
foo(); // 输出 2
```
　　毫不意外，这种模式的 this 是非常普遍的。可以看出，foo 的调用位置在全局环境中，并且它不带任何修饰，所以 this 指向了全局对象，this.a 访问的自然是全局环境中的 a 了。

#### 隐式绑定
　　思考如下代码：
```
function foo() {
  console.log(this.a);
}
var obj = {
  a: 2,
  foo: foo
};
obj.foo(); // 输出 2
```
　　上面的代码可以稍作分析。在 obj 中定义了属性 foo，它指向了全局函数 foo 的引用；然后在全局环境中，obj 调用了自己的 foo 函数，可以说最终调用时 foo 函数的落脚点是 obj 对象。此时 foo 函数中的 this 上下文是 obj 对象。

#### 显式绑定
　　思考如下代码：
```
function foo() {
  console.log(this.a);
}
var obj = {
  a:2
};
foo.call(obj); // 输出 2
```
　　通过 foo.call(...)，我们可以在调用 foo 时强制把它内部的 this 绑定到 obj 上，apply 函数也可以达到相同的效果。这是基础 API，详情可以参考 [Function.prototype.apply()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply) 和 [Function.prototype.call()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)。

#### new 绑定
　　思考如下代码：
```
function foo(a) {
  this.a = a;
}
var bar = new foo(2);
console.log(bar.a); // 输出 2
```
　　这种绑定形式也十分常见，具体细节可以参考我的另一篇文章[面向对象的 JS](https://zhuyali.github.io/2018/01/11/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%9A%84JS/)

#### 小结
　　以上四种 this 的绑定规则我们都已经有所了解，接下来你所要做的就是找到函数的调用位置并判断应该应用哪条规则。但是，如果某个调用位置可以同时应用很多条规则怎么办？答案是优先级！四种规则的优先级如下：默认绑定 < 隐式绑定 < 显式绑定 < new 绑定。

### 一个栗子
　　以下是我在刷知乎的时候，看到的一道题目。这道题目很好地展示了一种现象：“绑定丢失”。先贴代码：
```
var foo = function(){
  this.myName = 'Foo function.';
}
foo.prototype.sayHello = function(){
  alert(this.myName);
}
foo.prototype.bar = function(){
  setTimeout(this.sayHello, 1000);
}
var f = new foo();
f.bar(); // undefined
```
　　为什么答案是 `undefined` 而不是 `Foo function.` 呢？这里我们就来分析一下。首先，从调用位置来看，bar 函数的调用位置是在 f 上，所以 bar 函数内部的 this 应该指向 f。你可以验证一下，在 bar 中输出 this 试试。然后进入 bar 函数之后，内部执行了 `setTimeout(this.sayHello, 1000)`，setTimeout 函数传入了两个参数，第一个参数是 this.sayHello，这里的 this 仍然指向 f，原因是这时候只是传参数而已，还没有涉及到任何的调用逻辑。然后，调用了 this.sayHello 之后，神奇的事情发生了，undefined 的结果说明 this 的指向变了，可是到底在哪里发生了变化呢？我们先揭露一下 setTimeout 的本质：
```
function setTimeout(fn, delay) {
  // 等待 delay 毫秒
  fn();
}
```
　　接下来分析函数的调用栈，进入 sayHello 函数以后，函数的调用栈为 bar -> setTimeout -> sayHello。诶，中间多了一个 setTimeout，我想这就是问题的关键所在了。sayHello 函数调用时，它处在 setTimeout 函数中，而 setTimeout 是全局对象的函数，所以这时候的 this 指向全局对象，全局对象上没有定义 myName，所以得到 undefined 的结果。要不，你试试全局定义一个 myName，看看会发生什么吧！

### 留下思考
　　最后留下一个简单的思考题，看看你能不能迅速 GET 到答案吧！
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
bar();
```