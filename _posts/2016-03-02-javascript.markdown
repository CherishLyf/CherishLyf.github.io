---
layout:     post
title:      "【转】深入理解 JavaScript 原型和闭包"
subtitle:   " \"原文写得非常好, 通俗易懂\""
author:     "刘一凡"
header-img: "img/post-bg-2015.jpg"
tags:
    - 前端开发
---

[原文地址：王福朋的博客](http://www.cnblogs.com/wangfupeng1988/p/3977924.html)

## 理解对象

在理解对象之前，先看下都有哪几种数据类型，这里我们要用到一个 JavaScript 中的一个常用函数 --- `typeof()`

    function show(x){
        console.log(typeof(x))          // undefined
        console.log(typeof(10))         // number
        console.log(typeof("abc"))      // string
        console.log(typeof(true))       // boolean

        console.log(typeof(function(){   }))        // function

        console.log(typeof([1, "a", true]))                  // object
        console.log(typeof({ name : "Lyf", age : "20" }))    // object
        console.log(typeof(null))                            // object
        console.log(typeof(new Number(10)))                  // object
    } 

以上 typeof 输出的类型中，前四种 `undefined`, `number`, `string`, `boolean` 属于简单的**值类型**，剩下的几种 `函数`, `对象`, `数组`, `null`, `new Number(10)`, 都是对象，他们都是**引用类型** <br />
判断一个变量是不是对象非常简单。值类型的类型判断用 `typeof`, 引用类型判断用 `instanceof`

    var fn = function(){};
    console.log(fn instanceof Object);          // true

那么问题来了，JavaScript 中对象到底如何定义?<br />
**对象 --- 若干属性的集合**

下面举一个例子
    
    var person = {
        name : "Lyf", 
        age : "20"
    }

以上代码中，`person` 就是一个对象， name 和 age 就是他的属性<br/>

**一切（引用类型）都是对象，对象是属性的集合**<br />
那么问题来了, 在 typeof 的输出类型中，functin 和 object 都是对象，为什么都要输出两种答案那，都叫 object 不行吗？

## 函数和对象的关系

函数是对象的一种，可以通过 instanceof 函数判断
    
    var fn = function(){  };
    console.log(fn instanceof Object);          // true

先看一个小例子

    function Fn(){
        this.name = "Lyf"
        this.age = "20"
    }
    var fn = new Fn()

上面的例子很简单，他说明：**对象是通过函数创建的**。对，只能说明这一点。<br />
但是我要说 --- **对象都是通过函数创建的**，但是有人会反驳，因为:
    
    var obj = { a : 10, b : 20 }
    var man = [ 1, "x", true ]

但是，这个是一种**快捷方式**, 在编程语言中，这一般叫**"语法糖"**<br />
以上代码的本质是:
    
    // var obj = { a : 10, b : 20 }
    var obj = new Object()
    obj.a = 10
    obj.b = 20
    
    // var man = [ 1, "x", true ]
    var man = new Array()
    man[0] = 1
    man[1] = "x"
    man[3] = true

其中 `Object`  和 `Array` 都是函数

    console.log(typeof(Object));    // function
    console.log(typeof(Array));     // function

所以说，**对象都是通过函数创建的**

## prototype 原型
    
之前说过，函数也是一种对象，他是属性的集合，所以可以对函数进行自定义属性。<br />
函数在创建的时候，JavaScript 就给函数创建了一个属性 --- `prototype`，每个函数都有一个属性叫 `prototype` <br />
这个`prototype` 的属性值是一个对象(属性的集合)，默认只有一个`constructor`的属性，指向函数本身
![constructor](/img/in-post/post-js-prototype/constructor.png)

如上图，SuperType 是一个函数，右侧方框是他的原型。<br />
原型既然作为对象，属性的集合，不可能只有一个 `constructor` 属性，肯定可以添加许多属性，例如 `Object` 里面，又有好多属性
![ObjectConstructor](/img/in-post/post-js-prototype/ObjectConstructor.png)

还可以在自己自定义方法的 `prototype` 中新增自己的属性

    function Fn(){  }
    Fn.prototype.name = "Lyf"
    Fn.prototype.getYear = function(){
        return 1995
    }

这样就变成
![fnConstructor](/img/in-post/post-js-prototype/fnConstructor.png)

但是，这样有什么？我们看下 jQuery 的源码：

    var $div = $("div")
    $div.attr("myName", "Lyf")

以上代码中，$("div") 返回的是一个对象，对象一一被函数创建。假设创建这一对象是 myjQuery，其实他是这样实现的：

    myjQuery.prototype.attr = function(){
        // ...
    }
    $("div") = new myjQuery()

用自己的代码演示，就是这样

    function Fn(){
        // ...
    }
    Fn.prototype.name = "Lyf"
    Fn.prototype.getYear = function(){
        return 1995
    }

    var fn = new Fn()
    console.log(fn.name)
    console.log(fn.getYear)

即，Fn 是一个函数，fn 对象是从 Fn 函数 `new` 出来，这样 fn 对象就可以调用 Fn.prototype 中的属性<br />
因为每个对象都有一个隐藏属性 --- `__proto__`，这个属性引用了这个对象的 prototype。即 **fn.__proto__ === Fn.prototype** , 这里的 `__proto__` 成为 **隐式原型**

## 隐式原型

**每个函数 function 都有一个 prototype**，即原型。每个对象都有一个 `__proto__`，可成为隐式原型。

    var obj = { }
    console.log("obj.__proto__ =")
    console.log(obj.__proto__)
    console.log("Object.prototype =")
    console.log(Object.prototype)

![proto](/img/in-post/post-js-prototype/proto.png)

从截图可看出，`obj.__proto__` 和 `Object.prototype` 的属性一样 ! <br />
本质上说 obj 这个对象是被 Object 函数创建的，因此 `obj.__proto__ === Object.prototype`
![Object.prototype](/img/in-post/post-js-prototype/Object.prototype.png)

即，每个对象都有一个 `__proto__` 属性，指向创建该对象的 `prototype`<br />
自定义函数的 `prototype` 本质上就是和 var obj = {} 是一样的，都是被 `Object` 创建，所以它的`__proto__`指向的就是`Object.prototype`。<br />
但是`Object.prototype`确实一个特例 --- 它的 `__proto__ `指向的是 `null`，切记切记！
![Object.prototype1](/img/in-post/post-js-prototype/Object.prototype1.png)

函数也是一种对象，也是被创建出来的，谁创建了函数呢？ --- `Function`<br />

    function fn(x, y){
        return x + y
    }
    console.log(fn(10, 20))

    var fn1 = new Function("x", "y", "return x + y")
    console.log(fn1(5, 6))

以上代码中，第一种方式是比较传统的函数创建方式，第二种是用new Functoin创建。<br />
这里只是向大家演示，函数是被Function创建的。

根据上面说的一句话——对象的`__proto__`指向的是创建它的函数的`prototype`，就会出现：`Object.__proto__ === Function.prototype`。用一个图来表示：
![Function](/img/in-post/post-js-prototype/Function.png)

上图中，很明显的标出： `Foo.__proto__`指向`Function.prototype`, `Object.__proto__` 指向 `Function.prototype`<br />
其实`Function`也是一个函数，函数是一种对象，也有 `__proto__`属性。<br />
既然是函数，那么它一定是被`Function`创建。所以`Function是`被自身创建的。所以它的`__proto__`指向了自身的`prototype`。

最后一个问题：`Function.prototype`指向的对象，它的`__proto__`是不是也指向`Object.prototype`？<br />
答案是肯定的。因为`Function.prototype`指向的对象也是一个普通的被Object创建的对象，所以也遵循基本的规则。
![Function.prototype](/img/in-post/post-js-prototype/Function.prototype.png)

## 继承

JavaScript 中的继承是通过**原型链**来体现的，先看几个代码

    function Foo(){
        var f1 = new Foo()
    }

    f1.a = 10

    Foo.prototype.a = 100
    Foo.prototype.b = 200

    console.log(f1.a)       // 10
    console.log(f1.b)       // 200

以上代码中，`f1` 是 `Foo` 函数 `new` 出来的，`f1.a` 是 `f1` 对象的基本属性，`f1.b` 是怎么来的？ --- 从 `Foo.prototype` 得来，因为 `f1.__proto__` 指向的是 `Foo.prototype`

`访问一个对象的属性时，先在基本属性中查找，如果没有，再沿着 __proto__ 这条链向上找，这就是原型链`
![f1](/img/in-post/post-js-prototype/f1.png)

如图所示，访问 f1.b 时，f1 的基本属性没有 b ，于是沿着 `__proto__` 找到了 `Foo.prototype.b`

那么我们在实际应用中如何区分一个属性到底是基本的还是从原型中找到的呢？大家可能都知道答案 --- `hasOwnProperty`，特别是在for…in…循环中，一定要注意。
![hasOwnProperty](/img/in-post/post-js-prototype/hasOwnProperty.png)

等等，不对！ f1的这个`hasOwnProperty`方法是从哪里来的？ `f1`本身没有，`Foo.prototype`中也没有，哪儿来的？<br />
它是从`Object.prototype`中来的，请看图：
![hasOwnProperty1](/img/in-post/post-js-prototype/hasOwnProperty1.png)

对象的原型链是沿着 `__proto__` 这条线走的，因此在查找 `f1.hasOwnProperty` 属性时，会沿着原型链一直找到 `Object.prototype`

由于所有的对象的原型链都会找到 `Object.prototye`，因此所有的对象会有 `Object.prototype` 方法。这就是所谓的 **继承**



## 执行上下文环境

什么是“执行上下文”（也叫做“执行上下文环境”）？暂且不下定义，先看一段代码：
![a](/img/in-post/post-js-prototype/a.png)

第一句报错，`a`未定义，很正常。第二句、第三句输出都是`undefined`，说明浏览器在执行`console.log(a)`时，已经知道了a是`undefined`，但却不知道a是10（第三句中）。

在一段js代码拿过来真正一句一句运行之前，浏览器已经做了一些**“准备工作”**，其中就包括对变量的声明，而不是赋值。变量赋值是在赋值语句执行的时候进行的

我们总结一下，在“准备工作”中完成了哪些工作：

变量、函数表达式 --- 变量声明，默认赋值为 `undefined`；<br />
`this` --- 赋值；<br />
函数声明 --- 赋值；<br />
这三种数据的准备情况我们称之为“执行上下文”或者“执行上下文环境”。

如果在函数中，除了以上数据，还会有其他数据。先看下面代码：

    function fn(x){
        console.log(arguments);     [10]
        console.log(x);     10
    }
    fn(10)

以上代码展示了在函数体的语句执行之前，`arguments`变量和函数参数都已经赋值，从这里看出，`函数在每被调用一次，都会产生一个新的上下文环境`，因为不同的调用会有不同的参数。

另外，`函数在定义的时候(不是调用的时候)，就已经确定了函数体内部自由变量的作用域`
    
    var a = 10;
    function fn(){
        console.log(a)      // a是自由变量
                            // 函数创建的时候，就确定了 a 要取值的作用域
    }

    function bar(f){
        var a = 20
        f()         // 打印 "10", 而不是 "20"
    }
    bar(fn)

## this

this的取值，分四种情况。我们来挨个看一下。

在此再强调一遍一个非常重要的知识点：`在函数中this到底取何值，是在函数真正被调用执行的时候确定的，函数定义的时候确定不了`。因为this的取值是执行上下文环境的一部分，每次调用函数，都会产生一个新的执行上下文环境。

**情况1 : 构造函数**<br />
所谓的构造函数，就是用来 new 对象的函数。严格来说，所有的函数都可以 new 一个对象，但有些函数的定义是为了 new 一个对象，有些函数不是。

    function Foo(){
        this.name = "Lyf"
        this.age = 20

        console.log(this)       // Foo
    }

    var f1 = new Foo()
    console.log(f1.name)        // Lyf
    console.log(f1.age)         // 20

以上代码中，如果函数作为构造函数用，那么其中的`this`就代表它即将`new`出来的对象

注意，以上仅限`new Foo()`的情况，即`Foo`函数作为构造函数的情况。如果直接调用`Foo`函数，而不是`new Foo()`，情况就大不一样了。

    function Foo(){
        this.name = "Lyf"
        this.age = 20

        console.log(this)       // Window
    }

    Foo()

这种情况下this是window，我们后文中会说到。

**情况2 : 函数作为对象的一个属性**<br />
如果函数作为`对象的一个属性`时，并且作为`对象的一个属性`被调用时，函数中的`this`指向该对象。

    var obj = {
        x : 10,
        fn : function(){
            console.log(this)       // obj
            console.log(this.x)         // 10
        }
    }

    obj.fn()

以上代码中，`fn`不仅作为一个对象的一个属性，而且的确是作为对象的一个属性被调用。结果`this`就是`obj`对象


注意，如果fn函数不作为obj的一个属性被调用，会是什么结果呢？

    var obj = {
        x : 10,
        fn : function(){
            console.log(this)       // Window
            console.log(this.x)         // undefined
        }
    }

    var fn1 = obj.fn
    fn1()

如上代码，如果`fn`函数被赋值到了另一个变量中，并没有作为`obj`的一个属性被调用，那么`this`的值就是`window`，`this.x`为`undefined`

**情况3：函数用call或者apply调用**<br />
当一个函数被call和apply调用时，this的值就取传入的对象的值。

    var obj = {
        x : 10
    }

    var fn = function(){
        console.log(this)       // obj
        console.log(this.x)         // 10
    }

    fn.call(obj)

**情况4：全局 & 调用普通函数**<br />
在全局环境下，this永远是window，这个应该没有非议。

    console.log(this === window);       // true

普通函数在调用时，其中的this也都是window。

    var x = 10
    var fn = function(){
        console.log(this)       // Window
        console.log(this.x)         // 10
    }

    fn()

下面这种情况也得注意

    var obj = {
        x : 10,
        fn : function(){
            function f(){
               console.log(this)       // Window
                console.log(this.x)         // undefined 
            }
            f()
        }
    }
    obj.fn()

函数`f`虽然是在`obj.fn`内部定义的，但是它仍然是一个普通的函数，`this`仍然指向`window`。