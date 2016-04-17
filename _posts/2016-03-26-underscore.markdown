---
layout:     post
title:      "Underscore 源码阅读"
subtitle:   " \"阅读 underscore源码, 提高 JS 水平\""
author:     "刘一凡"
header-img: "img/post-bg-2015.jpg"
tags:
    - underscore
---

经常听别人说 `underscore` 代码写得优雅, 适合阅读, 近几天抽时结合别人的分析, 自己也看一下.
我看的版本是 `1.8.2` [underscore 带注释源码](http://www.css88.com/doc/underscore/docs/underscore.html)

这是我看的比较好的一篇分析的博文<br>
[underscore源码经典（一）](http://yalishizhude.github.io/2015/09/22/underscore-source/) <br>
[underscore源码经典（二）](http://yalishizhude.github.io/2015/10/12/underscore-source-2/) <br>
[underscore源码经典（三）](http://yalishizhude.github.io/2015/10/17/underscore-source-3/) <br>
[underscore源码经典（完）](http://yalishizhude.github.io/2015/10/21/underscore-source-4/)

## 闭包

---

`underscore` 代码包在一个自执行的匿名函数中

    (function(){
        
        // ...

    }.call(this))


## 基本设置

---

### 对象原型
    
保存对象原型, 以便下面的调用

    // 创建一个全局对象, 在浏览器中表示 window, 在服务器中表示 export
    var root = this;

    // 保存"_"(下划线变量)被覆盖之前的值
    // 如果出现命名冲突或考虑到规范, 可通过_.noConflict()方法恢复"_"被Underscore占用之前的值, 并返回Underscore对象以便重新命名
    var previousUnderscore = root._;

    // 保存对象原型
    var ArrayProto = Array.prototype,
        ObjProto   = Object.prototype,
        FuncProto  = Object.prototype;


### 常用方法

保存常用方法, 以便快速引用

    // 保存常用方法, 以便快速引用
    var push            = ArrayProto.push,
        slice           = ArrayProto.slice,
        toString        = ObjProto.toString,
        hasOwnPrototype = ObjProto.hasOwnPrototype;

### 保存 ES5 原生方法

将 `ES5` 实现的原生方法保存, 以便浏览器支持原生方法时, 使用原生方法
    
    // 保存 ES5 实现的原生方法
    var nativeIsArray = Array.isArray,
        nativeKeys    = Object.keys,
        nativeBind    = FuncProto.bind,
        nativeCreate  = Object.create;

### 创建一个 Underscore 对象

`Underscore` 对象, 用 `_` 表示

    // 一个全局引用函数
    var Ctor = function(){};
    
    // Underscore 对象以便下面调用

    var _ = function(obj){
      if(obj instanceof _) return obj
      if(!(this instanceof _)) return new _(obj)
      this._wraped = obj;
    }

### 命名空间
    
针对不同的浏览器环境, 不同的命名空间

    // 针对不同的环境, 将 underscore 的命名空间存放到不同的对象
    if(typeof exports !== "undefined"){   // node.js 环境
      if(typeof module !== "undefined" && module.exports){
        exports = module.exports = _;
      }
      exports._ = _;
    } else { // 浏览器环境
      root._ = _;
    }

### 版本号
    
记录版本号

    _.VERSION = "1.8.2"

<div id="optimizeCb"></div>

### 内部函数, 改变函数作用域

这是一个比较重要的内部函数, 主要来**执行函数**并**改变所执行函数的作用域**<br>
`argCount` 参数来指定参数的个数，并对参数个数小于等于 `4` 的情况做了处理

    var optimizeCb = function(func, context, argCount){
      if(context === void 0) return func;
      switch(argCount == null ? 3 : argCount){
        case 1 : return function(value){
          return func.call(context, value)
        };
        case 2 : return function(value, other){
          return func.call(context, value, other)
        };
        case 3 : return function(value, index, collection){
          return func.call(context, value, index, collection)
        };
        case 4 : return function(accumulator, value, index, collection){
          return func.call(context, accumulator, value, index, collection)
        };
      }

      return function(){
        return func.apply(context, arguments)
      }

`switch` 这里不太理解, 看了别人的分析, 这段 `switch` 完全是多余的, 可以用下面这段代替 <br>
`switch` 那段的唯一用处, 就是可以向你展示一个规范, 告诉你在传递几个参数的时候, 相应的参数什么是什么东东而已, 而这种东东, 完全可以写在注释里.
    
    var optimizeCb = function(func, context) {
        if (context === void 0) return func;
        return function() {
          return func.apply(context, arguments);
        };
    };

这也是一个比较常用的内部函数，只是对参数进行了判断：<br>
如果是函数则返回上面说到的回调函数；<br>
如果是对象则返回一个能判断对象是否相等的函数；<br>
默认返回一个获取对象属性的函数。

      var cb = function(value, context, argCount){
        // value 是否为空, 如果是空, 返回本身
        if(value == null) return _.identity;
        // value 为函数时, 改变作用域
        if(_.isFunction(value)) return optimizeCb(value, context, argCount)
        // value 为对象时, 判断是否匹配
        if(_.isObject(value)) return _.matcher(value)
        return _.property(value)
      }
      _.iteratee = function(value, context){
        return cb(value, context, Infinity)
      }
    
    // 这里还没搞懂
    var createAssigner = function(keysFunc, undefinedOnly){
      return function(obj){
        // 参数的数量
        var length = arguments.length;
        console.log(length)
        // 如果对象为 null, 或者只有一个参数, 返回这个对象
        if(length < 2 || obj == null) return obj;
        // 从第二个对象开始循环
        for(var index = 1; index < length; index ++){
          var source = arguments[index],
              keys   = keysFunc(source),
              l      = keys.length;
          for(var i = 0; i < l; i++){
            var key = keys[i];
            if(!undefinedOnly || obj[key] === void 0) obj[key] = source[key];
          }
        }
      }

      return obj
    }

### 创建一个继承自其他对象的对象
    
创建一个继承自其他对象的对象

    var baseCreate = function(prototype){
      if(!_isObject(prototype)) return {}
      if(nativeCreate) return nativeCreate(prototype)
      Ctor.prototype = prototype
      var result = new Ctor
      Ctor.prototype = null
      return result
    }

### JS 最大精确整数

JS能够处理的最大精确整数

    var MAX_ARRAR_INDEX = Math.pow(2, 53) - 1;
    var isArrayLike = function(collection){
      var length = collection && collecion.length;
      // 如果集合有 length 属性且为数字并且大于0小于最大精确整数, 则判断为类数组
      return typeof length == "number" && length >= 0 && length <= MAX_ARRAR_INDEX;
    }  

### 数据判断

判断是否为对象

    _.isObject = function(obj){
      var type = typeof obj;
      return type === "function" || type === "object" && !!obj
    }

判断是否为函数

    _.isFunction = function(obj) {
          return typeof obj == 'function' || false;
    };

## 集合方法

---

### each(forEach)

`_.each(list, literator, [context])` 别名 `forEach` <br>
对一个 `list` 的所有元素进行迭代，对每一个元素执行 `iterator` 函数。`iterator` 和 `context` 对象绑定，如果传了这个参数。<br>
每次 `iterator` 的调用将会带有三个参数：`(element, index, list)`。<br>
如果 `list` 是一个 `JavaScript` 对象，`iterato`r 的参数将会是 `(value, key, list)`。如果有原生的 `forEach` 函数就会用原生的代替。

这里用到了几个方法    <br>
[optimizeCb()](#optimizeCb) <br>
[iteratee()](#optimizeCb)  <br>
[枚举 _.(keys) ](#keys)

    // 迭代 each
    _.each = _.forEach = function(obj, iteratee, context){
      iteratee = optimizeCb(iteratee, context)
      var i, length;
      if(isArrayLike(obj)){      // 当传入的 obj 是 数组
        for(i = 0, length = obj.length; i < length; i++){
          iteratee(obj[i], i, obj)
        }
      } else {          // 当传入的 obj 是 对象
        var keys = _.keys(obj);
        for(i = 0, length = keys.length; i < length; i++){
          iteratee(obj[keys[i]], keys[i], obj)
        }
      }

      return obj;
    }

**示例 1 : 数组**

    var a = [1, 2, 3];
    _.each(a, function(value, index, list){
      console.log("value:" + value + ", " + "index:" + index + ", " + "list:" + list)
    })

    => 输出 value:1, index:0, list:1,2,3
    => 输出 value:2, index:1, list:1,2,3
    => 输出 value:3, index:2, list:1,2,3

 **示例 2 : 对象**

    var b = {"前端" : "张三", "后端" : "李四"}
    _.each(b, function(value, index, list){
      console.log("value:" + value + ", " + "index:" + index + ", " + "list:" + list)
    })

    => value:张三, index:前端, list:[object Object]
    => value:李四, index:后端, list:[object Object]

<div id="map"></div>

### map(collect)

`_.map(list, literator, [context])` 别名 `collect` <br>
对 `list` 中每个元素运行 `literator` 函数, 并返回执行过后的新生成的数组 <br>
与 `each` 方法一样, 每次 `iterator` 的调用将会带有三个参数：`(element, index, list)`

    _.map = _.collect = function(obj, iteratee, context){
      iteratee    = cb(iteratee, context)
      var keys    = !isArrayLike(obj) && _.keys(obj),
          length  = (keys || obj).length,
          results = Array(length);
      for(var index = 0; index < length; index++){
        var currentKey = keys ? keys[index] : index;
        results[index] = iteratee(obj[currentKey], currentKey, obj)
      }

      return results
    }

**示例 1 : 数组**

    var a = [1, 2];
    var b = _.map(a, function(value, key, list){
      return value + 1
    })
    console.log(b)

    => 输出 [2, 3]

**示例 2 : 对象**

    var a = {"月薪" : 1000};
    var b = _.map(a, function(value, key, list){
      return value + 200;
    })
    // 实际 b 返回的是数组 [1200]
    console.log("当前月薪为 " + b)

    => 当前月薪为 1200

### reduce, reduceRight

`_.reduce(list, iteratee, [memo], [context])` 别名 `inject, foldl` <br>
`reduce` 方法把 `list` 中元素归结为一个单独的数值 <br>
`memo` 是 `reduce` 函数的初始值, `reduce` 的每一步都需要由 `iteratee` 返回 <br>
`iteratee` 迭代传递 `4` 个参数, `memo`, `value`, `index(key)`, `list`

    // 创建一个向左, 向右的 reduce 函数
        function createReduce(dir){
          function iterator(obj, iteratee, memo, keys, index, length){
            for(; index >=0 && index < length; index += dir){
              var currentKey = keys ? keys[index] : index;
              memo = iteratee(memo, obj[currentKey], currentKey, obj);
            }

            return memo;
          }

          return function(obj, iteratee, memo, context){
            iteratee   = optimizeCb(iteratee, context, 4);
            var keys   = !isArrayLike(obj) && _.keys(),
                length = (keys || obj).length,
                index  = dir > 0 ? 0 : length - 1;

            // 没有传递初始值, 将第一个元素将取代传递给列表中下一个元素调用 iteratee的 memo 参数
            if(arguments.length < 3){
              memo = obj[keys ? keys[index] : index];
              index += dir;
            }

            return iterator(obj, iteratee, memo, keys, index, length)
          }
        }

        // 从开始左侧迭代 reduce
        _.reduce = _.foldl = _.inject = createReduce(1);

        // 从右侧右侧迭代 reduce
        _.reduceRight = _.foldr = createReduce(-1);

**示例 1 : reduce**

    var a = [1, 2, 3];
    var sum = _.reduce(a, function(memo, num){
      return memo + num
    }, 0)
    console.log(sum)

    => 输出 6

**示例 2 : reduceRight**

    var list = [[0, 1], [2, 3], [4, 5]];
    var flat = _.reduceRight(list, function(a, b) { return a.concat(b); }, []);
    console.log(flat)

    => [4, 5, 2, 3, 0, 1]

### find

`_.find(list, predicate, [context])` 别名: `detect`  <br> 
遍历 `list`, 返回第一个通过 `predicate` 函数的**元素值**, 没有通过返回 `undefined` <br>
如果找到符合判断的值, 立即返回, 不会遍历整个 `list` <br>
`predicate` 函数包含 `3` 个属性, `value`, `index`, `list`

这里用到了几个方法    <br>
[createIndexFinder()](#createIndexFinder)  <br>
[_.findIndex()](#findIndex) 

    _.find = _.detect = function(obj, predicate, context){
      var key;
      if (isArrayLike(obj)) {
        key = _.findIndex(obj, predicate, context);
      } else {
        key = _.findKey(obj, predicate, context);
      }
      if (key !== void 0 && key !== -1) return obj[key];
    };

**示例**

    var a = [1, 2, 3, 4];
    var b = _.find(a, function(num){
      return num % 2 == 0
    })
    console.log(b)

    => 2

### filter

`_.filter(list, predicate, [context])` 别名 `select`  <br>
遍历 `list` 的每个值, 返回一个包含所有通过 `predicate` 函数的元素值的集合    <br>
`predicate` 函数包含 `3` 个属性, `value`, `index`, `list`

    _.filter = _.select = function(obj, predicate, context){
      var results = [];
      predicate = cb(predicate, context);
      _.each(obj, function(value, index, list){
        if(predicate(value, index, list)) results.push(value)
      })

      return results;
    }

**示例**

    var a = [1, 2, 3, 4];
    var b = _.filter(a, function(num){
      return num % 2 == 0
    })
    console.log(b)

    => [2, 4]

### reject

`_.reject(list, predicate, [context])`  <br>
返回 `list` 中没有通过 `predicate` 函数检测的元素集合, 与 `filter` 相反

    _.reject = function(obj, predicate, context){
      return _.filter(obj, _.negate(cb(predicate)), context)
    }

**示例**

    var a = [1, 2, 3, 4];
    var b = _.reject(a, function(num){
      return num % 2 == 0
    })
    console.log(b)

    => [1, 3]

## 数组方法

--- 

<div id="createIndexFinder"></div>

### 获取 Index 方法

创建一个可以从左、从右获取 `index` 值的方法

    function createIndexFinder(dir){
      return function(array, predicate, context){
        predicate = cb(predicate, context);
        var length = array != null && array.length;
        var index = dir > 0 ? 0 length - 1;
        for(;index >=0 && index < length; index += dir){
          // 获取符合条件的 key, 返回 index
          if(predicate(array[index], index, array)) return index;
        }

        return -1;
      }
    }

<div id="findIndex"></div>
    
    // 从左开始
    _.findIndex = createIndexFinder(1);

    // 从右开始
    _.findLastIndex = createIndexFinder(-1);

## 对象方法

---

<div id="keys"></div>

### 枚举

返回对象的可枚举属性和方法的名称, 返回一个数组 <br>
这是用到的了 [_.has()](#has), 因为 `for ... in ... ` 可获取到继承自原型的属性

    // IE9下面枚举 BUG 处理

    /**
    *  propertyIsEnumerable()是用来检测属性是否属于某个对象的,如果检测到了,返回true,否则返回false.
    *  1.这个属性必须属于实例的,并且不属于原型.
    *  2.这个属性必须是可枚举的,也就是自定义的属性,可以通过for..in循环出来的.
    *  只要符合上面两个要求,就会返回true;
    */
    var hasEnumBug = !{toString : null}.propertyIsEnumerable("toString")
    console.log(hasEnumBug)
    var hasEnumBug = !{toString: null}.propertyIsEnumerable('toString');
    var nonEnumerableProps = ['valueOf', 'isPrototypeOf', 'toString',
                    'propertyIsEnumerable', 'hasOwnProperty', 'toLocaleString'];

    function collectNonEnumProps(obj, keys) {
      var nonEnumIdx = nonEnumerableProps.length;
      var constructor = obj.constructor;
      var proto = (_.isFunction(constructor) && constructor.prototype) || ObjProto;

      var prop = 'constructor';
      if (_.has(obj, prop) && !_.contains(keys, prop)) keys.push(prop);

      while (nonEnumIdx--) {
        prop = nonEnumerableProps[nonEnumIdx];
        if (prop in obj && obj[prop] !== proto[prop] && !_.contains(keys, prop)) {
          keys.push(prop);
        }
      }
    }   

    // 返回对象的可枚举属性和方法的名称, 返回一个数组
    _.keys = function(obj){
      if(!_.isObject(obj)) return []
      if(nativeKeys) return nativeKeys(obj)
      var keys = []
      // 遍历 obj 中的属性
      for(var key in obj)
        // 找到 obj中的可枚举的属性, 放到 keys 中
        if(_.has(obj, key)) keys.push(key)

      // 处理IE9下面的不能通过 for key in ... 遍历
      if(hasEnumBug) collectNonEnumProps(obj, keys);
      return keys;
    }

<div id="has"></div>

### has

判断属性是否是对象的实例属性, 换句话说, 不是来自 prototype

    // 判断属性是否是对象的实例属性, 换句话说, 不是来自 prototype
    _.has = function(obj, key){
      return obj != null && hasOwnProperty.call(obj, key)
    }

## 函数方法

---

### negate

 返回一个新的 `predicate` 函数的否定版本

    _.negate = function(predicate){
      return function(){
        return !predicate.apply(this, arguments)
      }
    }

### compose 

`_.compose(*functions)` <br>
返回函数集 `functions` 组合后的复合函数, 也就是一个函数执行完再把函数执行结果作为参数传递给下个函数执行

    _.compose = function(){
      var args = arguments;
      // 最后一个参数的 index 值
      var start = args.length - 1;
      return function(){
        var i = start;  
        var result = args[start].apply(this, arguments)
        while (i--) result = args[i].call(this, result)
        return result;
      }

**示例**

    var personName = function(name){
      return name;
    };
    var personal = function(str){
      return "我叫" + str + "!";
    }
    var introduction = _.compose(personal, personName);
    console.log(introduction("刘一凡"))

    => 我叫刘一凡!

### after

`_.after(count, function)` <br>
创建一个函数, 运行 `count` 次后, 触发 `function`

    _.after = function(times, func){
      return function(){
         if(--times < 1){
           return func.apply(this, arguments)
         }
      }
    }

**示例**

    function a(){
      console.log("a")
    }

    var ConsoleA = _.after(3, a);

    ConsoleA();  // 没触发 a()
    ConsoleA();  // 没触发 a()
    ConsoleA();  // 触发 a(), 输出 a

### before

`_.before(count, function)` <br>
创建一个函数, 调用 `function` 不超过 `count` 次, 第 `count` 次返回, 不执行 `function`

    _.before = function(times, func){
      var memo ;
      return function(){
        if(--times > 0){
          memo = func.call(this, arguments)
        }
        if(times <= 1) func = null;

        return memo;
      }
    }

**示例**

    function a(){
      console.log("a")
    }

    var ConsoleA = _.before(3, a);

    ConsoleA();  // 触发 a()
    ConsoleA();  // 触发 a()
    ConsoleA();  // 没触发 a()


## 公用函数

---

### identity

返回本身

    _.identity = function(value){
      return value;
    }
