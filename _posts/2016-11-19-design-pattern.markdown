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
