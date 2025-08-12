---
title: 前后端的DTO数据同步
categories:
  - null
description: 前后端的DTO数据同步
date: 2025-08-12 14:54:45
tags:
---

在 Electron+Vue3 中，我采用主进程和渲染进程都使用相似的 domain 文件（相同属性、DTO方法）来确保主进程和渲染进程之间传输的对象一致。  
正常前后端分离的项目中，是前后端都使用相同的 dto 文件来确保前后端之间传输的对象一致的吗，还是说有什么最佳实践