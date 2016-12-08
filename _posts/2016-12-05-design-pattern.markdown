---
layout:     post
title:      "【读书笔记】 闭包和高阶函数"
subtitle:   "《Javascript 设计模式与开发实践》读书笔记 第一部分 --- 基础知识"
author:     "刘一凡"
header-img: "img/post-bg-2015.jpg"
tags:
    - js
---

# 闭包和高阶函数

## 闭包

一个经典的闭包应用：

```
<div>1</div>
<div>2</div>
<div>3</div>
<div>4</div>
<div>5</div>
<script>
  var nodes = document.getElementsByTagName('div')

  for (var i = 0, len = nodes.length; i < len; i++) {
    nodes[i].onclick = function () {
      alert(i)
    }
  }
</script>
```

无论点击哪个 div ，弹出的结果都是 5，因为 click 事件是异步触发的，当事件被触发时，for 循环早结束了，此时 i 的值已经是 5 了。

解决办法就是闭包，把每次循环的 i 都封闭起来，当在事件函数中顺着作用域链从内到外找到变量 i ，会找到被封闭在闭包环境中的 i 。

```
<script>
  var nodes = document.getElementsByTagName('div')

  for (var i = 0, len = nodes.length; i < len; i++) {
    (function(i){
      nodes[i].onclick = function () {
        alert(i)
      }
    })(i)
  }
</script>
```

### 闭包的作用

1.封装变量

2.延续局部变量的寿命

### 闭包与面向对象设计

下面看看这段和闭包有关的的代码：

```
var extent = {
  var value = 0
  return {
    call: function () {
      value++
      console.log(value)
    }
  }
}

var extent = extent()
extent.call()     // 输出 1
exten2.call()     // 输出 2

```
换成面向对象的写法，就是：

```
var extent = {
  value: 0,
  call: function () {
    this.value++
    console.log(this.value)
  }
}

extent.call()   // 输出 1
extent.call()   // 输出 2
```

或者 :

```
var Extent = function () {
  this.value = 0
}

Extent.prototype.call = function () {
  this.value++;
  console.log(this.value)
}

var extent = new Extent()

extent.call()
```

### 用闭包实现命令模式

下面用面向对象的方式编写一段命令模式的代码

```
<html>
  <body>
    <button id="execute">点击我执行命令</button>
    <button id="undo">点击我执行命令</button>
    <script>
    var Tv = {
      open: function () {
        console.log('打开电视机')
      },
      close: function () {
        console.log('关闭电视机')
      }
    }

    var OpenTvCommand = function (receiver) {
      this.receiver = receiver
    }

    OpenTvCommand.prototype.execute = function () {
      this.receiver.open()    // 执行命令，打开电视
    }

    OpenTvCommand.prototype.undo = function () {
      this.receiver.close()   // 撤销命令，关闭电视
    }

    var setCommand = function (command) {
      document.getElementById('execute').onclick = function () {
        command.execute()   // 输出，打开电视机
      }
      document.getElementById('undo').onclick = function () {
        command.undo()      // 输出，关闭电视机
      }
    }

    setCommand( new OpenTvCommand(Tv) )
    </script>
  </body>
</html>
```

命令模式的意图是把请求封装为一个对象，从而分离请求的发起者和请求的接收者（执行者）之间的耦合。在命令对象执行前，可以预先往命令对象中植入命令的接收者。

在面向对象版本的命令模式中，预先植入的命令接收者被当成对象的属性保存起
来；而在闭包版本的命令模式中，命令接收者会被封闭在闭包形成的环境中，代码如下：

```
 var Tv = {
   open: function () {
     console.log('打开电视机')
   },
   close: function () {
     console.log('关上电视机')
   }
 }

var createCommand = function (receiver) {

  var execute = function () {
    return receiver.open()    // 执行命令，打开电视机
  }

  var undo = function () {
    return receiver.close()   // 执行命令，关闭电视机
  }

  return {
    execute: execute,
    undo: undo
  }
}

var setCommand = function (command) {
  document.getElementById('execute').onclick = function () {
    command.execute()   // 输出: 打开电视机
  }
  document.getElementById('undo').onclick = function () {
    command.undo()      // 输出: 关闭电视机
  }
}
```

## 高阶函数

高阶函数至少满足下列条件之一的函数：

- 函数可以作为参数被传递
- 函数可以作为返回值输出

### 函数作为参数传递

1.回调函数

在 AJAX 异步请求中，回调函数的使用非常频繁。最常见的方案就是把 callback 函数当作参数发起 ajax 请求，请求完成后执行 callback 函数。

```
var getUserInfo = function (userId, callback) {
  $.ajax('http://xxx.com/getUserInfo?' + userId, function (data) {
    if (typeof callback === 'function') {
      callback(data)
    }
  })
}

getUserInfo(13157, function (data) {
    alert(data.userName)
})
```

回调函数的应用不仅在异步请求中，当一个函数不适合执行一些请求时，可以把这个请求封装成一个函数，并把它作为参数传递给另外一个函数。

比如，我们想在页面中创建 100 个 div 节点，然后把节点都设置为隐藏。下面是一种编写代码的方式：

```
var appendDiv = function () {
  for (var i = 0; i < 100; i++) {
    var div = document.createElement('div')
    div.innerHTML = i
    document.body.appendChild(div)
    div.style.display = 'none'
  }
}

appendDiv()
```
`div.style.display = 'none'` 的逻辑硬编码在这里是不合理的，我们把这部分抽离出来，用回调函数的形式传入 `appendDiv`

```
var appendDiv = function (callback) {
  for (var i = 0; i < 100; i++) {
    var div = document.createElement('div')
    div.innerHTML = i
    document.body.appendChild(div)
    if (typeof callback === 'function') {
      callback(div)
    }
  }
}

appendDiv(function(node){
  node.style.display = 'none'
})
```
