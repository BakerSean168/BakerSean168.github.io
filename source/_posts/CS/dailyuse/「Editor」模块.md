---
title: 「Editor」模块
categories:
  - null
description: 「Editor」模块
date: 2025-09-21 18:28:39
tags:
  - dailyuse
  - editor
  - monaco
---

你现在是一个精通 vscode 编辑器和 typora md编辑器的 全栈工程师。
请你基于 一下技术或者其他更好的技术来实现一个 类似 typora 的md编辑器。
"monaco-editor": "^0.52.2",
"monaco-editor-vue3": "^0.1.10",

首先先确定技术架构：你可以从 prompt 文件中知道对应说明的文件的路径，都在.github/prompts目录下

根据已有的架构，你应该先采用面向对象+DDD+contracts first。先确定好一个 editor 领域应该有的聚合根、实体、值对象以及他们的结构