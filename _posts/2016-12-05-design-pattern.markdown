---
layout:     post
title:      "【读书笔记】 闭包和高阶函数"
subtitle:   "《Javascript 设计模式与开发实践》读书笔记 第一部分 --- 基础知识"
author:     "刘一凡"
header-img: "img/post-bg-2015.jpg"
tags:
    - js
    - 设计模式
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

2.`Array.prototype.sort`

`Array.prototype.sort` 接受一个函数作为参数，这个函数里面封装了数组元素的排序规则。

```
// 从小到大

[1, 4, 3].sort(function(a, b){
  return a - b
})

// 输出 [1, 3, 4]

// 从大到小的排序

[1, 4, 3].sort(function(a, b){
  return b - a
})

// 输出 [4, 3, 1]
```

### 函数作为返回值输出

相比把函数当作参数传递，函数当作返回值输出的应用场景也许更多，也更能体现函数式编
程的巧妙。

1.判断数据的类型

比如 `Object.prototype.toString.call([1,2,3])` 总是返回 "[object Array]"，而
`Object.prototype.toString.call('str')` 总是返回 "[object String]"。所以我们可以编写一系列的isType 函数。代码如下：

```
var isString = function (obj) {
  return Object.prototype.toString.call(obj) === '[object String]'
}

var isArray = function (obj) {
  return Object.prototye.toString.call(obj) === '[object Array]'
}

var isNumber = function (obj) {
  return Object.prototype.toString.call(obj) === '[object Number]'
}
```

我们大部分代码都是相同的，不同的只是 `Object.prototype.toString.call(obj)` 返回的字符串。为了避免多余的代码，我们把这些字符串作为参数提前传入 `isType` 函数。

```
var isType = function (type) {
  return function (obj) {
    return Object.prototype.toString.call(obj) === '[object ' + type + ']'
  }
}

var isString = isType('String')
var isArray = isType('Array')
var isNumber = isType('Number')

console.log(isArray([1, 2, 3]))   // 输出 true  
```
我们还可以通过循环语句，来批量注册这些 isType 函数:

```
var Type = {}
for (var i = 0, type; type = ['String', 'Array', 'Number'][i++] ) {
  (function(type){
    Type['is' + type] = function (obj) {
      return Object.prototype.toString.call(obj) === '[object ' + type + ']'
    }
  })(type)
}

Type.isArray([])        // 输出 true
Type.isString('str')    // 输出 true
```

2.getSingle

下面是一个单例模式的例子：

```
var getSingle = function (fn) {
  var ret;
  return funtion () {
    return ret || (ret = fn.apply(this, arguments))
  }
}
```

这个高阶函数的例子，既把函数当做参数传递，又让函数执行后返回了另外的一个函数。下面看下这个函数的效果：

```
var getScript = getSingle(function(){
  return document.createElement('script')
})

var script1 = getScript()
var script2 = getScript()

alert(script1 === script2)    // 输出 true
```

### 高阶函数实现 AOP

AOP(面向切面编程)主要把一些跟核心业务逻辑无关的功能抽离出来，这些跟业务逻辑无关的功能通常包括日志统计、安全控制、异常处理等。

通常，在 Javascript 中实现 AOP，都是指把一个函数“动态织入”到另外的函数中，代码如下：

```
Function.prototype.before = function (beforefn) {
  var __self = this   // 保存原函数的引用

  return function () {    // 返回包含了原函数和新函数的'代理'函数
    beforefn.apply(this, arguments)   // 执行新函数，修正 this
    return __self.apply(this, arguments)    // 执行原函数
  }
}

Function.prototype.after = function (afterfn) {
  var __self = this

  return function () {
    var ret = __self.apply(this, arguments)
    afterfn.apply(this, arguments)
    return ret
  }
}

var func = function () {
  console.log(2)
}

func = func.before(function(){
    console.log(1)
}).after(function(){
  console.log(3)
})

func()    // 依次输出 1, 2 ,3

```

这种使用 AOP 的方式给函数添加职责，也是 JavaScript 语言中一种非常特别和巧妙的装饰者模式实现。

### 高阶函数的其他应用

1.currying

函数柯里化 (function currying)。 `currying` 又称为部分求值。一个 curring 函数首先会接受一些函数，接受这些函数后，该函数并不会立即求值，而是继续返回另外一个函数，刚才传入的参数在函数形成的闭包中被保存起来。待到真正需要求值的时候，之前传入的所有参数都会被一次性用于求值。

下面个例子来理解 currying。

假设我们要编写计算每月开销的函数。在每天结束之前，我们都要记录今天花了多少钱：

```
var monthlyCost = 0

var cost = function (money) {
  monthlyCost += money
}

cost(100)   // 第一天开销
cost(200)   // 第二天开销
cost(300)   // 第三天开销
```

这段代码可以看出，每天结束我们都会记录并计算今天为止花掉的钱。其实我们并不关心每天花了多少，只想知道月底的时候花了多少。

下面代码有助于我们理解其思想：

```
var cost = (function () {
  var args = []

  return function () {
    if (arguments.length === 0) {
      var money = 0
      for (var i = 0, l = args.length; i < l; i++) {
        money += args[i]
      }
      return money
    } else {
      [].push.apply(args, arguments)
    }
  }
})()

cost(100)   // 未真正求值
cost(200)   // 未真正求值
cost(300)   // 未真正求值

console.log(cost())   // 求值并输出: 600
```

接下来我们编写一个通用的 `function curring() {}`，`function curring() {}` 接受一个参数，既要被 `currying` 的函数。在这个例子中，这个函数的作用遍历本月每天的开销并求出他们的总和。

```
var curring = function (fn) {
  var args = []

  return function () {
    if (arguments.length === 0) {
      return fn.apply(this, args)
    } else {
      [].push.apply(args, arguments)
      return arguments.callee
    }
  }
}

var cost = (function(){
    var money = 0;

    return function () {
      for (var i = 0, l = arguments.length; i < l; i++) {
        money += arguments[i]
      }
      return money
    }
})();

var cost = currying(cost)   // 转化成 currying 函数

cost(100)   // 未计算
cost(200)   // 未计算
cost(300)   // 未计算

alert(cost())   // 求值并输出 600
```

2.uncurrying

3.函数节流

函数被频繁调用的场景：

- `window.onresize` 事件
- `mousemove` 事件
- 上传进度

函数节流的原理：

上面提到的场景，都是函数被处罚的频率太高了，这就需要我们按照时间段来忽略掉一些事件请求。

函数节流的实现：

关于函数节流的代码实现有许多种，下面的 `throttle` 函数的原理，将即将被执行的函数用 `setTimeout` 延迟一段时间执行。`throttle` 函数接受 2 个参数，第一个为需要被延迟执行的函数，第二个参数为延时执行的时间。

```
var throttle = function (fn, interval) {

  var __self = fn,    // 保存需要被延迟执行的函数引用
      timer,          // 定时器
      firstTime = true;   // 是否是第一次调用

  return function () {
    var args = arguments,
        __me = this

    if (firstTime) {    // 如果是第一次调用，不需要延迟执行
      __self.apply(__me, args)
      return firstTime = false
    }

    if (timer) {    // 如果定时器还在，说明前一次延时执行还没完成
      return false
    }

    timer = setTimeout(function () {    // 延迟一段时间执行
      clearTimeout(timer)
      timer = null
      __self.apply(__me, args)
    }, interval || 500)
  }
}

window.onresize = throttle(function(){
  console.log(1)
}, 500)
```
