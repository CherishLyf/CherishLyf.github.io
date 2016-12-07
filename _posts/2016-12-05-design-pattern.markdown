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
