---
layout:     post
title:      "【读书笔记】 this 、call 和 apply"
subtitle:   "《Javascript 设计模式与开发实践》读书笔记 第一部分 --- 基础知识"
author:     "刘一凡"
header-img: "img/post-bg-2015.jpg"
tags:
    - js
---

# `this` 、`call` 和 `apply`

## `this`

`this` 总是指向一个对象，具体只想哪个基于函数的`运行环境`动态绑定，而不是声明环境。

### `this` 的指向

实际应用中，`this` 大多分为一下 4 种：

- 作为对象的方法调用
- 作为普通函数调用
- 构造器调用
- `Function.prototype.call` 或者 `Function.prototype.call`

1.作为方法调用

作为方法调用时，`this` 指向该对象：

```
var obj = {
  a: 1,
  getA: function () {
    alert(this === obj)   // 输出 true
    alert(this.a)    // 输出 1
  }
}
```

2.作为普通函数调用

当函数不作为对象的属性被调用时，`this` 指向全局对象。在浏览器的 Javascript 里，这个全局对象就是 `window` 对象。

```
window.name = 'globalName'

var getName = function () {
  return this.name
}

console.log(getName())  // 输出 globalName
```

或者 :

```
window.name = 'globalName'

var myObject = {
  name: 'sven',
  getName: function () {
    return this.name
  }
}

var getName = myObject.getName()

console.log(getName)    // 输出 globalName
```

3.构造器调用

大部分 Javascript 函数都可以作为构造函数使用。当使用 `new` 调用函数时，改函数总是返回一个对象，通常情况下，构造器里的 `this` 指向返回的这个对象。

```
var MyClass = function () {
  this.name = 'sven'
}

var obj = new MyClass()
alert(obj.name)   // 输出: sven
```

但要注意，构造器显式的返回一个object 类型的对象，那么此次的运算结果最终会返回这个对象，而不是`this`：

```
var MyClass = function () {
  this.name = 'sven'
  return {    // 显式的返回一个对象
    name: 'anne'
  }
}

var obj = new MyClass()
alert(obj.name)   // 输出 anne  
```

如果构造器不显式的返回任何数据，而是返回一个非 object 类型数据，就不会出现上面的问题：

```
var MyClass = function () {
  this.name = 'sven'
  return 'anne'   // 返回 String 类型
}

var obj = new MyClass()
alert(obj.name)   // 输出 sven  
```

4.`Function.prototype.call` 或 `Function.prototype.apply` 调用

`Function.prototype.call` 或 `Function.prototype.apply` 可以动态地改变传入函数的 `this`

```
var obj1 = {
  name: 'sven',
  getName: function () {
    return this.name
  }
}

var obj2 = {
  name: 'anne'
}

console.log(obj.getName())    // sven
console.log(obj1.getName.call(obj2))    // anne
```

## `call` 和 `apply`

### 区别

`Function.prototype.call` 和 `Function.prototype.apply`，作用一样，区别在于传入的参数不同。

`apply` 接受两个参数，第一个参数指定了函数体内 `this` 对象的指向，第二参数为一个带有下标的集合，可以为数组，也可以为类数组，`apply` 方法把这个集合中的元素作为参数传递给被调用的函数。

`call` 参数数量不固定，第一个参数也是代表函数体内的 `this` 指向，从第二个参数开始往后，每个参数被依次传入函数。

使用 `call` 或者 `this` 时，当传入的第一个参数为 `null` 时，函数体内的 `this` 会指向默认的宿主对象，在浏览器中则是 `window`

### 用途

1.改变 `this` 指向

`call` 和 `call` 最常见的用途就是改变内部 `this` 指向：

```
var obj1 = {
  name: 'sven'
}

vr obj2 = {
  name: 'anne'
}

window.name = 'window'

var getName = function () {
  alert(this.name)
}

getName()   // 输出 window
getName.call(obj1)    // 输出 sven
getName.call(obj2)    // 输出 anne
```

实际开发中，比如一个 div 节点，在 `onClick` 事件中的 `this` 指向这个 div:

```
document.getElementById('div1').onClick = function () {
  alert(this.id)    // div1
}
```

假设事件函数内部有一个内部函数 func，func 函数体内的 `this` 就指向了 `window`

```
document.getElementById('div1').onClick = function () {
  alert(this.id)    // div1
  var func = function () {
    alert(this.id)    // 输出 undefined
  }
  func()

}
```

这个时候可以使用 `call` 来修正 `this`，使其依旧指向 div:

```
document.getElementById('div1').onClick = function () {
  alert(this.id)    // div1
  var func = function () {
    alert(this.id)    // 输出 undefined
  }
  func.call(this)
}
```

2.`Function.prototype.bind`

大部分浏览器都实现了内置的 `Function.prototype.bind` 来指定函数内部的 `this` 指向，即使没有，我们也可以来模拟：

```
Function.prototype.bind = function (context) {
  var self = this   // 保存原函数
  return function () {    // 返回一个新函数
    return self.apply(context, arguments)   
    // 执行新函数的时候，会把之前传入的context 当做新的 函数体内的 this
  }
}

var obj = {
  name: 'sven'
}

var func = function () {
  alert(this.name)    // 输出 sven
}.bind(obj)

func()
```

上面是一个简单版的 `bind`，通常我们需要稍微复杂一点的，使得可以往 func 函数中预填一些参数

```
Function.prototype.bind = function () {
  var self = this,  // 保存原函数
      context = [].shift.call(arguments),   // 需要绑定的 this 上下文
      args = [].slice.call(arguments);    // 剩余的参数转换成数组

      return function () {    // 返回一个新的函数
        return self.apply(context, [].concat.call(args, [].slice.call(arguments)))
      }
      // 执行新的函数的时候，会把之前传入的 context 当做新函数体内的 this
      // 并且组合两次分别传入的参数，作为新函数的参数(书中这里写的特别好)
}

var obj = {
  name: 'sven'
}

var func = function (a, b, c, d) {
  alert(this.name)
  alert([a, b, c, d])
}.bind(obj, 1, 2)

func(3, 4)
```

3.借用其他对象的方法

借用方法的第一种场景是 "借用构造函数"，通过这种技术可以实现一种类似继承的效果：

```
var A = function (name) {
  this.name = name
}

var B = function () {
  A.apply(this, arguments)
}

B.prototype.getName = function () {
  return this.name
}

var b = new B('sven')
console.log(b.getName())    // 输出 'sven'
```

借用方法的第二种场景跟我们关系密切:

```
(function(){
  Array.prototype.push.call(arguments, 3)
  console.log(arguments)  // 输出 [1, 2, 3]
})(1, 2)
```
