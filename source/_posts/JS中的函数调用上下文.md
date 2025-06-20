---
title: JS中的函数执行上下文
categories:
  - null
description: JS中的函数执行上下文
date: 2025-01-18 19:51:37
tags:
---

# JS 中的执行上下文

## 前置知识

- 作用域

## 基础知识

变量或函数的上下文决定了它们可以访问哪些数据，以及它们的行为。

执行上下文的三种类型：

- 全局执行上下文：全局执行环境是最外围的一个执行环境，在浏览器的全局对象是 window, this 指向这个对象。
- 函数执行上下文：可以有无数个，函数被调用的时候会被创建。每次调用函数都会创建一个新的执行上下文。
- eval 执行上下文，很少用。

每个执行上下文，都有三个重要属性：

- 变量对象 (variable object, VO)  
   每个执行环境都有一个与之关联的变量对象，环境中定义的所有变量和函数都保存在这个对象中。虽然我们编写的代码无法访问这个对象，但解析器在处理数据时会在后台使用它。
- 作用域链（scope chain）  
   当代码在一个环境中执行时，会创建变量对象的一个作用域链。作用域链的用途，是保证对执行环境有权访问的所有变量和函数的有序访问。
- this  
   普通函数指向函数的调用者：有个简便的方法就是看函数前面有没有点，如果有点，那么就指向点前面的那个值;  
   箭头函数指向函数所在的所用域：注意理解作用域，只有函数的{}构成作用域，对象的{}以及 if(){}都不构成作用域;

## this

this 的值取决于函数的调用方式，而不是定义位置。JavaScript 中有五种主要的 this 绑定规则。

### 默认绑定

当函数作为独立函数调用时，this 的绑定规则：

- 非严格模式：this 指向全局对象（浏览器中为 window）
- 严格模式：this 为 undefined

```js
function printThis() {
  return this;
}

console.log(printThis() === window);
// 非严格模式：true
// 严格模式：false
```

### 隐式绑定（方法调用）

当函数作为对象方法调用时，`this` 绑定到该对象：

```js
function printThis() {
  return this;
}

const obj = { printThis };
console.log(obj.printThis() === obj); // true
```

**隐式丢失问题**

当方法被赋值给变量或作为回调传递时，可能丢失`this`绑定

```js
const bar = obj.printThis;
bar(); // 此时 this 指向全局对象或 undefined
```

### 显式绑定

使用 `call()` `apply()` `bind()` 方法显示设置 `this`：

```js
function greet() {
  return `Hello, ${this.name}!`;
}

const person = { name: "Alice" };
greet.call(person); // "Hello, Alice!"
greet.apply(person); // "Hello, Alice!"

const boundGreet = greet.bind(person);
boundGreet(); // "Hello, Alice!"
```

### new 绑定

使用 `new` 调用构造函数时，`this` 绑定到新创建的对象：

```js
function Dog(name) {
  this.name = name;
}
const myDog = new Dog("Buddy");
console.log(myDog.name); // "Buddy"
```

new 操作符执行以下步骤:

- 创建一个新对象
- 将新对象的 [[Prototype]] 链接到构造函数的原型
- 将 this 绑定到新对象
- 如果函数没有返回其他对象，则返回这个新对象

### 箭头函数绑定

箭头函数没有自己的 `this`，它继承自外层函数或全局作用域的 `this`:

```js
const obj = {
  name: "Bob",
  greet: function () {
    const arrowFunc = () => {
      return `Hello, ${this.name}!`;
    };
    return arrowFunc();
  },
};
console.log(obj.greet()); // "Hello, Bob!"
```

箭头函数的 this 无法通过 `call()` `apply()` `bind()` 改变。

### 优先级

1. new
2. 显示
3. 隐式
4. 默认

```js
function foo(a) {
  this.a = a;
}
var obj1 = {};
var bar = foo.bind(obj1);
bar(2);
console.log(obj1.a); // 2

var baz = new bar(3);
console.log(obj1.a); // 2
console.log(baz.a); // 3
```

## `call()` `apply()` `bind()`区别

相同点：

- 都是用来改变 this 的指向的

不同点：

- call 第一个参数是要给 this 的绑定值，后面是一个参数列表
- apply 第一个参数是要给 this 的绑定值，后面是一个参数数组
- bind 第一个参数是要给 this 的绑定值，后面一个是参数列表，返回的是一个函数，不是立即执行的，需要调用；而 call，apply 是立即执行的

如果 call 方法没有参数，或者参数为 null 或 undefined 或 this，则等同于指向全局对象

# 题目

## 题目一

```js
var name = 'window';
var obj1 = {
    name: "花非花",
    fn1: function () { //正常属性
        console.log(this.name);
    },
    fn2: () => console.log(this.name), //箭头函数
    fn3: function () { //闭包
        return function () {
            console.log(this.name);
        }
    },
    fn4: function () {
        return () => console.log(this.name); //返回箭头函数
    }
}
var obj2 = {
    name: '雾非雾'
};
obj1.fn1();//花非花 ，隐式绑定
obj1.fn1.call(obj2);//雾非雾 ， 显式绑定

obj1.fn2();//undefined
// 箭头函数继承外部函数或全局作用域，由于外部没有函数作用域（obj1 不是函数，是对象），所以为全局作用域，指向 window

obj1.fn2.call(obj2);//undefined
// 箭头函数的 this 无法通过 `call()` `apply()` `bind()` 改变。

obj1.fn3()();//undefined
// 隐式丢失问题？？
// 等价于
// var fn = obj1.fn3();
// fn();

obj1.fn3().call(obj2);//雾非雾
// 等价于
// var fn  = obj1.fn3();
// fn.call(obj2); // 改变了this的指向

obj1.fn3.call(obj2)();//undefined
// 等价于
// var fn = obj1.fn3.call(obj2);
// window.fn(); //默认绑定

obj1.fn4()();//花非花
// 箭头函数的 this 指向外部函数作用域 

obj1.fn4().call(obj2);//花非花
// 箭头函数的 this 无法通过 `call()` `apply()` `bind()` 改变。

obj1.fn4.call(obj2)();//雾非雾
// 改变了 外部函数的指向
```

## 题目二

```js
var name = "window";
function Person(name) {
    this.name = name;
    this.fn1 = function () {
        console.log(this.name);
    },
    this.fn2 = () => console.log(this.name),
    this.fn3 = function () {
        return function () {
            console.log(this.name);
        }
    },
    this.fn4 = function () {
        return () => console.log(this.name);
    }
}
var obj1 = new Person('花非花');
var obj2 = new Person('雾非雾');

obj1.fn1(); //花非花
//

obj1.fn1.call(obj2);//雾非雾
// 显式绑定

obj1.fn2();//花非花
// 箭头函数继承外部函数作用域

obj1.fn2.call(obj2);//花非花
// 箭头函数 this 绑定后不会改变

obj1.fn3()();//undefined
// 隐式丢失？？
// 相当于执行一个匿名函数

obj1.fn3().call(obj2);//雾非雾
// 给匿名函数显式绑定

obj1.fn3.call(obj2)();//undefined
// 还是匿名函数

obj1.fn4()();//花非花
// 继承外部函数作用域

obj1.fn4().call(obj2);//花非花
obj1.fn4.call(obj2)();//雾非雾
```

# 参考来源

https://juejin.cn/post/7119882395741847565#heading-12