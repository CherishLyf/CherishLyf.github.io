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

## 表单验证

在一个 Web 项目中，注册、登陆、修改用户信息等功能的实现的都离不开提交表单。

假设我们正在编写一个注册的页面，在点击注册之前，有几条校验逻辑。

- 用户名不能为空
- 密码长度不能少于 6 位
- 手机号码必须符号格式

### 表单验证的第一个版本

第一个版本没有引入策略模式，代码如下：

```
<html>
  <body>
    <form action="#" id="registerForm" method="post">
      请输入用户名: <input type="text" name="userName" />
      请输入密码: <input type="text" name="password" />
      请输入手机号码: <input type="text" name="phoneNumber">
      <button>提交</button>
    </form>
    <script>
      var registerForm = document.getElementById('registerForm')

      registerForm.onsubmit = function () {
        if (registerForm.userName.value === '') {
          alert('用户名不能为空')
          return false
        }

        if (registerForm.password.value.length < 6) {
          alert('密码长度不能少于6位')
          return false
        }

        if ( !/(^1[3|5|8][0-9]{9}$)/.test(registerForm.phoneNumber.value)){
          alert('手机号码格式不正确')
          return false
        }
      }
    </script>
  </body>
</html>
```

这是一种常见的编码方式，缺点跟计算奖金的最初版本一样。

- onsubmit 函数庞大，包含很多 if-else 分支
- 缺乏弹性，如果增加一种新的校验规则，必须深入 onsubmit 的内部实现，这是违反开放-封闭原则的
- 算法复用性差，如果在程序中另外添加一个表单，那么我们很有可能将这些校验逻辑复制的满天飞

### 用策略模式重构表单校验

下面我们使用策略模式重构表单验证代码，很显然我们要把这些检验逻辑都封装成策略对象：

```
var strategys = {
  isNonEmpty: function (value, errorMsg) {  // 不能为空
    if ( value === '' ) {
      return errorMsg
    }
  },
  minLength: function (value, length, errorMsg) {   // 限制最小长度
    if ( value.length < length ) {
      return errorMsg
    }
  },
  isMobile: function (value, errorMsg) {    // 手机号码格式
    if ( !/(^1[3|5|8][0-9]{9}$)/.test(value) ){
      return errorMsg
    }
  }
}
```
接下来准备实现 Validator 类，Validator 类在这里作为 Context，负责接收用户的请求并委托给 strategy 对象。在给出 Validator 类的代码之前，有必要提前了解用户是如何向 Validator 类发送请求的，这有助于我们如何去编写 Validator 类代码：

```
var validataFunc = function () {
  var validator = new Validator()     // 创建一个 validator 对象

  /***************添加一些校验规则****************/
  validator.add(registerForm.userName, 'isNonEmpty', '用户名不能为空')
  validator.add(registerForm.password, 'minLength:6', '密码长度不能少于 6 位')
  validator.add(registerForm.phoneNumber, 'isMobile', '手机号码格式不正确')

  var errorMsg = validator.start()    // 获得校验结果
  return errorMsg;    // 返回校验结果
}

var registerForm = document.getElementById('registerForm')
registerForm.onsubmit = function () {
  var errorMsg = validataFunc()   // 如果 errorMsg 有确切的返回值，说明未通过校验
  if (errorMsg) {
    alert(errorMsg)
    return false    // 组织表单提交
  }
}
```
最后是 Validator 类的实现：

```
var Validator = function () {
  this.cache = []     // 保存校验规则
}

Validator.prototype.add = function (dom, rule, errorMsg) {
  var ary = rule.split(':')   // 把 strategy 和 参数分开
  this.cache.push(function(){   // 把校验的步骤用空函数包装起来，并且放入 cache
    var strategy = ary.shift()    // 用户挑选的 strategy
    ary.unshift(dom.value)      // 把 input 的 value 添加进参数列表
    ary.push(errorMsg)          // 把 errorMsg 添加进参数列表
    return strategies[strategy].apply(dom, ary)
  })
}

Validator.prototype.start = function () {
  for (var i = 0, validatorFunc; validatorFunc = this.cache[i++];) {
    var msg = validatorFunc()     // 开始效验，并取得校验后的返回信息
    if (msg) {    // 如果有确切的返回值，说明校验没有通过
      return msg
    }
  }
}
```

