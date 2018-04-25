---
title: JS设计模式-创建型模式
date: 2018-04-25 12:58:46
tags:
---
　　创建型设计模式专注于处理对象创建机制，以适合给定情况的方式来创建对象。创建对象的基本方式可能导致项目复杂性增加，而这些模式旨在通过控制创建过程来解决这种问题。

### 构造器模式（Constructor）
　　在《JavaScript设计模式》一书中将构造器模式纳入设计模式之中，而四人帮提出的设计模式中不包含构造器模式，不过想要如何划分取决于个人的想法，我比较偏向于前者。其实构造器模式在我们的日常编码工作中处处可见，这已经是一种类似于规范的模式。在面向对象编程思想中，我们通过“创建构造函数，然后 new 构造函数”的方式来创建对象，这是一种非常普遍的用法，这里不再赘述。

### 工厂模式（Factory）
![](//wx3.sinaimg.cn/mw690/79b5b053gy1fnilnnymb4j20l007xmxx.jpg)　　工厂模式的适用场景是，需要动态的创建对象。它需要根据不同的传入参数或者程序运行的上下文来创建不同的对象。设想一个场景：你是一家水杯生产厂商，你的工厂中生产的水杯种类很多，比如塑料杯、玻璃杯以及其它。这种问题适合使用简单工厂模式解决：
```
//抽象工厂的接口
var AbstractFactory = function() {};
AbstractFactory.prototype = {
  createPlasticCup: function() {}, //生产抽象塑料杯
  createGlassCup: function() {} //生产抽象玻璃杯
};

//抽象水杯的接口
var Cup = function() {};
//塑料水杯的实现
var PlasticCup = function() {};
PlasticCup.prototype = Object.create(Cup.prototype);
//玻璃水杯的实现
var GlassCup = function() {};
GlassCup.prototype = Object.create(Cup.prototype);

//具体的工厂实现对象
var concretFactory = function() {};
concretFactory.prototype = Object.create(AbstractFactory.prototype);
concretFactory.prototype.createPlasticCup = function() {
  return new PlasticCup();
};
concretFactory.prototype.createGlassCup = function() {
  return new GlassCup();
};

//调用端
var factory = new concretFactory();
var plasticCup = factory.createPlasticCup();
var glassCup = factory.createGlassCup();
```

### 抽象工厂模式（Abstract Factory）
![](//wx3.sinaimg.cn/mw690/79b5b053gy1fnilno2jknj20mb0algmy.jpg)　　抽象工厂模式用于封装一组具有共同目标的单个工厂，在工厂模式的基础上新增了家族的概念。以上面举的例子为基础扩展，你打算把你的水杯工厂代理给两个代理商，这两个代理商也代理生产塑料杯和玻璃杯的业务（此时两个代理商对应的水杯工厂具有共同的目标：生产塑料杯和玻璃杯），只不过代理商 A 生产的是宝马牌塑料杯和玻璃杯（宝马就是一个家族），代理商 B 生产的是奔驰牌塑料杯和玻璃杯（奔驰也是一个家族），此时问题的解决方式变为以下：
```
//抽象工厂的接口
var AbstractFactory = function() {};
AbstractFactory.prototype = {
  createPlasticCup: function() {}, //生产抽象塑料杯
  createGlassCup: function() {} //生产抽象玻璃杯
};

//塑料抽象水杯的接口
var PlasticCup = function() {};
//塑料水杯的实现
var PlasticCupBM = function() {};
PlasticCupBM.prototype = Object.create(PlasticCup.prototype);
var PlasticCupHD = function() {};
PlasticCupHD.prototype = Object.create(PlasticCup.prototype);

//玻璃抽象水杯的接口
var GlassCup = function() {};
//玻璃水杯的实现
var GlassCupBM = function() {};
GlassCupBM.prototype = Object.create(GlassCup.prototype);
var GlassCupHD = function() {};
GlassCupHD.prototype = Object.create(GlassCup.prototype);

//具体的BM工厂实现对象
var concretFactory = function() {};
concretFactoryBM.prototype = Object.create(AbstractFactory.prototype);
concretFactoryBM.prototype.createPlasticCup = function() {
  return new PlasticCupBM();
};
concretFactoryBM.prototype.createGlassCup = function() {
  return new GlassCupBM();
};
//具体的HD工厂实现对象
var concretFactoryHD = function() {};
concretFactoryHD.prototype = Object.create(AbstractFactory.prototype);
concretFactoryHD.prototype.createPlasticCup = function() {
  return new PlasticCupHD();
};
concretFactoryHD.prototype.createGlassCup = function() {
  return new GlassCupHD();
};

//调用端
var factory = new concretFactoryBM();
var plasticCupBM = factory.createPlasticCup();
var glassCupBM = factory.createGlassCup();
```

### 原型模式（Prototype）
![](https://gss2.bdstatic.com/9fo3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=dc176e019c510fb36c147fc5b85aa3f0/d8f9d72a6059252deef03dbc369b033b5ab5b9a6.jpg)　　原型模式通过给出一个原型对象来指明所要创建的对象的类型，然后用复制这个原型对象的办法创建出更多同类型的对象。在 JS 语言中，这个设计模式显得尤其的亲切，因为 JS 中的继承就是通过原型链来完成的。JS 提供了原生的方式 `var b = Object.create(a)`来实现继承，其中 b 继承自 a，其底层其实就是通过原型来完成的。以下是 `Object.create()` 方法的一种可替代品：
```
var beget = (function() {
  function F() {}
  return function(proto) {
    F.prototype = proto;
    return new F();
  }
})();
```

### 单例模式（Singleton）
![](https://gss1.bdstatic.com/9vo3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike72%2C5%2C5%2C72%2C24/sign=ad9321205cdf8db1a8237436684ab631/f703738da977391242e8a5d9fa198618377ae2b8.jpg)　　单例模式确保一个类只有一个实例，并且提供一个单一访问点。单例模式的实现很简单：
```
const singleton = (function() {
  var instance;
  function init() {
    var privateMethod = function() {
      console.log('This is a private method');
    };
    var privateVariable = 'This is a private variable';
    return {
      publicMethod: function() {
        privateMethod();
        console.log(privateVariable);
      },
      publicVariable: 'This is a public variable'
    };
  }
  return {
    getInstance: function() {
      if (!instance) {
        instance = init();
      }
      return instance;
    }
  }
})();
```
　　以上代码实现了一个简单的单例模式，可以通过 `singleton.getInstance()` 来获取唯一的实例。上面的代码中有很多跟单例模式无关的代码，主要是为了展示一下私有变量和公有变量的访问。

### 建造者模式（Builder）
![](https://gss3.bdstatic.com/-Po3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=1a5ca9b6b33533fae1bb9b7cc9ba967a/9f2f070828381f30eaa0ad11ab014c086f06f0c8.jpg)　　建造者模式将产品的结构和产品的零件建造过程对客户端隐藏起来，把对建造过程进行指挥的责任和具体建造零件的责任分割开来，达到责任划分和封装的目的。比如说构建一个人分为构建头部，身体和脚三个部分，但是用户无需知道构建过程。代码实现如下：
```
var Director = function(builder) {
  this.builder = builder;
};
Director.prototype.construct = function() {
  this.builder.buildHead();
  this.builder.buildBody();
  this.builder.buildFoot();
};

var Builder = function() {};
Builder.prototype = {
  buildHead: function() {},
  buildBody: function() {},
  buildFoot: function() {}
};

var concreteBuilder = function() {};
concreteBuilder.prototype = Object.create(Builder.prototype);
concreteBuilder.prototype.buildHead = function() {
  console.log('构建头部');
};
concreteBuilder.prototype.buildBody = function() {
  console.log('构建身体');
};
concreteBuilder.prototype.buildFoot = function() {
  console.log('构建脚');
};

const director = new Director(new concreteBuilder());
director.construct();
```

