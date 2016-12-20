---
layout:     post
title:      "JS 数组拷贝"
subtitle:   "JS 数组拷贝"
author:     "刘一凡"
header-img: "img/post-bg-2015.jpg"
tags:
    - js
---

# JS 数组拷贝

我们在使用 JavaScript 对数组进行操作，经常需要把数组进行备份，事实证明如果只是简单的将它赋予其他的变量，只要改变其中的任何一个，然后其他的也会跟着改变：

```
var data = [1, 2, 3];

var newData = data
newData[3] = 4
console.log('数据的原始值为: ' + data + ', ' + '数组的当前值为: ' + newData + '.')
// data: [1, 2, 3, 4]
// newData: [1, 2, 3, 4]
```

这种直接赋值的方式就是浅拷贝，这样并不是我们想要的结果，其实我们想要的 data 值不变。

方法一： `slice` 方法

[slice() | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)

`slice()` 方法会浅复制（shallow copy）数组的一部分到一个新的数组，并返回这个新数组。

```
var data = [1, 2, 3];

var newData = data.slice(0)
newData[3] = 4
console.log('数据的原始值为: ' + data + ', ' + '数组的当前值为: ' + newData + '.')
// data: [1, 2, 3]
// newData: [1, 2, 3, 4]
```

方法二： `concat` 方法

[concat() | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/concat)

`concat()` 方法将传入的数组或非数组值与原数组合并，组成一个新的数组并返回。

```
var data = [1, 2, 3];

var newData = data.concat
newData[3] = 4
console.log('数据的原始值为: ' + data + ', ' + '数组的当前值为: ' + newData + '.')
// data: [1, 2, 3]
// newData: [1, 2, 3, 4]
```
