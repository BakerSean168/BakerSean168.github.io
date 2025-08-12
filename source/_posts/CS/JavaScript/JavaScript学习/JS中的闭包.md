---
title: JS中的闭包
categories:
  - JavaScript
description: JS中的闭包
date: 2025-01-19 09:20:22
tags:
---

# JS 中的闭包

## 什么是闭包

### 定义

闭包是指有权利访问另一个函数作用域中变量的函数，创建闭包的一般的形式就是在一个函数内创建另一个函数

闭包 一个函数和词法环境的引用捆绑在一起，这样的组合就是闭包（closure）。一般就是一个函数 A，return 其内部的函数 B，被 return 出去的 B 函数能够在外部访问 A 函数内部的变量，这时候就形成了一个 B 函数的闭包，A 函数执行结束后这个 A 函数的变量也不会被销毁，并且这个变量在 A 函数外部只能通过 B 函数访问。

### 闭包形成的原理：

作用域链，当前作用域可以访问上级作用域中的变量

### 闭包解决的问题：

能够让函数作用域中的变量在函数执行结束之后不被销毁，同时也能在函数外部可以访问函数内部的局部变量。

### 闭包带来的问题：

由于垃圾回收器不会将闭包中变量销毁，于是就造成了内存泄露，内存泄露积累多了就容易导致内存溢出。

### 加分回答

闭包的应用，能够模仿块级作用域，能够实现柯里化，在构造函数中定义特权方法、Vue 中数据响应式 Observer 中使用闭包等。

## 使用

- 通过在外部调用闭包函数，从而在外部访问到函数内部的变量，可以使用这个方法创建私有变量
- 使已经运行结束的函数上下文中的变量继续留在内存中。保存变量对象的引用，不会被垃圾回收机制回收

### 使用案例

#### 1. 闭包，通过点击链接，实现改变body的字体大小

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <a href="#" id="size-12">12</a>
    <a href="#" id="size-14">14</a>
    <a href="#" id="size-16">16</a>
    <script>
        function makeSize(size) {
            return function () {
                document.body.style.fontSize = size + 'px';
            }
        }
        document.getElementById('size-12').onclick = makeSize(12);
        document.getElementById('size-14').onclick = makeSize(14);
        document.getElementById('size-16').onclick = makeSize(16);
    </script>
</body>

</html>

```

这里通过闭包让内部的函数记住了创建时的 `size` 变量，保持对 size 的引用。

#### 2. 闭包模拟私有方法

```js

/**
 * 闭包模拟私有方法
 */
var makeCounter = function () {
    var privateCounter = 0; //私有变量
    function changeBy(val) { //私有变量
        privateCounter += val;
    }
    return {
        increment: function () {
            changeBy(1);
        },
        decrement: function () {
            changeBy(-1);
        },
        value: function () {
            return privateCounter;
        }
    }
}
// console.log(makeCounter().value()); // 0
var count1 = makeCounter();
var count2 = makeCounter();
console.log(count1.value()); //0
count1.increment();
console.log(count1.value()); //1
console.log(count2.value()); // 0
count2.increment();
count2.increment();
console.log(count2.value()); //2

```


# 参考

https://juejin.cn/post/7120242566729039903
