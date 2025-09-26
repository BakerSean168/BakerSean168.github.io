---
title: agent-tars
categories:
  - null
description: agent-tars
date: 2025-09-15 18:57:38
tags:
---

在配置 copilot mcp 服务时， csdn 上看到了 Copilot MCP 插件，搜索 filesystem 插件安装，然后点击弹出来的说明页上的 repo，结果跑到了 [AGENT-TARS GitHub 仓库](https://github.com/bytedance/UI-TARS-desktop)  

## 介绍

Agent TARS 是一个开源的 Multimodal AI Agent，旨在通过丰富的多模态能力（如 GUI Agent、Vision） ，并与各种现实世界工具无缝集成，其内置了常用的工具（Search，Browser、File、Command 等），来不断探索一种能够更接近人类完成任务的工作形态。

## 安装与配置

```powershell
# 全局安装
pnpm install @agent-tars/cli@latest -g

# 创建 （Global，？，官方文档是这么写的，但是感觉实际上就是创建普通 workspace） Workspace（在想要的文件夹）
agent-tars workspace --init

# 通过 vscode 打开
agent-tars workspace --open
```

### Global Workspace 的具体影响

当你完成 Global Workspace 的创建后，对你带来的具体影响是，无论你在什么目录下运行 agent-tars：

- Global Workspace 中的 Config File 都将生效；
- Agent 的 workspace 将会被设定为 ~/.agent-tars-workspace；
- File System 将指向 Global Workspace，也就是说，LLM 通过 File 工具输出的文件都将位于这个目录；
- Agent TARS CLI 中一些依赖 File System 的能力也会指向这个目录，如 Snapshot。