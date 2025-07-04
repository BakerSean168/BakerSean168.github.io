---
title: Cookie和LocalStorage的区别
categories:
  - null
description: Cookie和LocalStorage的区别
date: 2025-07-02 21:13:14
tags:
---

# Cookie 和 LocalStorage 的区别（面试回答版）

作为被面试者，我会从以下几个方面来回答 Cookie 和 LocalStorage 的区别：

## 1. 存储容量
- **Cookie**：存储容量较小，通常限制在 **4KB** 左右。每个域名下一般最多只能存储约 20 个 Cookie。
- **LocalStorage**：存储容量大得多，现代浏览器通常提供 **5MB** 左右的存储空间。

## 2. 生命周期
- **Cookie**：
  - 可以设置过期时间（通过 `expires` 或 `max-age` 属性）
  - 如果没有设置过期时间，则是会话级别的，浏览器关闭后自动删除
- **LocalStorage**：
  - **永久存储**，除非用户手动清除或通过 JavaScript 删除
  - 数据不受页面刷新、浏览器关闭或系统重启影响

## 3. 数据传输
- **Cookie**：
  - 每次 HTTP 请求都会自动携带同源的 Cookie
  - 这会增加网络流量，可能影响性能
- **LocalStorage**：
  - 数据仅存储在客户端，不会自动发送到服务器
  - 需要手动通过 JavaScript 读取和设置

## 4. 作用域
- **Cookie**：
  - 可以通过 `domain` 和 `path` 属性设置作用域
  - 默认情况下，同源的所有页面共享 Cookie
- **LocalStorage**：
  - 受同源策略限制，只能被同源页面访问
  - 同源的不同标签页和窗口间共享数据

## 5. API 和使用方式
- **Cookie**：
  - 操作相对复杂，需要通过 `document.cookie` 字符串操作
  - 需要手动解析键值对
- **LocalStorage**：
  - 提供简单的键值对 API：`setItem()`、`getItem()`、`removeItem()` 等
  - 操作更方便，支持直接存储对象（需 JSON 序列化）

## 6. 安全性
- **Cookie**：
  - 可以设置 `HttpOnly` 标志防止 JavaScript 访问，提高安全性
  - 可以设置 `Secure` 标志仅通过 HTTPS 传输
  - 但仍易受 CSRF 攻击
- **LocalStorage**：
  - 没有类似 Cookie 的安全标志
  - 易受 XSS 攻击，恶意脚本可读取全部数据
  - 不适合存储敏感信息

## 7. 典型应用场景
- **Cookie**：
  - 用户身份认证（存储 Session ID 或 Token）
  - 需要服务器端读取的小数据（如用户偏好）
- **LocalStorage**：
  - 客户端大量数据存储（如用户设置、缓存数据）
  - 前后端分离架构中的客户端状态管理
  - 需要持久化但不需要与服务器交互的数据

## 补充说明

在实际应用中，选择使用 Cookie 还是 LocalStorage 需要根据具体需求决定：
- 如果需要与服务器交互的小数据（如认证信息），优先考虑 Cookie
- 如果是纯客户端存储的大量数据，优先考虑 LocalStorage
- 对于敏感数据，Cookie 的 `HttpOnly` 和 `Secure` 标志能提供更好保护
