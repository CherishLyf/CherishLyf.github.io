---
layout:     post
title:      "【读书笔记】 单例模式"
subtitle:   "《Javascript 设计模式与开发实践》读书笔记 第二部分 --- 设计模式"
author:     "刘一凡"
header-img: "img/post-bg-2015.jpg"
tags:
    - js
---

# 单例模式

单例模式的定义：保证一个类仅有一个实例，并提供一个访问它的全局访问点。比如，我们单击登录按钮的时候，页面弹出一个登录浮窗，无论单击多少次按钮，这个浮窗只会被创建一次，那么这个浮窗就适合用单例模式。

## 实现单例模式

要实现一个标准的单例模式并不复杂，无非用一个变量来标志当前是否已经为某个类创建过对象，如果是，则在下次获取该类实例时，直接返回之前创建的对象。

```
var Singleton = function (name) {
  this.name = name
  this.instance = null
}

Singleton.prototype.getName = function () {
  alert(this.name)
}

Singleton.getInstance = function (name) {
  if (!this.instance) {
    this.instance = new Singleton(name)
  }

  return this.instance
}

var a = Singleton.getInstance('sven1')
var b = Singleton.getInstance('sven2')

alert(a === b)      // true
```

或者

```
var Singleton = function (name) {
  this.name = name
}

Singleton.prototype.getName = function () {
  alert(this.name)
}

Singleton.getInstance = (function(){
  var instance = null
  return function (name) {
    if (!instance) {
      instance = new Singleton(name)
    }
    return instance
  }
})()
```

## JavaScript 中的单例模式

单例模式的核心是确保只有一个实例，并提供全局访问。

全局变量不是单例模式，但在 JavaScript 开发中，我们经常会把全局变量当成单例来使用。例如：

```
var a = {}
```

全局变量存在很多问题，很容易造成全局污染。有必要尽量减少全局变量的使用，下面有几种方式可以相对降低全局变量的命名污染。

1.使用命名空间

最简单的方法是使用对象字面量的方式：

```
var namespace1 = {
  a: function () {
    alert(1)
  },
  b: function () {
    alert(2)
  }
}
```
另外还可以动态的添加命名空间:

```
var MyApp = {}

MyApp.namespace = function (name) {
  var parts = name.split('.')
  var current = MyApp
  for (var i in parts) {
    if (!current[parts[i]]) {
      current[parts[i]] = {}
    }
    current = current[parts[i]]
  }
}

MyApp.namespace('event')
MyApp.namespace('dom.style')

// 上述代码等价于

var MyApp = {
  event: {},
  dom: {
    style: {}
  }
}
```
2.使用闭包封装私有变量

这种方法把一些变量封装在闭包的内部，只堡垒一些接口跟外界通信。

```
var user = (function(){
  var __name = 'sven',
      __age = 29

      return {
        getUserInfo: function () {
          return __name + '-' + __age
        }
      }
})()
```
