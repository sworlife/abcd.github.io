---
title: 无 new 操作符构造对象
date: 2019/7/14 10:00:00
tags: javaScript
categories: 前端技术
comments: true
---

## 前言
我们知道，使用一个构造函数来创建对象时，需要使用 `new` 操作符。但在日常开发过程中，往往又会发现很多框架或者库（例如：jQuery、underscore）创建新对象时并没有使用 `new` 操作符。这样方便了大家书写代码，也显得更易读。那么，他们是怎么实现的呢？

## 解决方案
总的来说，这些方案都并没有很神奇的突破了 `JavaScript` 语法限制。而是通过一些巧妙的逻辑（**共享原型**、**原型判断**）来实现。

### 方案1——共享原型

``` js
// 构造函数
var Class = function(){
    return new init();
}
Class.prototype = {};

var init = function(){
  // 构造对象的逻辑
  // ...
};
init.prototype = Class.prototype

// 创建对象
var object = Class();

```
上述解决方案可以看出，构造函数`Class`调用时，返回了通过 `new` 操作符来运行 `init` 方法。但因为 `init` 的原型与 `Class` 的原型指向同一个对象，因此 `new init()` 创建出的对象拥有 `Class` 的原型。

### 方案2——原型判断

``` js
var Class = function(){
    if( !(this instanceof Class) ) {
      return new Class();
    };
    // 构造对象的逻辑
    // ...
}
Class.prototype = {}

// 创建对象
var object = Class();
```
坦白说，这个方案需要对 `this` 的指向理解比较深刻。当执行 `Class()` 时，`this` 为 `undefined`(严格模式) 或 `window`。此时，经过 `if` 判断后，会执行 `new Class()`。再次执行时，根据 `new` 操作符的运行机制，`this` 指向的是将要创建出的对象，这个对象的原型肯定就是 `Class`，因此，经过 `if` 判断就不会继续执行 `new`, 而是向下执行构造对象的逻辑。

