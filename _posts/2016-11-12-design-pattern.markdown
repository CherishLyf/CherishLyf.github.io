---
layout:     post
title:      "【读书笔记】 面向对象的 JavaScript"
subtitle:   "《Javascript 设计模式与开发实践》读书笔记 第一部分 --- 基础知识"
author:     "刘一凡"
header-img: "img/post-bg-2015.jpg"
tags:
    - js
---

# 面向对象的 JavaScript

Javascript 没有传统面向对象语言的类式继承，而是通过原型委托的方式实现对象与对象之间的继承。

## 动态语言类型和鸭子类型

编程语言按照数据类型可分为，静态类型语言和动态类型语言。

静态语言在编译时已经确定变量类型，动态类型语言要在程序运行时待变量被赋予值之后，才会具有某种类型。Javascript 是一种典型的动态语言。动态类型语言对变量的宽容给实际代码带来很大的灵活性。

这一切都建立在鸭子类型（duck typing）的概念上，鸭子类型的通俗说法：“如果它走起来路像只鸭子，叫起来也是鸭子，那么它就是鸭子。”

通过一个小故事来理解鸭子模型：

   从前在 JavaScript 王国里,有一个国王,他觉得世界上最美妙的声音就是鸭子的叫
声,于是国王召集大臣,要组建一个 1000 只鸭子组成的合唱团。大臣们找遍了全国,
终于找到 999 只鸭子,但是始终还差一只,最后大臣发现有一只非常特别的鸡,它的叫
声跟鸭子一模一样,于是这只鸡就成为了合唱团的最后一员。

用代码来模拟这个故事：

```
var duck = {
  duckSinging: function () {
    console.log('嘎嘎嘎')
  }
}

var chicken = {
  duckSinging: function () {
    console.log('嘎嘎嘎')
  }
}

var choir = [];   // 合唱团

var joinChoir = function (animal) {
  if (animal && animal.duckSinging === 'function') {
    choir.push(animal)
    console.log('恭喜加入合唱团')
    console.log('合唱已有成员数量: ' + choir.length)
  }
}

joinChoir(duck)
joinChoir(chicken)
```

我们看到加入合唱团的动物，不在乎它是什么类型，只要保证拥有 `duckSinging` 方法就行。<br/>
利用鸭子类型的思想，我们不必借用超类型的帮助，就能轻松在动态语言中实现一个原则：`“面向接口编程，而不是面向过程编程。”`

## 多态

多态的含义是：同一操作作用于不同的对象上，可以产生不同的解释和不同的执行结果。换句话说就是不同的对象发送同一个消息时，这些对象会根据这个消息发出不同的反馈。

举个例子说明下多态，

   主人家里养了两只动物,分别是一只鸭和一只鸡,当主人向它们发出“叫”的命令
时,鸭会“嘎嘎嘎”地叫,而鸡会“咯咯咯”地叫。这两只动物都会以自己的方式来发
出叫声。它们同样“都是动物,并且可以发出叫声”,但根据主人的指令,它们会各自
发出不同的叫声。

用代码描述：

```
var makeSound = function (animal) {
  if (animal instanceof Duck) {
    console.log('嘎嘎')
  } else if (animal instanceof Chicken) {
    console.log('咯咯')
  }
}

var Duck = function () {};
var Chicken = function () {};

makeSound(new Duck());    // 嘎嘎
makeSound(new Chicken());   // 咯咯
```

这样的代码确实体现了多态性，但是如果需要增加一种动物时，必须要改动 `makeSound` 函数，并且添加的动物越多， `makeSound` 函数将变得不可维护。

多态背后的思想是将 “做什么” 和 “谁去做以及怎么做” 分离开，也就是将 “不变的事物” 和 “可能改变的事物” 分离开。在这个故事中，动物都会叫，这是不变的，但是不同的动物具体怎么叫是可变的。`把不变的部分隔离，把可变的部分封装起来`，这给与我们拓展程序的能力。

### 对象的多态性

我们把不变的部分隔离开，那就是所有动物会叫：

```
var makeSound = function (animal) {
  animal.sound()
}
```
然后把可变的部分封装，刚才谈论到的多态性实际上指的是对象的多态性：

```
var Duck = function () {}
Duck.prototype.sound = function () {
  console.log('嘎嘎')
}

var Chicken = function () {}
Chicken.prototype.sound = function () {
  console.log('咯咯')
}

makeSound(new Duck())    // 嘎嘎
makeSound(new Chicken())    // 咯咯
```

如果多了一种动物，只需要简单的追加一些代码就行：

