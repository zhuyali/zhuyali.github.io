---
title: 浏览器和Node中不同的Event Loop
date: 2018-08-02 15:53:43
tags:
---
　　对于每个前端学习者来说，可能在刚接触 JS 的时候就会不断地被灌输一个概念：JS 是单线程的。持着对这个概念的深（似）入（懂）理（非）解（懂），我在 JS 的道路上摸爬滚打了些时日，直到一年多前第一次看到朴灵大大的《深入浅出 Node.js》中对事件循环的解释，我才第一次真正的去尝试着理解 JS 语言的单线程与异步。
　　原谅我又要重复一遍：JS 是单线程的，这句话的主语是 JS。譬如我们在 JS 写了一个 ajax 来发送异步的 HTTP 请求，无需等待 HTTP 的返回结果，我们照样可以继续向下执行代码，那是因为 HTTP 请求本身就不归 JS 管，自然也不需要占用 JS 线程来处理，只有当返回结果后，JS 会在某个空闲的时间去执行回调函数。JS 中另外一个很重要的概念大概就是异步了，它允许我们在不阻塞 JS 线程的前提下，还能拿到我们需要的结果（比如 ajax 拿到响应数据）。异步弥补了 JS 单线程的欠缺，正是因为有这样的机制，才使现在的 JS 语言如此流行，而这一切，都离不开事件循环机制。

### macrotask & microtask
　　接下来的事件循环总是围绕着 macrotask 和 microtask 来进行的，所以有必要提前搞清能够触发 macrotask 和 microtask 的任务源。

##### macrotask（宏任务）
　　在一个事件循环中，可能存在一个或多个宏任务队列，来自不同任务源的任务会放入不同的队列中，而任务的执行遵守先进先出（FIFO）原则。典型的宏任务源有：
1. 同步的 JS 代码
2. DOM 操作任务源：比如用户触发了点击事件，那么会将回调函数作为宏任务放入任务队列中
3. 网络任务源：比如 ajax 的回调函数
4. setTimeout 和 setInterval 定时器
5. setImmediate（ Node 环境中）

##### microtask（微任务）
　　在一个事件循环中，仅仅存在一个微任务队列，通常以下几种任务被认为是微任务：
1. Promise.prototype.then 和 Promise.prototype.catch。这里要注意下，创建 Promise 实例是同步的，所以属于宏任务
2. process.nextTick（ Node 环境中）

### 浏览器中的事件循环
<img src="https://haitao.nos.netease.com/ecf197da-9f12-4fb4-b486-8cf88f756507_1352_616.png" style="width: 70%;"/>　　浏览器内核是多线程的，它包括了 GUI 渲染线程、JS 引擎线程、定时器触发线程、事件触发线程以及异步 HTTP 请求线程。我们今天所要讨论的事件循环就是事件触发线程的主要职责之一。
　　在浏览器中的事件循环过程如下：
1. 在所有的宏任务队列中选择一个最早进入的任务，如果没有可选的任务，则调到第 6 步
2. 将上一步选择的任务设置为 current running task
3. 运行被选择的任务
4. 运行结束后，设置 current running task 为空
5. 将运行过的 task 从任务队列中移除
6. 执行微任务队列中的所有微任务
7. DOM 渲染
8. 回到第一步

### Node 中的事件循环
　　Node 中的事件循环由 libuv 库实现，它为 Node 提供了跨平台、线程池、事件池、异步 IO 等能力，自然也是 event loop 的源泉。虽然在浏览器和 Node 中都实现了事件循环，但是实现手段不同，平台不同，所以也导致差异也是必然的。在 Node 中，事件循环主要有六个阶段：Timers、I/O callbacks、idle/prepare、poll、check、close callbacks。每一次事件循环都要跑完这六个阶段，每个阶段都有自己的回调函数队列，事件循环每进入一个阶段，就会执行里面所有的操作，直到队列为空或者回调函数执行数量达到最大限制，然后会清理微任务队列，再进入下一个阶段。
- 阶段一：Timers。执行满足条件的 setTimeout，setInterval 回调。该阶段的队列我们称为 Timers Queue。
- 阶段二：I/O callback。执行已完成的 I/O 操作的回调函数。该阶段的队列我们称为 I/O Queue。
- 阶段三：idle，prepare（此阶段只在内部使用）
- 阶段四：poll：获取新的 I/O 事件，适当的条件下 node 将阻塞在该阶段。该阶段在 I/O Queue 的范围内。
- 阶段五：check：执行 setImmediate 的回调。该阶段的队列我们称为 Check Queue。
- 阶段六：close callback。执行一些 onclose 事件的回调。该阶段的队列我们称为 Close Queue。

##### 循环之前
　　在进入第一次循环之前，会先进行如下操作：
- 同步任务
- 发出异步请求
- 规划定时器的生效时间
- 执行 process.nextTick()

##### 开始循环
　　按照我们的循环的6个阶段依次执行，每次拿出当前阶段中的全部任务执行，然后清空微任务队列（先清空 process.nextTick()，再清空其余微任务）。再执行下一阶段，全部6个阶段执行完毕后，进入下轮循环。即：
- 清空当前循环内的 Timers Queue，清空微任务队列
- 清空当前循环内的 I/O Queue，清空微任务队列
- 清空当前循环内的 Check Queue，清空微任务队列
- 清空当前循环内的 Close Queue，清空微任务队列
- 进入下轮循环

##### 伪代码
```
while (true) {
  loop.forEach((阶段) => {
    阶段全部任务()
    nextTick全部任务()
    microTask全部任务()
  })
  loop = loop.next
}
```

### 课后题
```
setTimeout(() => console.log('setTimeout1'), 0);
setTimeout(() => {
    console.log('setTimeout2');
    Promise.resolve().then(() => {
        console.log('promise3');
        Promise.resolve().then(() => {
            console.log('promise4');
        })
        console.log(5)
    })
    setTimeout(() => console.log('setTimeout4'), 0);
}, 0);
setTimeout(() => console.log('setTimeout3'), 0);
Promise.resolve().then(() => {
    console.log('promise1');
});
```
　　浏览器中的事件循环过程如下：
<img src="https://haitao.nos.netease.com/fdb31d95-19be-46fa-a7a7-e9b9fa7950f5_1292_1084.jpg" style="width: 70%;" />
　　Node 中的事件循环过程如下：
<img src="https://haitao.nos.netease.com/bea1944b-28e2-4c51-9bf1-5486c052ec12_1260_1110.jpg" style="width: 70%;" />

### 参考资料
1. [从浏览器多进程到JS单线程，JS运行机制最全面的一次梳理](https://juejin.im/post/5a6547d0f265da3e283a1df7#heading-18)
2. [浏览器和 Node 中不同的Event Loop](https://docs.google.com/presentation/d/1XkyQcB9c9DCrscvab2WxdfHz2SV_D9yDUwSFSHeM32A/edit#slide=id.g38844fb25d_0_265)
3. [浏览器和Node不同的事件循环（Event Loop）](https://juejin.im/post/5aa5dcabf265da239c7afe1e)
4. [node中的Event模块(上）](https://github.com/SunShinewyf/issue-blog/issues/34#issuecomment-371106502)