---
title: JavaScript高级程序回顾(五)——面向对象的JS
date: 2018-01-12 18:03:24
tags:
---

### 对象的构造

#### 字面量方式
　　使用对象字面量形式是一种常见的构造对象的方式，不再多说。如下：
```
var person = {
  name: 'Julia',
  age: 23,
  address: {
    prov: 'Zhejiang',
    city: 'hangzhou'
  },
  'test attr': 1, //当属性名不是有效标识符时，需要给属性名加上''
  sayHi: function(name) {
    console.log(`Hi, ${name}, I'm ${this.name}`);
  }
};
```

#### 构造器方式
　　类似于 Java 中使用构造器进行对象构造的形式，JS 中也可以用构造器的方式来构造对象。如下：
```
function Person(name, age, address, test) {
  this.name = name;
  this.age = age;
  this.address = address;
  this['test attr'] = test;
}
Person.prototype.sayHi = function(name) {
  console.log(`Hi ${name}, I'm ${this.name}`);
}
var person = new Person('Julia', 23, { prov: 'Zhejiang', city: 'hangzhou' }, 1);
```
　　这是常用的一种构造对象的方式，它组合了构造器的方式和原型的方式。构造器主要用于构造对象自身有的属性，比如本例中的姓名、年龄等；原型方式主要用于构造对象之间公有的特征，比如 sayHi 方法。最后，通过 new 操作符构造出了 Person，以这种方式调用构造函数实际会经历如下四个步骤：
1. 创建一个新对象
2. 将构造函数的作用域赋给新对象(因此 this 就指向了这个新对象)
3. 执行构造函数中的代码(为这个新对象添加属性)
4. 返回新对象

#### Object.create 方法
　　使用这种方法可以为你要创建的对象指定其原型，而不用定义一个构造函数。如下：
```
var person = {
  name: 'Julia',
  age: 23,
  address: {
    prov: 'Zhejiang',
    city: 'hangzhou'
  },
  'test attr': 1, //当属性名不是有效标识符时，需要给属性名加上''
  sayHi: function(name) {
    console.log(`Hi, ${name}, I'm ${this.name}`);
  }
};
var person1 = Object.create(person);
```
　　以上代码构造了一个 person1 对象，它是以 person 对象作为其原型的。换言之，如果我使用 Object.create(person) 构造了 person2，person3...很多个对象，它们都会拥有 person 对象上的所有属性和方法。

### 原型与原型属性
　　原型指的是 prototype，原型属性指的是 \__proto\__。研究生课程中老师说过的一句话，我觉得可以深刻地解释它们分别是什么：prototype 代表胚胎，\__proto\__ 代表父亲。这句话很值得挖掘。先说 prototype，它代表的是胚胎，意味着它是可以用来产生新的对象的，可以不基于已有对象，并且用来产生新对象的方法是通过使用函数，所以 prototype 属性是函数才有的(这是个人看法，这样理解会变得十分简单)。而所有的对象都有父亲，所以每个对象都有 \__proto\__ 属性。为了更清楚的理解这两个属性，这里结合以上三种构造对象的方式来进行解释。

#### 字面量方式
```
var a = {};
```
![](//wx4.sinaimg.cn/mw690/79b5b053gy1fne3p7cpj5j20el06rjrz.jpg)

#### 构造器方式
```
var A = function() {};
var a = new A();
```
![](//wx4.sinaimg.cn/mw690/79b5b053gy1fne3p7505lj20ef06xgm4.jpg)

#### Object.create 方法
```
var a1 = {};
var a2 = Object.create(a1);
```
![](//wx3.sinaimg.cn/mw690/79b5b053gy1fne3p7bzh9j20eq06v3yw.jpg)

### 原型链
　　原型链是 JS 中实现继承的主要方式。原型链这个词中的原型二字，指的实际上是 \__proto\__ 属性而非 prototype 属性。可以这么理解，JS 中万物皆对象，而 \__proto\__ 是所有对象都有的属性，prototype 只是函数才有的属性，所以 prototype 是无法形成链条的，而 \__proto\__ 是可以做到的。下面一张图，可以清楚地展示出原型链。
```
var A = function() {};
var a = new A();
```
![](https://wx2.sinaimg.cn/mw690/79b5b053gy1fne3v9tn5sj20la053q3p.jpg)
　　从这张图我们可以看出一条由 \__proto\__ 链起来的链条，这就是原型链。在 JS 中，Object 是所有对象的基类，这一点我们也可以从图中观察到，null 总是位于原型链的顶端，Object 仅次于 null。当 JS 引擎查找属性时，会先查找对象本身是否存在该属性，如果不存在，会在原型链上查找，直至回溯到原型链的顶端。

### 参考资料
[三张图搞懂JavaScript的原型对象与原型链](http://www.cnblogs.com/shuiyi/p/5305435.html)
