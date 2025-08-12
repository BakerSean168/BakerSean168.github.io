---
title: ipc事件处理器注册不成功
categories:
  - null
description: ipc事件处理器注册不成功
date: 2025-07-11 22:02:13
tags:
---

### 1. 代码出错，导致没生效
```
// Authentication 模块导出
export { AuthenticationService } from './application/services/authenticationApplicationService';

```

好像是一处报错，导致部分代码没有执行，导致初始化代码没有生效
 Database initialized
✓ Task completed: database
Executing initialization task: filesystem
✓ Filesystem handlers registered
✓ Task completed: filesystem
Executing initialization task: account-ipc-handlers
🔄 [AccountIpc] 开始初始化账号IPC处理器...
✅ [AccountIpc] Account IPC handlers registered
✅ [AccountIpc] 账号IPC处理器初始化完成
✓ Account IPC handlers registered
✓ Task completed: account-ipc-handlers
Executing initialization task: account-event-handlers
[Authentication] 注册事件处理器...
✓ Account event handlers registered
✓ Task completed: account-event-handlers
Executing initialization task: authentication-ipc-handlers
📝 [EventBus] 订阅事件: AccountStatusVerificationResponse
📝 [EventBus] 订阅事件: AccountIdGetterResponse
🚀 [AuthenticationIpc] 启动登录请求处理
✅ [AuthIpc] Authentication IPC handlers registered
✓ Authentication IPC handlers registered
✓ Task completed: authentication-ipc-handlers
Executing initialization task: git
✓ Git handlers registered
✓ Task completed: git
Executing initialization task: notification
✓ Notification handlers registered
✓ Task completed: notification
Executing initialization task: schedule
✓ Schedule handlers registered
✓ Task completed: schedule
Executing initialization task: eventSubscription
🔧 [EventSubscription] 开始初始化事件订阅
📝 [EventBus] 订阅事件: AccountRegistered
🔐 [EventSubscription] Authentication 模块事件处理器初始化完成
✅ [EventSubscription] 事件订阅初始化完成
✓ Event subscriptions initialized
✓ Task completed: eventSubscription
Executing initialization task: task-ipc-handlers
✓ Task IPC handlers registered
✓ Task completed: task-ipc-handlers
Executing initialization task: goal-module
🎯 正在初始化 Goal 模块...
✅ Goal IPC handlers registered
✅ [Goal事件处理器] 事件处理器注册完成
✅ Goal 模块初始化完成
✓ Goal module initialized
✓ Task completed: goal-module
Completed initialization phase: APP_STARTUP
✓ Application initialization completed
✅ [Main] 应用初始化完成
🎯 [Main] 主进程初始化完成
[32288:0711/232335.118:ERROR:CONSOLE(1)] "R


### 2. 有同名事件，覆盖了