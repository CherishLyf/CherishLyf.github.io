---
layout:     post
title:      "【读书笔记】 单例模式"
subtitle:   "《Javascript 设计模式与开发实践》读书笔记 第二部分 --- 设计模式"
author:     "刘一凡"
header-img: "img/post-bg-2015.jpg"
tags:
    - js
    - 设计模式
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

## 惰性单例

惰性单例指的是在需要的时候才创建对象实例，这个在实际开发中相当重要。instance 实例对象总是在我们调用 Singleton.getInstance 的时候被创建，而不是在页面加载时候创建：

```
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

比如一个登录浮窗，很明显这个浮窗在页面是唯一的，不能同时存在两个登录窗。

第一种解决方案是创建好这个 `div` 弹窗，这个浮窗一开始肯定是隐藏状态，当用户登录按钮才显示：

```
<html>
  <body>
    <button id="loginBtn">登录</button>
  </body>

  <script>
    var loginLayer = (function(){
      var div = document.createElement('div')
      div.innerHTML = '我是登录浮窗'
      div.style.display = 'none'
      return div
    })();

    document.getElementById('loginBtn').onclick = function () {
      loginLayer.style.display = 'block'
    }
  </script>
</html>
```

上面这种方法有一个问题，这个浮窗一开始就被创建好，那么很有可能白白浪费了一些 DOM 节点。

下面改写一下，点击按钮才创建：

```
<html>
  <body>
    <button id="loginBtn">登录</button>
  </body>
  <script>
    var createLoginLayer = function () {
      var div = document.createElement('div')
      div.innerHTML = '登录浮窗'
      div.style.display = 'none'
      document.body.appendChild(div)
      return div
    }

    document.getElementById('loginBtn').onclick = function () {
      var loginLayer = createLoginLayer()
      loginLayer.style.display = 'block'
    }
  </script>
</html>
```

这样虽然达到惰性的目的，但是每次点击都会新创建一个浮窗，失去了单例的效果。

我们可以通过一个变量来判断是否已经创建过浮窗：

```
var createLoginLayer = (function () {
  var div;

  return function () {
    if (!div) {
      div = document.createElement('div')
      div.innerHTML = '登录浮窗'
      div.style.display = 'none'
      document.body.appendChild(div)
    }
    return div
  }
})()

document.getElementById('loginBtn').onclick = function () {
  var loginLayer = createLoginLayer()
  loginLayer.style.display = 'block'
}
```

## 通用的惰性单例

上面我们创建了一个可用惰性单例，但是还有一些问题，

- 这段代码是违法单一职责的，创建对象和管理单例的逻辑都在 `createLoginLayer` 对象内部
- 如果要创建页面唯一的 iframe 或者其他标签，就必须把 `createLoginLayer` 函数照抄一遍

```
var createIframe = (function () {
  var iframe ;

  return function () {
    if (!iframe) {
      iframe = document.createElement('iframe')
      iframe.style.display = 'none'
      document.body.appendChild(iframe)
    }
    return iframe
  }
})()
```
我们需要把不变的部分隔离开来，不管创建的是 div 还是 iframe，管理单例的逻辑是一样的：用一个变量来标志是否创建过的对象，如果是直接返回这个对象。

```
var obj;
if (!obj) {
  obj = xxx
}
```
我们把管理单例的逻辑抽离出来封装在 getSingle 函数内部，创建对象的方法 fn 被当做参数传入 getSingle 函数：

```
var getSingle = function (fn) {
  var result;

  return function () {
    return result || (result = fn.apply(this, arguments))
  }
}
```

我们将创建浮窗的方法用参数 fn 的方式传入 getSingle，不仅可以传入 createLoginLayer ，还能传入 createIframe 等。并用 getSingle 返回一个新的函数，并且用变量 result 来保存 fn 的计算结果。result 变量因为在闭包中，它永远不会销毁，在将来的请求中，如果 result 已经赋值，那么它将返回这个值。

```
var createLoginLayer = function () {
  var div = document.createElement('div');
  div.innerHTML = '登录浮窗';
  div.style.display = 'none';
  document.body.appendChild(div);
  return div;
}

var createSingleLoginLayer = getSingle(createLoginLayer)

document.getElementById('loginBtn').onclick = function () {
  var loginLayer = createSingleLoginLayer();
  loginLayer.style.display = 'block';
}
```

## 小结

单例模式是一种非常简单但非常实用的模式，特别是惰性单例技术，在合适的时候创建对象，并且只创建唯一的对象。
