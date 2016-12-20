---
layout:     post
title:      "【读书笔记】 策略模式"
subtitle:   "《Javascript 设计模式与开发实践》读书笔记 第二部分 --- 设计模式"
author:     "刘一凡"
header-img: "img/post-bg-2015.jpg"
tags:
    - js
    - 设计模式
---

# 策略模式

策略模式的定义是：定义一系列的算法，把它们封装起来，并且它们之间可以相互替换。

## 使用策略模式计算奖金

策略模式有着广泛的应用，这里我们用年终奖为例介绍。

例如：有的人年终奖 4 倍工资，有的人年终奖 3 倍工资，有的人两倍。假设财务部让我们用一段代码，方便他们计算员工的绩效奖。

1.最初的代码实现

我们可以编写一个名为 `calculateBonus` 的函数来计算每个人的奖金金额。这个函数需要两个参数：员工的工资数和他的绩效考核等级。

```
var calculateBonus = function (performanceLevel, salary) {
  if (performanceLevel === 'S') {
    return salary * 4
  }

  if (performanceLevel === 'A') {
    return salary * 3
  }

  if (performanceLevel === 'B') {
    return salary * 2
  }
}
```
这个代码十分简单，但是也存在显而易见的缺点。

- 包含很多逻辑分支(if-else)
- 缺乏弹性，如果想加一种新的绩效考核，或者需要把S改成5，就必须深入函数内部
- 算法的复用性差，如果其他地方需要使用这些计算奖金的算法，我们只能复制和粘贴

2.使用组合函数重构代码

将各种算法封装到一些小函数中，这些小函数有着良好的命名，可以一目了然地知道它对应着哪种算法，也可以被复用到其他地方。

```
var performanceS = function (salary) {
  return salary *4
}

var performanceA = function (salary) {
  return salary *3
}

var performanceB = function (salary) {
  return salary *2
}

var calculateBonus = function (performanceLevel, salary) {
  if (performanceLevel === 'S') {
    return performanceS(salary)
  }

  if (performanceLevel === 'A') {
    return performanceS(salary)
  }

  if (performanceLevel === 'B') {
    return performanceS(salary)
  }
}

calculateBonus('A', 10000)  // 输出 30000
```

目前我们的程序得到了改善，但这种改善非常有限，我们依然没有解决最重要的问题，
calculateBonus 函数有可能越来越大，并且在系统变化的时候缺乏弹性。

3.使用策略模式重构代码

策略模式指的是定义一系列的算法，把他们一个个封装，将不变的部分和变化的部分隔开是每个设计模式的主题，策略模式的目的就是将算法的使用和算法的实现分离开。

这个例子中，算法的使用方式是不变的，都是根据某个算法取得计算后的奖金数额。
而算法的实现是各异和变化的，每种绩效对应着不同的计算规则。

一个基于策略模式的程序至少由两部分组成，第一部分是一组策略类，封装具体的算法，并负责具体的计算过程。第二部分是环境类 Context，Context 接受客户的请求，随后把请求委托给某个策略类，要做到这点，说明 Context 中要维持对某个策略对象的引用。

现在用策略模式重构上面的代码，第一版本是模仿传统语言的实现。先把每种绩效的计算规则封装到对应的策略类里面：

```
var performanceS = function () {}

performanceS.prototype.calculate = function (salary) {
  return salary * 4
}

var performanceA = function () {}

performanceA.prototype.calculate = function (salary) {
  return salary * 3
}

var performanceB = function () {}

performanceB.prototype.calculate = function (salary) {
  return salary * 2
}
```
定义奖金类 Bonus:

```
var Bonus = function () {
  this.salary = null    // 原始工资
  this.strategy = null  // 绩效等级对应的策略对象
}

Bonus.prototype.setSalary = function (salary) {
  this.salary = salary  // 设置员工的原始工资
}

Bonus.prototype.setStrategy = function (strategy) {
  this.strategy = strategy    // 设置员工绩效等级对应的策略对象
}

Bonus.prototype.getBonus = function () {    // 取得奖金数额
  return this.strategy.calculate(this.salary)   // 把计算奖金的操作委托给对应的策略对象
}
```

在完成代码前，在回顾下策略模式的思想：

定义一系列的算法，把他们一个个封装起来，并且使它们可以相互替换。

详细一点就是：定义一系列的算法，把他们封装成策略类，算法被封装在策略类内部的方法里。在客户对 Context 发起请求时，Context 总是把请求委托给这些策略对象中间的某一个计算。

下面完成这个例子剩下的代码，

```
var bonus = new Bonus()

bonus.setSalary(10000)
bonus.setStrategy(new performanceS())   // 设置策略对象

console.log(bonus.getBonus())     // 输出 40000

bonus.setStrategy(new performanceA())   // 设置策略对象
console.log(bonus.getBonus())     // 输出 30000        
```

这些代码是基于传统面向对象语言的模仿，下面将介绍用 Javascript 实现的策略模式。

## JavaScript 版本的策略模式

上面我们使用 strategy 对象从各个策略类中创建而来，这是模拟一些传统面向对象语言的实现。实际上在 Javascript 语言中，函数也是对象，所以更简单和直接的做法是把 strategy 直接定义为函数：

```
var strategys = {
  'S': function (salary) {
    return salary * 4
  },
  'A': function (salary) {
    return salary * 3
  },
  'B': function (salary) {
    return salary * 2
  }
}
```
Context 也没必要用 Bonus 类表示，可以使用 calculateBonus 函数充当 Context 来接受用户的请求。

```
var strategys = {
  'S': function (salary) {
    return salary * 4
  },
  'A': function (salary) {
    return salary * 3
  },
  'B': function (salary) {
    return salary * 2
  }
}

var calculateBonus = function (level, salary) {
  return strategys[level](salary)
}

console.log(calculateBonus('S', 20000))   // 输出 80000
console.log(calculateBonus('A', 10000))   // 输出 30000
```
