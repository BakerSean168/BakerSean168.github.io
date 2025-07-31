---
title: 「node-schedule」库的使用
categories:
  - null
description: 「node-schedule」是 Node.js 的一个灵活的 cron 类和非 cron 类作业调度器。它允许您计划作业（任意函数）在特定日期执行，并具有可选的重复规则。它在任何给定时间只使用一个计时器（而不是每秒/每分钟重新评估即将到来的作业）。
date: 2025-07-16 10:32:12
tags:
---

[node-schedule Github](https://github.com/node-schedule/node-schedule)

- node-schedule 继承了 nodejs 中的 EventEmitter 类
- 有 run、scheduled、canceled、error、success 事件

### Cron 式调度

使用 Cron 格式的时间来调度任务：  

### 日期式调度

使用 Date 对象来调度任务：

### 循环规则调度

使用 rule 对象来调度任务：