---
title: JS原型
categories:
  - JavaScript
description: JS原型
date: 2025-01-15 23:09:45
tags:
---

## 原型的基本概念

### 显式原型（prototype）

- 原型是为了实现对象（函数）属性的继承。
- 当创建一个函数时，也会创建一个 `prototype` 对象（默认原型对象，默认继承自 `Object.prototype`），这个对象包含所有可被实例共享的属性和方法。

```js
function Car(color) {
  this.color = color;
}
Car.prototype.name = "su7-Ultra"; // 所有Car实例共享name属性
const car1 = new Car("red");
console.log(car1.name); // 'su7-Ultra'（通过原型链访问）
```

### 隐式原型（`_proto_`）

- 隐式原型表明自己继承了的属性。
- 每个对象都有一个`_proto_`属性，指向构造函数的 `prototype` 对象。
- Object 是原型链的顶端， `Object.prototype.__proto__ === null`。
- 浏览器中显示的是`[[prototype]]`

```js
console.log(car1.__proto__ === Car.prototype); // true
```

可以用 `_proto_`，来访问自己的原型对象，但现代代码推荐使用`Object.getPrototyprOf()`

## 原型的具体例子

```js
function car() {}
// 创建函数 `car` 时，也会创建一个 `car.prototype` 对象。
let car1 = new car();
// car1 为 `car()` 的实例，它拥有每个对象都有的 `_proto` 属性，指向 `car()` 的 `prototype` 对象，来继承这些属性
console.log(car.prototype.__proto__ === Object.prototype);
// `car.prototype` 对象的 `__proto__` 属性指向 `Object.prototype`（原型对象的构造函数）。
console.log(car.__proto__ === Function.prototype);
// car 本身是一个函数对象，是 Function 的实例，它的构造函数就是 Function，它也继承了 Function 的属性。
console.log(car1.__proto__ === car.prototype);
// car1 是 car 构造的实例，car1 继承了 car 的属性。
```

## 原型链的运作机制

### 属性查找

当访问对象的属性时：

1. 在对象自身查找
2. 通过`_proto_`在原型对象找
3. 一直到找到属性或到达原型链顶端 Object

## 构造函数与原型

### 构造函数的作用

通过 `new` 调用构造函数时：  
- 创建一个实例对象；
- 将其 `_proto_` 指向构造函数的 `prototype`；
- 执行构造函数代码

```js

function Person(name) { this.name = name; }
Person.prototype.greet = function() { console.log(`Hello, ${this.name}`); };
const p = new Person('Alice');
p.greet(); // "Hello, Alice"

```

### constructor 属性

原型对象默认有一个 constructor 属性指向构造函数

```js

console.log(Person.prototype.constructor === Person); // true

```

## 类与原型

ES6 中的 `Class` 类的本质就是原型的语法糖，是基于原型实现的。  