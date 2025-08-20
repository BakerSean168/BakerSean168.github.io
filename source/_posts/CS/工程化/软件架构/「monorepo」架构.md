---
title: 「monorepo」架构
categories:
  - null
description: 「monorepo」架构
date: 2025-08-10 12:13:48
tags:
---

# 基础概念

monorepo 将多个项目放在同一个仓库中，便于统一管理，复用代码，提升开发效率。  

- 怎么复用代码（提取代码，项目中引用代码）

## contracks

在 Monorepo 的上下文中，Contracts（契约） 指的是项目或包之间约定的交互规则，确保代码的稳定性和可维护性。这些契约可以是显式的（如 TypeScript 接口、API 文档）或隐式的（如文件路径约定）。

## Q&A

### **Q1 Domain vs Contracts 的职责分工**

我使用 DDD 架构，需要复用一些 domain，以下以 Account 为例。  
我有 Account aggregate  `export class Account extends AggregateRoot implements IAccount{}`，显然他有`AggregateRoot`、`IAccount`等接口。  
```
domain/
├── aggregates/
├── entities/
├── events/
├── types/
└── valueObjects/
```

当前是吧 AggregateRoot 从 util 包中导出的，IAccount 则是从 types 中导出。  
此时 contracks 有什么用呢

**A1**

1. **Domain 包负责**：
   - ✅ **业务逻辑**：Account 聚合的业务行为
   - ✅ **业务规则**：账号验证、状态转换规则
   - ✅ **领域概念**：IAccount、AccountStatus 等领域接口
   - ✅ **数据完整性**：包含敏感信息如 passwordHash、验证令牌

2. **Contracts 包负责**：
   - ✅ **API 契约**：前后端通信的数据格式
   - ✅ **数据验证**：Zod Schema 用于请求验证
   - ✅ **序列化格式**：去除敏感信息的安全数据传输
   - ✅ **版本控制**：API 接口的版本管理

这两个负责的并不相同，直接在 domain 的 types 中定义要使用的接口就好了。不用管 Contracts 包。

