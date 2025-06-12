---
title: ES6模块导入导出方式
categories:
  - 前端
description: ES6模块导入导出方式
date: 2025-06-03 18:19:58
tags:
---

# 模块化基础

ES6 模块化的核心是通过 export 导出模块功能，通过 import 导入其他模块的功能。  

模块化的主要优势：  
- 代码重用
- 更好的依赖管理
- 易于维护
- 封装性
- 性能优化：支持懒加载

# 基本导出方式

## 命名导出

命名导出
- 允许一个模块导出多个值
- 每个导出都有一个名称
- 导入时必须使用相同名称
- 适用于导出多个相关功能

### 内联命名导出

在声明时导出
```ts

export const PI = 3.14159;
export function add(a, b) {
  return a + b;
}
export function subtract(a, b) {
  return a - b;
}
```

### 统一命名导出

统一导出
```ts

const PI = 3.14159;
function add(a, b) {
  return a + b;
}
function subtract(a, b) {
  return a - b;
}

export { PI, add, subtract }; // 统一导出
```

## 默认导出

- 每个模块只能有一个默认导出
- 导入时可以任意命名
- 适用于模块的主要功能或主要类

```ts

export default function add(a, b) {
  return a + b;
}
```

## 默认导出+命名导出

# 导出进阶用法

## 导出重命名

在命名导出中使用 as 来重命名:
避免命名冲突或提供更语义化的名称
```ts

const PI = 3.14159;
function add(a, b) {
  return a + b;
}

export { PI as piValue, add as sum }; // 导出时重命名
```

## 重新导出

可以从其他模块导入内容并立即导出：  
创建聚合模块或库的入口文件
```ts
export { add } from './math.js';
export * from './math.js'; // 导出所有命名导出（不包括默认导出）
```

# 导入

## 命名导入

当模块使用命名导出时,可以使用命名导入：  

```js
// math.js
export function add(a, b) { return a + b; }
export const PI = 3.1415926;

// main.js
import { add, PI } from './math.js';
console.log(add(2, 3)); // 5
console.log(PI); // 3.1415926
```

命名导入要求导入的名称必须与导出的名称一致。

## 默认导入

当模块使用默认导出时：  

```js
// math.js
export default function add(a, b) { return a + b; }

// main.js
import add from './math.js';
console.log(add(2, 3)); // 5
```

默认导入不需要使用花括号，且可以为导入的成员指定任意名称。

## 命名空间导入

使用`* as`语法可以将模块的所有命名导出作为一个命名空间对象导入：

```js
import * as math from './math.js';
console.log(math.add(2, 3)); // 5
console.log(math.PI); // 3.1415926
```
适合需要导入模块大量成员的情况。

## 混合导入

```js
import { default as add, PI } from './math.js';
```

## 仅导入副作用

如果只需要执行模块的代码而不需要导入任何成员，可以只写导入语句：  

```js
import './module.js'
```

# 导入的特殊用法

## 重命名导入

可以使用as关键字为导入的成员重命名：

```JavaScript
import { add as sum, PI as piValue } from './math.js';
```

这在避免命名冲突时非常有用。

## 动态导入

ES2020引入了动态导入语法import()，它返回一个Promise：

```JavaScript
import('./math.js')
  .then(module => {
    console.log(module.add(2, 3));
  });
```

动态导入适用于按需加载、条件加载等场景。

## 导入字符串名称的成员

如果模块导出的成员名称不是有效的JavaScript标识符（如包含连字符），需要使用字符串形式导入：

```JavaScript
// module.js
export { someValue as "some-value" };

// main.js
import { "some-value" as someValue } from './module.js';
```