在修改某个校验规则的时候，只需要编写少量代码，比如我们想要将用户名输入框的校验规则改成用户名不能少于 4 个字符，这时候修改是很简单的：

```
validator.add(registerForm.userName, 'isNonEmpty', '用户名不能为空')

//　改成：
validator.add(registerForm.userName, 'minLength:10', '用户名不能少于10位')
```

### 给某个文本输入框添加多种校验规则

目前的表单只能对应一种校验规则，比如，只能校验输入是否为空，如果我们想要即校验它是否为空，又想校验它输入的文本不小于 10？我们期望以这样的形式进行校验：

```
validator.add(registerForm.userName, [{
  strategy: 'isNonEmpty',
  errorMsg: '用户名不能为空'
}, {
  strategy: 'minLength:6',
  errorMsg: '用户名长度不能小于 10 位'
}])
```
下面提供的代码可用于一个文本框输入框对应多种校验规则：

```
<html>

<body>
    <form action="http:// xxx.com/register" id="registerForm" method="post">
        请输入用户名：<input type="text" name="userName" /> 请输入密码：
        <input type="text" name="password" /> 请输入手机号码：
        <input type="text" name="phoneNumber" />
        <button>提交</button>
    </form>
    <script>
        /***********************策略对象**************************/
        var strategies = {
            isNonEmpty: function(value, errorMsg) {
                if (value === '') {
                    return errorMsg;
                }
            },
            minLength: function(value, length, errorMsg) {
                if (value.length < length) {
                    return errorMsg;
                }
            },
            isMobile: function(value, errorMsg) {
                if (!/(^1[3|5|8][0-9]{9}$)/.test(value)) {
                    return errorMsg;
                }
            }
        };
        /***********************Validator 类**************************/
        var Validator = function() {
            this.cache = [];
        };
        Validator.prototype.add = function(dom, rules) {
            var self = this;
            for (var i = 0, rule; rule = rules[i++];) {
                (function(rule) {
                    var strategyAry = rule.strategy.split(':');
                    var errorMsg = rule.errorMsg;
                    self.cache.push(function() {
                        var strategy = strategyAry.shift();
                        strategyAry.unshift(dom.value);
                        strategyAry.push(errorMsg);
                        return strategies[strategy].apply(dom, strategyAry);
                    });
                })(rule)
            }
        };
        Validator.prototype.start = function() {
            for (var i = 0, validatorFunc; validatorFunc = this.cache[i++];) {
                var errorMsg = validatorFunc();
                if (errorMsg) {
                    return errorMsg;
                }
            }
        };
        /***********************客户调用代码**************************/
        var registerForm = document.getElementById('registerForm');
        var validataFunc = function() {
            var validator = new Validator();
            validator.add(registerForm.userName, [{
                strategy: 'isNonEmpty',
                errorMsg: '用户名不能为空'
            }, {
                strategy: 'minLength:6',
                errorMsg: '用户名长度不能小于10 位'
            }]);
            validator.add(registerForm.password, [{
                strategy: 'minLength:6',
                errorMsg: '密码长度不能小于6 位'
            }]);
            validator.add(registerForm.phoneNumber, [{
                strategy: 'isMobile',
                errorMsg: '手机号码格式不正确'
            }]);
            var errorMsg = validator.start();
            return errorMsg;
        }
        registerForm.onsubmit = function() {
            var errorMsg = validataFunc();
            if (errorMsg) {
                alert(errorMsg);
                return false;
            }

        };
    </script>
</body>

</html>

```