```
var Dog = function () {}
Dog.prototype.sound = function () {
  console.log('汪汪')
}

makeSound(new Dog())    // 汪汪
```

### 多态在面向对象程序设计中的作用

Martin Fowler 在《重构:改善既有代码的设计》里写到:

  多态的最根本好处在于,你不必再向对象询问“你是什么类型”而后根据得到的答
案调用对象的某个行为——你只管调用该行为就是了,其他的一切多态机制都会为你安
排妥当。

换句话说，多态的根本作用是 `通过把过程化的条件分支语句转化为对象的多态性，从而消除这些条件分支语句。`

将行为分布在各个对象中，并让这些对象各自负责自己的行为，这正是面向对象设计的优点。

再看实际开发中遇到的例子，与动物叫声非常类似。

我们要编写一个地图应用，首先选择的是谷歌地图：

```
var googleMap = {
  show: function () {
    console.log('开始渲染谷歌地图')
  }
}

var renderMap = function () {
  googleMap.show()
}

renderMap();    // 开始渲染谷歌地图
```
后来因为一些原因换成百度地图，将 `renderMap` 中添加了一些条件分支，让这个函数同时支持百度地图和谷歌地图：

```
var googleMap = {
  show: function () {
    console.log('开始渲染谷歌地图')
  }
}

var baiduMap = {
  show: function () {
    console.log('开始渲染百度地图')
  }
}

var renderMap = function (type) {
  if (type == 'google') {
    googleMap.show()
  } else if (type == 'baidu') {
    baiduMap.show()
  }
}

renderMap('google')   // 开始渲染谷歌地图
renderMap('baidu')    // 开始渲染百度地图
```

一旦需要替换新的地图，需要继续在 `renderMap` 中添加一些分支，这样后期会变得很难维护。

我们需要把程序中相同的部分抽象出来，那就是显示地图：

```
var renderMap = function (map) {
  if (map.show instanceof Function) {
    map.show()
  }
}

renderMap(googleMap)    // 开始渲染谷歌地图
renderMap(baiduMap)     // 开始渲染百度地图
```

以后添加新的地图就不需要更改 `renderMap`:

```
var sosoMap = {
  show: function () {
    console.log('开始渲染soso地图')
  }
}
```
这个例子假设每个地图 API 展示方法都是 `show`，实际开发中不会如此顺利，还需用到适配器模式来解决问题。

## 封装

### 封装数据

Javascript 没有 `private`，`public` 等关键字，只能通过变量的作用域来实现封装。

除了 ES6 提供的 `let`，我们一般通过函数来创建作用域：

```
var myObject = (function(){
  var _name = 'sven';     // 私有 (private) 变量
  return {
    getName: function () {    // 公开 (public) 方法
      return _name
    }
  }
})();

console.log(myObject.getName())   // 输出 'sven'
console.log(myObject._name)   // 输出 undefined
```

## 原型模式和基于原型继承的 Javascript 对象系统

### 使用克隆的原型模式

比如我们编写一个飞机大战的网页游戏，飞机具有分身技能，分身后创建一个一模一样的飞机。这时候就要用到原型模式，原型模式的关键在于语言是否提供了 `clone` 方法, ES5 提供了 `Object.create` 方法用来克隆对象。

```
var Plane = function () {
  this.blood = 100;
  this.attackLevel = 1;
  this.defenseLevel = 1;
}

var plane = new Plane();
plane.blood = 500;
plane.attackLevel = 10;
plane.defenseLevel = 7;

var clonePlane = Object.create(plane)
console.log(clonePlane);    // {blood: 500, attackLevel: 10, defenseLevel: 7}
```

不支持 `Object.create` 方法的浏览器，可以使用以下代码：

```
Object.create = Object.create || function (obj) {
  var F = function () {}
  F.prototype = obj

  return new F()
}
```

### JavaScript 中的原型继承

圆形编程的基本规则：

- 所有数据都是对象。
- 要得到一个对象，不是通过实例化类，而是找到一个对象作为原型并克隆它。
- 对象会记住它的原型。
- 如果对象无法响应某个请求，它会把这个请求委托给原型。

### 原型继承的未来

ES6 提供了新的 `Class` 语法。通过类创建对象：

```
Class Animal {
  constructor (name) {
    this.name = name;
  }
  getName () {
    return this.name
  }
}

Class Dog extend Animal {
  constructor (name) {
    super(name)
  }
  speak () {
    return 'woof'
  }
}

var dog = new Dog('Scamp')
console.log(dog.getName() + 'says' + dog.speak())
```
