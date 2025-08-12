---
title: Conventional Commits 规范
categories:
  - Git
description: Conventional Commits 规范
date: 2025-06-21 22:02:58
tags:
---

[Conventional Commits 规范官网](https://www.conventionalcommits.org/en/v1.0.0/)

## 什么是 Conventional Commits

Conventional Commits 是一种提交消息格式规范，旨在让提交历史更清晰、自动化工具更易于解析。它通过约定提交消息的结构，帮助团队实现自动化版本管理、生成变更日志等。

## 基本格式

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

- **type**：提交类型（必填），如 feat、fix、docs、style、refactor、test、chore 等。
- **scope**：影响范围（可选），如模块名、文件名。
- **description**：简要描述（必填），一句话说明本次提交的目的。
- **body**：详细说明（可选），对本次提交的详细描述。
- **footer**：脚注（可选），如 BREAKING CHANGE 或关联 issue。

## 常见 type 类型

- **feat**：新功能
- **fix**：修复 bug
- **docs**：文档变更
- **style**：代码格式（不影响功能，如空格、分号等）
- **refactor**：重构（既不新增功能也不修复 bug）
- **test**：增加或修改测试
- **chore**：构建过程或辅助工具变动

## 示例

```
feat(login): 新增用户登录功能

fix(api): 修复接口返回数据格式错误

docs(readme): 更新使用说明

refactor: 优化数据处理逻辑

chore: 升级依赖包
```

## 规范优势

- 统一提交格式，便于团队协作
- 支持自动生成 CHANGELOG
- 支持自动化版本发布（如 semantic-release）
- 便于代码审查和历史追踪

