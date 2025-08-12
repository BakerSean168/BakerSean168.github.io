---
title: 「GitHub-Actions」
categories:
  - null
description: GitHub Actions 是 GitHub 提供的 CI/CD 服务。本文旨在介绍什么是 GitHub Actions，以及如何使用 GitHub Actions 创建 CI/CD 流程（构建项目）。
date: 2025-08-05 22:25:25
tags:
---



## 什么是 CI/CD

CI/CD 是**持续集成（Continuous Integration）/ 持续交付或持续部署（Continuous Delivery/Deployment）**的缩写，是现代软件开发流程中的核心理念。

---

### 1. **CI（持续集成）**

- **Continuous Integration**
- 指团队成员频繁地（通常是每天多次）将代码集成到主干分支。
- 每次集成都自动运行构建、测试、代码检查等流程，及时发现和修复问题。
- 目的是让代码始终保持可用、可测试、可发布的状态。

### 2. **CD（持续交付/持续部署）**

- **Continuous Delivery**：代码通过自动化流程构建、测试后，自动生成可发布的产物（如安装包、镜像等），但是否上线由人工决定。
- **Continuous Deployment**：在持续交付基础上，自动将产物部署到生产环境，实现全自动上线。

---

### 3. **为什么叫 CI/CD？**

- 这是一套自动化流程，贯穿代码提交、自动测试、自动构建、自动发布等环节。
- 让开发、测试、运维协作更高效，减少人为失误，提升交付速度和质量。

---

### 4. **GitHub Actions 在 CI/CD 中的作用**

- 你配置的 workflow（如 release.yml）就是一个 CI/CD 流程：
  - 自动拉取代码、安装依赖、构建产物、上传发布。
  - 保证每次发布都可复现、可追踪、自动化。

## `googleapis/release-please-action` 配置

[官方GitHub](https://github.com/googleapis/release-please-action)  

### 遇到的问题

1. 权限问题
action 自带的 token 权限不够，应该使用 pat（Actions、Contents、Metadata、Pull requests）  
[配置 pat 的权限分配](https://github.com/googleapis/release-please-action/issues/818)  

2. `"Error: release-please failed: Error adding to tree"`报错

[Error: release-please failed: Error adding to tree](https://github.com/googleapis/release-please-action/issues/938)  
[release-please 管理的文件不能包括 Workflow 文件](https://github.com/sugurutakahashi-1234/mermaid-markdown-wrap/pull/28)  

放弃支持 workflow 文件，所以如果修改了 workflow 文件，要手动 release。
