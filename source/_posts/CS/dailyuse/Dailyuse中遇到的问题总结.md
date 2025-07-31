---
title: Dailyuse中遇到的问题总结
categories:
  - null
description: Dailyuse中遇到的问题总结
date: 2025-07-13 18:11:56
tags:
---

### 1. `Uncaught Error: Dynamic require of "path" is not supported`

问 AI 说是错误原因是在渲染进程中使用了 `path` 模块，而 `path` 模块只能被 Node.js 模块使用。  
但是我使用搜索找不到前端中直接使用的 `path`。  
感觉可能是因为我在渲染进程中直接复用了一些主进程的代码，导致一不小心引入了相关模块。  
最终在清除了一些渲染进程的代码后，问题解决（还不清楚具体是什么原因）。  

**总结：**  
1. 主进程和渲染进程代码尽量不要复用了
2. 及时保存代码到仓库，到时候可以直接回退  
