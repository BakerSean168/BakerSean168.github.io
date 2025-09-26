---
title: 数据库（INDEX）
categories:
  - 数据库
description: 数据库相关知识的起点文章
date: 2025-08-24 15:33:24
tags:
  - index
---

### 分区表

- 分区表的主键必须包含分区键
- 分区键为时间，字段和值都要使用 `TO_DAYS()`

### 删除检查点

```SQL
RELEASE SAVEPOINT s1;
```