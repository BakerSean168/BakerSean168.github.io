---
title: MCP
categories:
  - null
description: MCP
date: 2025-09-01 13:24:47
tags:
  - AI
  - MCP
---

## 基础概念

[MCP document](https://modelcontextprotocol.io/docs/getting-started/intro)

一个定义了大语言模型如何调用工具的协议  

- MCP Host
  AI 应用（调用管理 MCP clients）
- MCP Client
  和一个 MCP 服务器一对一连接，用来从服务器获取上下文并给 MCP Host 使用。
- MCP Server
  提供上下文给客户端。  
  工具、资源、提示语等所在的地方  

流程：

1. 客户端向指定地址发起询问
2. 服务端返回自己有的工具、资源等
3. 用户提出问题时，客户端会请求调用需要的工具，来通过工具完整任务