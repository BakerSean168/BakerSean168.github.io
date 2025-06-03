---
title: ES6模块导出方式
categories:
  - 前端
description: ES6模块导出方式
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

# 进阶用法

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