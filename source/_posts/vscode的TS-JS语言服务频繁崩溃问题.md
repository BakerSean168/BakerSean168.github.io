---
title: vscode的TS/JS语言服务频繁崩溃问题
categories:
  - null
description: vscode的TS/JS语言服务频繁崩溃问题
date: 2025-06-11 18:36:12
tags:
---

# 起因

突然就发现鼠标瞄到代码上的类型提示等没了，一直时 loading 状态，发现右下角 提示：  
  The JS/TS language service crashed 5 times in the last 5 Minutes.
  This may be caused by a plugin contributed by one of these extensions: GitHub.copilot-chat, Vue.volar
  Please try disabling these extensions before filing an issue against VS Code.

# 解决

发现还有 vscode 更新的消息，更新完重启就好了