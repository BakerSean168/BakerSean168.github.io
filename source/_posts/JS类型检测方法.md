---
title: JS类型检测方法
categories:
  - JavaScript
description: JS类型检测方法
date: 2025-01-17 22:34:31
tags:
---



## typeof

最简单的一种方式，可以用来检测原始类型;  
会把 null、array、{} 都判断成 object

```js

console.log(typeof 2);               // number
console.log(typeof true);            // boolean
console.log(typeof 'str');           // string
console.log(typeof []);              // object    
console.log(typeof function(){});    // function
console.log(typeof {});              // object
console.log(typeof undefined);       // undefined
console.log(typeof null);            // object

```

### 为什么 `typeof` 会把 `null` 判断成 object

历史遗留问题。  
在 JavaScript 早期的引擎实现中，值的类型标签存储在 32 位单元的低位。null 的机器码被设计为全零（000），而对象类型的标签恰好也是 000，导致 typeof null 错误地返回 "object" 。  

### 为什么 `array` 和 `{}` 判断成对象

- 数组的本质是特殊对象（具有数字索引和 length 属性的对象）。  
- 对象是 JavaScript 的复合数据类型，typeof 对所有对象（包括内置对象如 Date自定义对象等）统一返回 "object" 。  

### 为什么 `function` 能正确判断，而非返回 object

函数是 JavaScript 的一等公民，既是可执行代码块，也是对象。ECMAScript 规范明确将函数归类为实现了 [[Call]] 内部方法的对象，而 typeof 通过检查这一内部标记来区分函数和其他对象。  

- 函数内部有可调用性标记
- 对象类型标签为 `010` 与 普通对象 `000` 区别

## instanceof

- 只能用来判断引用数据类型
- 底层机制是基于原型

```js

console.log(2 instanceof Number);                    // false
console.log(true instanceof Boolean);                // false 
console.log('str' instanceof String);                // false 
 
console.log([] instanceof Array);                    // true
console.log(function(){} instanceof Function);       // true
console.log({} instanceof Object);                   // true


```

## constructor

- 通过访问对象的 constructor 属性来判断类型

```js

console.log((2).constructor === Number); // true
console.log((true).constructor === Boolean); // true
console.log(('str').constructor === String); // true
console.log(([]).constructor === Array); // true
console.log((function() {}).constructor === Function); // true
console.log(({}).constructor === Object); // true

```

## Object.prototype.toString.call()

- 调用 Object.prototype.toString 方法，这个方法的主要作用是返回一个表示对象类型的字符串`"[object Type]"`

```js

//1.判断基本类型：
console.log(Object.prototype.toString.call(null));//"[object Null]"
console.log(Object.prototype.toString.call(undefined));//"[object Undefined]"
console.log(Object.prototype.toString.call("abc"));//"[object String]"
console.log(Object.prototype.toString.call(123));//"[object Number]"
console.log(Object.prototype.toString.call(true));//"[object Boolean]"

//2.判断原生引用类型：
//函数类型
function fn() { console.log("test"); }
console.log(Object.prototype.toString.call(fn));//"[object Function]"
//日期类型
var date = new Date();
console.log(Object.prototype.toString.call(date));//"[object Date]"
// 数组类型
var arr = [1, 2, 3];
console.log(Object.prototype.toString.call(arr));//"[object Array]"
//正则表达式
var reg = /[h]at/g;
console.log(Object.prototype.toString.call(reg));//"[object RegExp]"


```