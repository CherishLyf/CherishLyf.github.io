---
layout:     post
title:      "【转】JavaScript 奇淫异巧 44 招"
subtitle:   "【转】JavaScript 奇淫异巧 44 招"
author:     "刘一凡"
header-img: "img/post-bg-2015.jpg"
tags:
    - JavaScript
---

# 【转】JavaScript 奇淫异巧 44 招

[原文地址](http://forums.fami2u.com/t/javascript-44/77/1)

### 1. 首次变量赋值时务必使用 var 关键字

变量没有声明而直接赋值得话，默认会作为一个新的全局变量，要尽量避免使用全局变量。

### 2. 使用 === 取代 ==

==和!=操作符会在需要的情况下自动转换数据类型。但===和!==不会，它们会同时比较值和数据类型，这也使得它们要比==和!=快。

```
[10] === 10    // is false
[10]  == 10    // is true
'10' == 10     // is true
'10' === 10    // is false
 []   == 0     // is true
 [] ===  0     // is false
 '' == false   // is true but true == "a" is false
 '' === false  // is false
```

### 3. undefined、null、0、false、NaN、空字符串的逻辑结果均为 false

### 4. 行尾使用分号

### 5. 使用对象构造器

```
function Person(firstName, lastName) {
  this.firstName = firstName
  this.lastName = lastName
}

var Saad = new Person('Saad', 'Mousliki')
```

### 6. 小心使用 typeof、instanceof 和 contructor

- typeof: JavaScript 一元操作符，用于以字符串的形式返回字面量的原始类型，注意，typeOf null 也会返回 object，大多数的对象类型（Array，时间 Date等）也会返回 object
- contructor: 内部原型属性，可以通过代码重写
- instanceof： Javascript 操作符，也会在原型链中的构造函数中搜索，找到则返回 true, 否则返回 false

```
var arr = ['a', 'b', 'c']
typeof arr ;    // 返回 'object'
arr instanceof Array ;    // true
arr.constructor();  // []
```

### 7. 使用自调函数

函数在创建之后直接执行， 通常称为自调用匿名函数

```
(function(){
  // 置于次的代码将自动执行
})()

(function(a, b){
  var result = a + b
  return result
})(10, 20)
```

### 8. 从数组中随机获取成员

```
var item = [12, 548, 'a', 2, 5478, 'foo', 8852, 'Doe', 2145, 119];

var randomItem = items[Math.floor(Math.random() * items.length)]
```

### 9. 获取制定范围内的随机数

这个功能在生成测试用的假数据时特别有用，比如介于指定范围内的工资数

```
var x = Math.floor(Math.random() * (max - min + 1)) + min
```

### 10. 生成从 0 到指定值的数字数组

```
var numbersArray = [],
    max = 100;

for(var i = 0; numbersArray.push(i++) < max;);   
```
