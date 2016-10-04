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

### 11. 生成随机的字母数字字符串

```
function generateRandomAlphaNum(len) {
  var rdmString = ''

  for(; rdmString.length < len; rdmString += Math.random().toString(36).substr(2);)
  return rdmString.substr(0, len)

}
```

### 12. 打乱数字数组的顺序

```
var numbers = [5, 458, 120, -215, 228, 400, 122205, -85411]

numbers = numbers.sort(function(){ return Math.random() - 0.5 })
```
这里使用了JavaScript内置的数组排序函数，更好的办法是用专门的代码来实现（如Fisher-Yates算法），可以参见StackOverFlow上的这个讨论。

### 13. 字符串去空格

```
String.prototype.trim = function() {
  return this.replace(/^\s+|\s+$/g, "");
}
```

新的JavaScript引擎已经有了trim()的原生实现。

### 14. 数组之间追加

```
var array1 = [12, 'foo', { name: 'Joe' }, -2458]
var array2 = ['Doe', 555, 100]

Array.prototype.push.apply(array1, array2)
```

### 15. 对象转换为数组

```
var argArray = Array.prototype.slice.call(arguments)
```

### 16. 验证是否为数字

```
function isNumber(n) {
  return !isNaN(parseFloat(n)) && isFinite(n)
}
```

### 17. 验证是否是数组

```
function isArray(obj) {
  return Object.prototype.toString.call(obj) === '[object Array]'
}
```
但如果toString()方法被重写过得话，就行不通了。也可以使用下面的方法：
```
Array.isArray(obj);   // its a new Array method
```

### 18. 获取数组中最大值和最小值

```
var numbers = [5, 458, 120, -215, 228, 400, 122205, -85411]
var maxInNumbers = Math.max.apply(Math, numbers)
var minInNumbers = Math.min.apply(Math, numbers)
```

### 19. 清空数组

```
var myArray = [12, 222, 1000]
myArray.length = 0;   // myArray will be equal to []
```

### 20. 不要直接从数组中 delete 或者 remove 元素

如果对数组元素直接使用delete，其实并没有删除，只是将元素置为了undefined。数组元素删除应使用splice。

```
var items = [12, 548, 'a', 2, 5478, 'foo', 8852, 'Doe', 2154, 119]
items.length;   // return 11

delete items[3];  // return true

items.length;   // return 11

/* items 结果为 [12, 548, "a", undefined × 1, 5478, "foo", 8852, undefined × 1, "Doe", 2154, 119] */
```
而应：
```
var items = [12, 548 ,'a' , 2 , 5478 , 'foo' , 8852, , 'Doe' ,2154 , 119 ];
items.length; // return 11
items.splice(3,1) ;
items.length; // return 10
/* items 结果为 [12, 548, "a", 5478, "foo", 8852, undefined × 1, "Doe", 2154, 119] */
```

### 21. 使用 length 属性截断数组

前面的例子中用 length 属性清空数组，同样可以用它截断数组

```
var myArray = [12, 222, 1000, 124, 98, 10]
myArray.length = 4; // myArray will be equal to [12 , 222 , 1000 , 124].
```

### 22. 在条件中使用逻辑和或

```
var foo = 10;
foo == 10 && doSomething()  // is the same thing as if (foo == 10) doSomething()

foo == 5 || doSomething()   // is the same thing as if(foo != 5) doSomething()
```

逻辑或还可以用来设置默认值

```
function doSomething(arg1) {
  arg1 = arg1 || 10  // arg1 will have 10 as a default value if it’s not already set
}
```

### 23. 使用 map() 函数对数据循环

```
var squares = [1, 2, 3, 4].map(function(val){
  return val * val
})

// squares will be equal to [1, 4, 9, 16]
```

### 24. 保留制定小数位数

```
var num = 2.443242
num.toFixed(4);   // num will be equal to 2.4432
```

注意: toFixed() 返回的是字符串，不是数字

### 25. 浮点计算的问题

```
0.1 + 0.2 === 0.3 // is false
9007199254740992 + 1 // is equal to 9007199254740992
9007199254740992 + 2 // is equal to 9007199254740994
```

为什么呢？因为0.1+0.2等于0.30000000000000004。JavaScript的数字都遵循IEEE 754标准构建，在内部都是64位浮点小数表示，具体可以参见JavaScript中的数字是如何编码的.

可以通过使用toFixed()和toPrecision()来解决这个问题。

### 26. 通过 for-in 循环检查对象的属性

下面这样的用法，可以防止迭代的时候进入到对象的原型属性中。

```
for( var name in object ) {
  if( object.hasOwnProperty(name) ) {
    // do something with name
  }
}
```

### 27. 逗号操作符

```
var a = 0;
var b = ( a++, 99 );
console.log(a);  // a will be equal to 1
console.log(b);  // b is equal to 99
```

### 28. 临时储存用于计算和查询的变量

在 JQuery 选择器中，可以临时储存整个 DOM 元素。

```
var navright = document.querySelector('#right');
var navleft = document.querySelector('#left');
var navup = document.querySelector('#up');
var navdown = document.querySelector('#down');
```

### 29. 提前检查传入 isFinite() 的参数

```
isFinite(0/0) ; // false
isFinite("foo"); // false
isFinite("10"); // true
isFinite(10);   // true
isFinite(undefined);  // false
isFinite();   // false
isFinite(null);  // true，这点当特别注意
```

### 30. 避免在数组中使用负数做索引

```
var numbersArray = [1, 2, 3, 4, 5];
var from = numbersArray.indexOf('foo');   // from is equal to -1
numbersArray.splice(from, 2)  ;  // will return [5]
```

注意传给splice的索引参数不要是负数，当是负数时，会从数组结尾处删除元素。
