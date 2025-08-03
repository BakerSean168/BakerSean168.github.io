---
title: 「DailyUse」的logout功能实现
categories:
  - null
description: 「DailyUse」的logout功能实现
date: 2025-08-02 19:35:18
tags:
---

实现类似 QQ 的，登出后进入 登录窗口

主进程中实现登出业务：  
传入 sessionUuid、token、username、accountUuid，找到对应的 auth_ssession 并删除，然后发布登出事件。

在渲染进程中：  
得到成功登出后，清除 store 中的数据，并发送 logout 事件到窗口管理器，窗口管理器收到后，关闭主窗口，打开登录窗口。

### 问题1

渲染进程的用户登出逻辑好像有问题，登出，再次登录后，没有重新初始化，还保留着上次登出的信息  
```
authenticationIpcClient.ts:53 [AuthenticationIpcClient] 发送登出事件
useLogout.ts:25 注销结果： {success: true, sessionUuid: 'f1c4f93d-9b5c-4914-9486-43b89b41d53c', accountUuid: '6598b430-6a84-4ba0-bb4e-e9941b931b91', username: 'Test1', message: '注销成功', …}

```

解决方法：  
主进程的窗口管理代码中，不只是 hide 主窗口，而应该直接销毁主窗口，登录后再创建新的主窗口