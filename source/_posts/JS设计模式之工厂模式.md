---
title: JS设计模式之工厂模式
date: 2018-01-16 17:50:03
tags:
---

### 简单工厂模式

![](//wx3.sinaimg.cn/mw690/79b5b053gy1fnilnnymb4j20l007xmxx.jpg)　　设想一个场景：你是一家水杯生产厂商，你的工厂中生产的水杯种类很多，比如塑料杯、玻璃杯以及其它。这种问题适合使用简单工厂模式解决：
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

### 抽象工厂模式

![](//wx3.sinaimg.cn/mw690/79b5b053gy1fnilno2jknj20mb0algmy.jpg)　　还是上面的场景，但是你打算把你的水杯工厂代理给两个代理商，这两个代理商也代理生产塑料杯和玻璃杯的业务，只不过代理商 A 生产的是宝马牌塑料杯和玻璃杯，代理商 B 生产的是奔驰牌塑料杯和玻璃杯。此时问题的解决方式变为以下：
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
var factoryBM = new concretFactoryBM();
var plasticCupBM = factory.createPlasticCup();
var glassCupBM = factory.createGlassCup();
```