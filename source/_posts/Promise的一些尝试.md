---
title: Promise的一些尝试
date: 2018-03-05 16:35:33
tags:
---

### 前言
　　该文章不是 Promise 入门，不涉及基础讲解，只是博主对 Promise 的一些想法和尝试而已。

### Promise 基本用法

- __Promise 是一个容器，里面保存着某个未来才会结束的事件。当这个未来事件结束后，可以通过 resolve 函数和 reject 函数来改变 Promise 的状态，使它可以继续执行依赖该异步结果的代码（即 then 中或 catch 中的代码）__

- __Promise 共有三种状态：pending（进行中），fulfilled（已成功）和 rejected（已失败），只有异步操作的结果能够决定当前是哪一种状态，并且一旦决定，就无法再改变__

- __Promise 一旦新建就会立即执行，无法中途取消__
```
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});
promise.then(function() {
  console.log('resolved.');
});
console.log('Hi!');
// Promise
// Hi!
// resolved
// 通过上面的输出顺序可以看出，Promise 是立即执行的，而 then 中的回调函数会在同步代码都执行完毕后才会执行
```

- __如果不设置回调函数，Promise 内部抛出的错误不会反应到外部。Promise 内部的错误包括两种：一是 throw 出来的异常，二是调用 reject__
```
let promise = new Promise(function(resolve, reject) {
  reject('error'); // or throw new Error('error');
});
console.log('Hi!');
// Promise 内部抛出的错误并不会影响后续代码的执行，还是会输出 Hi!
```

- __Promise 中调用 resolve 和 reject 时都可以传入参数，这个参数会被传递给回调函数。如果该参数是一个 Promise，那么原 Promise 的状态就无效了，而是由传入的 Promise 的状态来决定__
```
const p1 = new Promise(function(resolve, reject) {
  setTimeout(() => reject(new Error('fail')), 3000)
})
const p2 = new Promise(function(resolve, reject) {
  setTimeout(() => resolve(p1), 1000)
})
p2
  .then(result => console.log(result))
  .catch(error => console.log(error))
// Error: fail
// 此时 p2 本身的状态无效，而是要依赖于 p1 的状态
```

- __Promise 并不会因 resolve 或 reject 函数的调用而终止自身的立即执行__
```
new Promise(function(resolve, reject) {
  resolve(1);
  console.log(2);
}).then(r => {
  console.log(r);
});
// 2
// 1
// 在 Promise 内部的代码都是立即执行的，不管它是否位于 resolve 之后
```

### Promise 原型函数

#### Promise.prototype.then()

- __then 可以接受两个参数，第一个参数是 resolved 状态的回调函数，第二个参数（可选）是 rejected 状态的回调函数__

- __返回一个新的 Promise 实例，所以可以进行链式调用 then，后续 then 中回调函数所接受的参数来源于之前 then 中的返回值。如果这个返回值是非 Promise，那么会直接传给 resolve 回调函数；如果这个返回值是 Promise，那么则需要等待该 Promise 的状态改变__
```
new Promise(function(resolve, reject) {
  resolve('外层 resolve');
}).then(val => {
  console.log(val);
  return new Promise(function(resolve, reject) {
    resolve('内部 resolve');
  });
}).then(val => {
  console.log(val);
})
// 外层 resolve
// 内部 resolve
// 后面输出内部 resolve，说明它等待了上一个 then 中返回的 Promise 的状态改变
```

#### Promise.prototype.catch()

- __Promise.prototype.catch 方法是 .then(null, rejection) 的别名，用于指定错误发生时的回调函数__

- __Promise 对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止__

- __总是使用 catch 方法代替 then 方法的第二个参数，因为 catch 还可以捕获前面 then 方法执行中的错误__

- __catch 返回的也是一个 Promise 对象，所以后面可以继续链式调用 then 或者 catch__

### Promise 静态函数

#### Promise.all()

- __接受的参数往往是一个 Promise 实例数组。如果不是数组，也可以是任何具有 Iterator 接口的迭代器；如果不是 Promise 实例，则会先调用 `Promise.resolve`方法将参数转为 Promise 实例__
```
Promise.all([1, 2, 3])
  .then(val => {
    console.log(val);
  });
// [ 1, 2, 3 ]
// 等价于
Promise.all([Promise.resolve(1), Promise.resolve(2), Promise.resolve(3)])
  .then(val => {
    console.log(val);
  });
```

- __返回一个新的 Promise（后面称为 p）。只有当传入的 Promise 实例都变成 fulfilled 状态，p 的状态才会变成 fulfilled；否则，如果传入的 Promise 实例有一个被 rejected，p 的状态就变成 rejected__
```
const p1 = new Promise((resolve, reject) => {
  resolve('hello');
});

const p2 = new Promise((resolve, reject) => {
  throw new Error('报错了');
})
.catch(e => e);

Promise.all([p1, p2])
.then(result => console.log(result))
.catch(e => console.log(e));
// ["hello", Error: 报错了]
// 上面的情况其实是进入了 then 方法中。原因是虽然 p2 定义的 Promise 状态变为 rejected 中了，但是其中最终传入 Promise.all() 的 p2 是 catch 方法的返回值
```

- __返回的新的 Promise 实例 p 后的 then 也接受两个回调函数作为参数。如果是 fulfilled 的状态，那么回调函数参数为传入的 Promise 实例的返回值组成的数组；如果是 rejected 的状态，那么回调函数为第一个被 rejected 的实例的返回值__

#### Promise.resolve()

- __一般用于将现有对象转为 Promise 对象__
```
Promise.resolve('foo');
// 等价于
new Promise(resolve => resolve('foo'));
```

- __立即 resolve 的 Promise 对象，是在本轮“事件循环”结束时而不是下一轮“事件循环”的开始时__
```
setTimeout(function () {
  console.log('three');
}, 0);
Promise.resolve().then(function () {
  console.log('two');
});
console.log('one');
// one
// two
// three
```

### 扩展题目
// 实现一个方法 parallel(tasks, concurrency)，让 tasks 并发执行（并控制并发数为 concurrency)
// 其中 tasks 为一个数组，每一个元素都是一个方法返回一个 promise
// 当所有 tasks 执行完成时，resolve 一个数组保存所有的结果
// 当任意一个 task 执行失败时，reject 这个错误

```
const assert = require('assert');

function task(input) {
  return new Promise(resolve => {
    setTimeout(() => resolve(input), 1000);
  });
}

parallel([
  task(1),
  task(2),
  task(3),
  task(4),
  task(5),
], 2)
.then(res => assert.deepEqual(res, [1, 2, 3, 4, 5]));

function parallel(tasks, concurrency) {
  
}
```
　　参考答案如下：
```
function parallel(tasks, concurrency) {
  let concurrentTasks = tasks.slice(0, concurrency);
  tasks = tasks.slice(concurrency);
  let promise = new Promise((resolve, reject) => {
    const results = [];
    for (let i = 0; i < concurrency; i++) {
      concurrentTasks[i].then(done, error);
    }

    function done(result) {
      results.push(result);
      const task = tasks.shift();
      if (task) {
        task.then(done, error);
      } else {
        resolve(results);
      }
    }

    function error(err) {
      reject(err);
    }
  });
  return promise;
}
```


### 参考资料
[ECMAScript 6 入门](http://es6.ruanyifeng.com/)
