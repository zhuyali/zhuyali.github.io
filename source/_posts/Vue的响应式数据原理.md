---
title: Vue的响应式数据原理
date: 2018-04-26 23:31:39
tags:
---

　　Vue 是一个开源的 MVVM 框架，实现了数据和视图的双向绑定，任意一方改变，也会引起另一方的变化。以下是我结合源码和一些网上的文章，做的一份关于响应式数据实现的笔记~

### 设计模式
　　在本文开头，我想先把响应式数据的大概原理说一下，因为后续的讲解可能比较杂乱，先理解了思想再看后面的部分，能够减少疑惑。Vue 采用数据劫持结合发布者-订阅模式的方式来实现数据的响应式。主要涉及的概念有以下三种：
- Observer：数据的观察者。在 Observer 中，通过 Object.defineProperty 来劫持属性的 set 和 get，然后在 get 中由发布者收集订阅者，在 set 中由发布者通知订阅者
- Dep：数据更新的发布者。订阅者由发布者进行收集和通知，所以在 Observer 中，要定义一个发布者，以便在数据劫持中收集和通知订阅者
- Watcher：数据更新的订阅者。当数据更新时，执行响应的回调函数

![](https://upload-images.jianshu.io/upload_images/5246378-67fc2d41cd9aee87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

### 从简单开始
　　data 是一个对象，其中的值都是简单数据类型。比如下面这个非常简单的 Vue 实例，在浏览器控制台中动态改变 name 的值，都会看到对应的视图也发生了改变。那么 Vue 底层是如何监视到这种变化，并且将其用于改变的呢？
```
var vm = new Vue({
  el: '#app',
  template: '<p>Hi, {{ name }}</p>',
  data: {
    name: 'Julia',
  }
});
```

### 追根溯源

#### Observer
　　在`/core/instance/state.js`文件中，可以看到有一个函数 initData，是用来初始化数据的，在该函数的最后一行看到`observe(data)`，字面意思来看，大概就是这个 observe 函数在底层监视着 data 数据的变化，并且响应式地改变视图了。
```
function initData (vm: Component) {
  let data = vm.$options.data
  // 获取 data 数据
  data = vm._data = typeof data === 'function'
    ? getData(data, vm)
    : data || {}
  // ...省略部分无关代码
  observe(data, true /* asRootData */)
}
```
　　找到这个 observe 函数在`/core/observer/index.js`文件中，诶..这个目录名叫 observer，那看来这个目录主要就是用来做响应式的，而且它的名字跟观察者模式（一种设计模式）一样，想必是观察者模式的一种实现方式吧。不难看出，这个函数做的事情就是返回一个 data 的观察者，如果 data 本身就有观察者，则返回已有的；如果没有，就新建一个然后返回。
```
export function observe (value: any, asRootData: ?boolean): Observer | void {
  // ...
  let ob: Observer | void
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    ob = value.__ob__
  } else if (
    shouldObserve &&
    !isServerRendering() &&
    (Array.isArray(value) || isPlainObject(value)) &&
    Object.isExtensible(value) &&
    !value._isVue
  ) {
    ob = new Observer(value)
  }
  // ...
  return ob
}
```
　　所以关注点从 observe 函数转到了 Observer 上。Observer 的构造函数主要就做了一件事情：那就是执行了 `this.walk` 函数，walk 函数内部是遍历 data 对象，针对其每一个 key 执行 `defineReactive(obj, keys[i])`。
```
export class Observer {
  value: any;

  constructor (value: any) {
    this.value = value
    // 在 data 上定义 __ob__ 属性，值为当前的观察者
    def(value, '__ob__', this)
    // data 数据是一个对象，所以不走 if 分支
    if (Array.isArray(value)) {
      // ...
    } else {
      this.walk(value)
    }
  }
  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
    }
  }
  // ...
}
```
　　defineReactive 函数的内容如下。它里面主要做了两件事情：一是定义发布者 `const dep = new Dep()`，二是劫持了 data 中每一个属性的 get 和 set。在 get 中，主要部分是 `if(Dep.target) { dep.depend(); }`，根据文初说过的设计模式，这一步应该是在收集订阅者。在 set 中，主要部分是 `dep.notify()`，这一步是通知订阅者。那为什么在收集订阅者之前，要有一个`if(Dep.target)`的判断呢？这是因为执行 get 的场景有很多种，比如在模板或 js 中用到了 data 的值，而这时候是无需收集订阅者的。只有当发布者和订阅者要产生依赖关系时，才需要进行收集。
```
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key]
  }
  // ...
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val      
      if (Dep.target) {
        dep.depend()
        // ...
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      // ...
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      // ...
      dep.notify()
    }
  })
}
```

#### Dep
　　以上，Observer 的工作已经完成了，接下来介绍发布者 Dep。发布者的主要功能就是维持一个订阅者数组，然后在数据发生改变的时候通知这些订阅者。上面数据劫持中调用过 Dep 的两个方法：一是 `depend()` 方法，二是 `notify()`方法，但是其实它们的内部都是调用了 Watcher 的函数，所以接下来看 Watcher 的内部实现。
```
export default class Dep {
  static target: ?Watcher;
  subs: Array<Watcher>;

  constructor () {
    this.subs = []
  }

  addSub (sub: Watcher) {
    this.subs.push(sub)
  }

  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }

  notify () {
    const subs = this.subs.slice()
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}

Dep.target = null
const targetStack = []

export function pushTarget (_target: ?Watcher) {
  if (Dep.target) targetStack.push(Dep.target)
  Dep.target = _target
}

export function popTarget () {
  Dep.target = targetStack.pop()
}
```

#### Watcher
　　在之前的代码中，两次判断到`if(Dep.target)`，那么这个 Dep.target 又是什么时候被赋值的呢？答案就在 Watcher 中。大概浏览一遍代码，可以注意到 get() 函数中有一个 pushTarget 的操作，结合上面 Dep 中给出的 pushTarget 的代码，就会发现这其实就是 target 的赋值操作呀，而 get 函数在 Watcher 的构造函数中调用过一次，所以猜想一定有某处构造过 Watcher...不过这个稍后再说。接着上面 Dep 中的代码说，depend 函数中调用了 `Dep.target.addDep(this)`，而 addDep 函数实际做的事情就是调用 dep.addSub 来在 Dep 维护的订阅者数组中添加一个订阅者。notify 函数中调用了 `subs[i].update()`，而 update 中主要是调用 `queueWatcher(this)`，所以继续追溯这个函数（在`/core/observer/schedular.js`中），发现它执行了 `nextTick(flushSchedulerQueue)`，看到这里大概就懂了，意思应该是在下一次事件循环中应用变化，也就是更新视图。至此，它的大概流程就说完了，不过还是很乱，文末整理了一份简洁版的原理代码。
```
export default class Watcher {
  // 一些变量定义

  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  ) {
    // ...
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
      // 在本文的简单示例中，不会走 else 语句
    }
    if (this.computed) {
      // 在本文的简单示例中，不会走 if 语句
    } else {
      this.value = this.get()
    }
  }

  get () {
    pushTarget(this)
    let value
    try {
      value = this.getter.call(vm, vm)
    } catch (e) {
      // ...
    } finally {
      // ...
      popTarget()
    }
    return value
  }

  addDep (dep: Dep) {
    // ...
    dep.addSub(this)
  }

  update () {
    // ...
    queueWatcher(this)
  }
  // ...
}
```
　　上面提到过应该有某个地方调用过 Watcher 的构造函数，果然，发现在 `/core/instance/lifecycle.js` 中出现了构造 Watcher 的代码，这样我们就知道了 Dep.Target 就是从这里开始被赋值的。
```
new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
```

　　以上介绍了在数据响应式中最重要的三种结构，并且梳理了一下它们之间的关系，会发现一切源于 Object.defineProperty 的拦截器。之所以能够有数据的响应式变化，那么一定在初始化的某个地方调用过数据的 get 函数来为数据添加订阅者。emmm...但是，上面好像并没有提到在哪里调用过 get 诶，这里呢说起来有点复杂，因为要追溯的文件非常多，所以就直接贴结论了，以下结论是我通过断点得出的，个人还不太理解，如果你恰好明白，可以留言给我：数据的替换过程是在 render 函数中进行的，具体长这样：
```
(function anonymous(
) {
with(this){return _c('p',[_v("Hi, "+_s(name))])}
})
```

### 梳理和简化
　　因为在源码中涉及了很多逻辑判断，看起来有点杂乱，使得响应式部分的逻辑也显得有些模糊。所以下面整理了一份自己实现的数据响应式，仅用来映射庞大的 Vue 的数据响应式原理，如有不足，敬请见谅。
```
// 观察者
function observe(data) {
  new Observer(data);
}

function Observer(data) {
  this.walk(data);
}

Observer.prototype.walk = function(data) {
  let keys = Object.keys(data)
  for (let i = 0; i < keys.length; i++) {
    defineReactive(data, keys[i])
  }
}

function defineReactive(data, key) {
  let dep = new Dep();
  let value = data[key];
  Object.defineProperty(data, key, {
    get: function () {
      Dep.target.addDep(dep);
      return value;
    },
    set: function (newVal) {
      if (newVal === value) {
        return;
      }
      value = newVal;
      dep.notify();
    }
  })
}

// 一个生成 id 的自增器
let counter = 0;

// 发布者
function Dep() {
  this.id = counter++;
  this.subs = [];
}

Dep.target = null;

Dep.prototype.addSub = function(watcher) {
  this.subs.push(Dep.target);
}

Dep.prototype.notify = function() {
  for (let i = 0; i < this.subs.length; i++) {
    this.subs[i].update();
  }
}

// 订阅者
function Watcher(data, fn) {
  this.update = fn;
  this.depIds = [];
  Dep.target = this;
  let keys = Object.keys(data)
  for (let i = 0; i < keys.length; i++) {
    data[keys[i]];
  }
}

Watcher.prototype.addDep = function(dep) {
  let id = dep.id;
  if (this.depIds.indexOf(id) === -1) {
    this.depIds.push(id);
    dep.addSub(this);
  }
}

// 调用
var data = {
  a: 1,
  b: 2
}
observe(data);
new Watcher(data, function() {
  console.log('数据发生变化了~');
});
```

### 参考资料
1. [Vue源码解读一：Vue数据响应式原理](https://www.jianshu.com/p/1032ecd62b3a)
2. [Vue2.1.7源码学习](http://hcysun.me/2017/03/03/Vue%E6%BA%90%E7%A0%81%E5%AD%A6%E4%B9%A0/)