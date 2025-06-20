---
title: JS中的数组
categories:
  - JavaScript
description: 简单地讲一下 JS 中的数组,Array是什么，怎么创建数组，常用的数组方法
date: 2025-01-18 11:50:59
tags:
---

# Array

## 是什么

### Array 是 JavaScript 中的一个内置类型

除了 Object，Array 应该就是 ECMAScript 中最常用的类型了。ECMAScript 数组跟其他编程语言的数组有很大区别。跟其他语言中的数组一样，ECMAScript 数组也是一组有序的数据，但跟其他语言不同的是，数组中每个槽位可以存储任意类型的数据。这意味着可以创建一个数组，它的第一个元素是字符串，第二个元素是数值，第三个是对象。ECMAScript 数组也是动态大小的，会随着数据添加而自动增长。

在 JavaScript 中，Array 是一种特殊的对象，用于存储有序的、可变长度的数据集合。并且提供了许多内置方法来操作数组。

**基本特点**
- 有序集合：元素按插入顺序排列，可通过索引访问（如 `arr[0]`）
- 动态大小：可以随时添加或删除元素，长度自动调整。
- 可存储任意类型：同一个数组可以混合存放数字、字符串、对象等。

```js
const arr = [1, "hello", { name: "Alice" }, [2, 3]];
```

### Array 也是一个内置构造函数

```js
console.log("typeof Array:",typeof Array); // Function
```

## 创建数组的方法

- 使用数组字面量创建
  不会调用 Array 构造函数
- 调用构造函数创建

#### 数组空位

`const options = [,,,,,]; // 创建包含 5 个元素的数组`

## 数组的使用

高级语言一般都会内置常用的、有各自特点的数组使用方法。  
学习某一种语言的任务之一就是：学习这些方法，在使用中调用。  
所以学习数组的使用就是学习内置的数组相关的方法、特性、原理。  

### 检测数组的相关方法

1. instanceof  
   使用 instanceof 的问题是假定只有一个全局执行上下文。如果网页里有多个框架，则可能涉及两
   个不同的全局执行上下文，因此就会有两个不同版本的 Array 构造函数。如果要把数组从一个框架传
   给另一个框架，则这个数组的构造函数将有别于在第二个框架内本地创建的数组。
2. Array.isArray()

```js
let arr = [1, 2, 3];
console.log(Array.isArray(arr)); // true
console.log(arr instanceof Array); // true
console.log(arr.constructor === Array); // true
console.log(arr.__proto__ === Array.prototype); // true
console.log(Object.getPrototypeOf(arr) === Array.prototype); // true
console.log(Array.prototype.isPrototypeOf(arr)); // true
console.log(Object.prototype.toString.call(arr).slice(8,-1)); // [object Array] -> Array
```

### 数组常用方法

数组常用方法的关键在：  
- 数组的作用
- 返回值
- 使用特点

| 方法名                                     | 作用                                | 返回值                         | 示例                                       |
| ------------------------------------------ | ----------------------------------- | ------------------------------ | ------------------------------------------ |
| **`push(item)`**                           | 在数组末尾添加元素                  | 新数组长度                     | `arr.push(5);` → 长度+1                    |
| **`pop()`**                                | 删除并返回最后一个元素              | 被删除元素                     | `const last = arr.pop();`                  |
| **`unshift(item)`**                        | 在数组开头添加元素                  | 新数组长度                     | `arr.unshift(0);` → 长度+1                 |
| **`shift()`**                              | 删除并返回第一个元素                | 被删除元素                     | `const first = arr.shift();`               |
| **`splice(start, deleteCount, ...items)`** | 在指定位置增删/替换元素             | 被删除元素的数组               | `arr.splice(1, 1, 'a');` → 替换索引 1 元素 |
| **`slice(start, end)`**                    | 截取数组片段（不修改原数组）        | 新数组                         | `const sub = arr.slice(0, 2);`             |
| **`indexOf(item)`**                        | 查找元素首次出现的索引              | 索引（未找到返回 `-1`）        | `arr.indexOf('apple');` → 0 或 -1          |
| **`find(callback)`**                       | 返回满足条件的第一个元素            | 元素（未找到返回 `undefined`） | `arr.find(x => x > 5);`                    |
| **`filter(callback)`**                     | 返回满足条件的所有元素的新数组      | 新数组                         | `arr.filter(x => x % 2 === 0);`            |
| **`map(callback)`**                        | 遍历处理元素并返回新数组            | 新数组                         | `arr.map(x => x * 2);`                     |
| **`forEach(callback)`**                    | 遍历执行操作（无返回值）            | `undefined`                    | `arr.forEach(x => console.log(x));`        |
| **`reduce(callback, initialValue)`**       | 累加处理数组元素                    | 最终累加结果                   | `arr.reduce((sum, x) => sum + x, 0);`      |
| **`sort([compareFn])`**                    | 排序数组（默认按字符串 Unicode 码） | 排序后的原数组                 | `arr.sort((a, b) => a - b);`               |
| **`reverse()`**                            | 反转数组元素顺序                    | 反转后的原数组                 | `arr.reverse();` → `[3, 2, 1]`             |
| **`concat(arr2)`**                         | 合并多个数组                        | 新数组                         | `const merged = arr1.concat(arr2);`        |
| **`join(separator)`**                      | 将数组转为字符串（指定分隔符）      | 字符串                         | `arr.join(', ');` → `"a, b, c"`            |
| **`includes(item)`**                       | 检查是否包含某元素                  | 布尔值                         | `arr.includes('apple');` → `true/false`    |
| **`some(callback)`**                       | 检查是否有至少一个元素满足条件      | 布尔值                         | `arr.some(x => x > 10);`                   |
| **`every(callback)`**                      | 检查是否所有元素都满足条件          | 布尔值                         | `arr.every(x => x > 0);`                   |

#### reduce

累加器

数组扁平化
```js

let arr = [1, 2, [3, 4,], [5, 6]];
function flatArray(arr) {
    return arr.reduce((result, item) => {
        if (Array.isArray(item)) {
            return [...result, ...flatArray(item)];
        } else {
            return [...result, item];
        }
    }, []);
}
console.log(flatArray(arr)); // [1, 2, 3, 4, 5, 6]

```