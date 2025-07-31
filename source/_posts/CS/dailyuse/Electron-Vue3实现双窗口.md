---
title: Electron+Vue3实现双窗口
categories:
  - null
description: Electron+Vue3实现双窗口
date: 2025-07-13 22:06:11
tags:
---

electron 的窗口管理  
## 需求说明

希望实现像 QQ 这样的登录界面与应用分离，登录成功后跳转主页面。

---

## 技术要点

- Electron 的多窗口管理
- 进程间通信（IPC）
- 认证信息的安全传递

---

## 业务流程

### 1. 登录窗口流程

1. 用户在登录窗口输入账号密码，点击登录。
2. 登录窗口通过 IPC 向主进程发起登录请求。
3. 主进程校验账号、密码，生成 token，并确保账户信息已存在于 accounts 表，再插入 token。
4. 登录成功后，主进程将 token、accountId 等认证信息临时存储在主进程内存（如 authSession 或全局变量）。
5. 主进程关闭登录窗口，打开主窗口，并通过 `additionalArguments: ['--window-type=main']` 标记主窗口类型。

### 2. 主窗口流程

1. 主窗口启动时，通过 `process.argv` 或 `window.env.argv` 判断自己是主窗口。
2. 主窗口初始化阶段（如 `MainWindowInit.initialize()`）通过 IPC（如 `authentication:get-login-info`）向主进程请求登录信息。
3. 主进程返回内存中的 token、accountId 等认证信息。
4. 主窗口收到后，将 token、accountId 写入自己的前端 store。
5. 主窗口用这些认证信息初始化用户数据（如调用 `accountLoggedService.initAccountInfo(accountId)`），并进行后续业务初始化。

---

## 状态传递与隔离

### 问题

登录窗口和主窗口的状态不通用。登录成功的状态信息（如 accountId、token）只保存在登录窗口，主窗口无法直接获取。

如果直接执行初始化代码，会导致登录窗口和主窗口都执行，状态混乱。

### 解决方案

1. 登录窗口登录成功后，主进程创建主窗口时通过 `additionalArguments` 传递认证参数。
2. 或主进程主动推送认证信息（auth 信息）到主窗口。