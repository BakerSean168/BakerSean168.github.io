---
title: JS中的DOM对象
categories:
  - JavaScript
description: JS中的DOM对象
date: 2025-01-18 15:41:12
tags:
---

# DOM

## 是什么

文档对象模型（DOM，Document Object Model）是 HTML 和 XML 文档的编程接口。  
DOM 表示由多层节点构成的文档，通过它开发者可以添加、删除和修改页面的各个部分。

主要就是提供了使用 JavaScript 来操作 HTML 文档。

### 文档对象

Document Object Model, 文档对象模型

- 元素（element）  
  文档中的所有标签都是元素，元素可以看成是对象
- 节点（node）  
  文档中所有的内容都是节点：标签，属性，文本
- 文档（document）  
  一个页面就是一个文档

文档包含节点，节点包含元素

### 节点层级

任何 HTML 或 XML 文档都可以用 DOM 表示为一个由节点构成的层级结构。  
节点分很多类型，每种类型对应着文档中不同的信息和（或）标记，也都有自己不同的特性、数据和方法，而且与其他类型有某种关系。  
这些关系构成了层级，让标记可以表示为一个以特定节点为根的树形结构。

## 使用

### 相关方法

```js
// 查询相关方法
getElementById();
getElementsByClassName();
getElementsByTagName();
querySelectorAll();
querySelector();
```

使用例子：

```js
// 取：
document.getElementById();

// 获取节点所有属性
node.attributes;

// 获取节点所有子元素
node.childNodes;

// 增：
let a = createElement("a");
document.body.append(a);

// 删：
document.body.removeChild(div);
```

## 现状

在使用 Vue 等框架时，好像已经不使用（不直接使用）DOM 接口了。  
框架通过虚拟 DOM 和 响应式数据绑定机制抽象了大部分 DOM 操作。

虚拟 DOM 算法，数据变化时：

- 生成新的虚拟 DOM 树
- 通过 Diff 算法比较新旧 DOM 树差异
- 更新必要部分
