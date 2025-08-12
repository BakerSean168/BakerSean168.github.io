---
title: 「Husky+lint-staged」
categories:
  - null
description: 「Husky+lint-staged」
date: 2025-08-05 22:22:24
tags:
---

下面详细讲讲**自动化与 CI/CD**中常用的 Husky、lint-staged 和 GitHub Actions 配置方法和原理：

---

## 1. Husky + lint-staged：提交前自动检查

### **Husky**  
Husky 用于管理 Git 钩子（如 pre-commit），让你在提交代码前自动执行脚本。

### **lint-staged**  
lint-staged 只对本次提交的文件运行 lint/format，速度快、效率高。

### **配置步骤**

#### 1. 安装依赖

```bash
pnpm add -D husky lint-staged
```

#### 2. 初始化 Husky

```bash
npx husky install
```
在 package.json 添加：
```json
"scripts": {
  "prepare": "husky install"
}
```
然后执行一次 `pnpm run prepare`。

#### 3. 添加 pre-commit 钩子

```bash
npx husky add .husky/pre-commit "npx lint-staged"
```

#### 4. 配置 lint-staged

在 package.json 添加：

```json
"lint-staged": {
  "*.{js,ts,vue}": [
    "eslint --fix",
    "prettier --write"
  ]
}
```
这样，每次 `git commit` 前，只有本次提交的 JS/TS/Vue 文件会被自动 lint 和格式化。

---

## 2. GitHub Actions：自动化测试、构建和发布

GitHub Actions 是 GitHub 提供的 CI/CD 服务，可以在推送、PR、发布等事件时自动执行脚本。

### **典型配置：自动测试和构建**

新建 `.github/workflows/ci.yml`：

````yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
        with:
          version: 8
      - name: Install dependencies
        run: pnpm install
      - name: Lint
        run: pnpm run lint
      - name: Build
        run: pnpm run build
      # 可以加测试、打包、发布等步骤
````

---

## 3. 总结

- **Husky + lint-staged**：保证每次提交的代码都经过自动检查和格式化，提升代码质量，防止不规范代码进入仓库。
- **GitHub Actions**：自动化测试、构建、发布，保证主干分支始终可用，减少人为失误。

如需更详细的某一项配置或遇到具体报错，欢迎继续提问！

你的理解很接近，但它们**不是重复**，而是**配合使用，分工不同**：

# Q/A

## Husky + lint-staged 这个是不是和 prettier+eslint 重复了？

### 1. **Prettier + ESLint**

- 作用：负责代码格式化（Prettier）和代码规范检查/修复（ESLint）。
- 触发方式：可以在 VSCode 保存时自动执行，也可以手动运行脚本（如 `pnpm run lint`、`pnpm run format`）。

---

### 2. **Husky + lint-staged**

- 作用：在**Git 提交前**，自动对本次提交的代码运行 ESLint/Prettier（或其他脚本），确保提交的代码是规范和格式化过的。
- 触发方式：只在 `git commit` 前自动执行，且只检查/修复本次提交的文件，速度快。

---

### **配合效果**

- VSCode 插件和保存时自动格式化，提升本地开发体验，但开发者可以选择不开启或忽略。
- Husky + lint-staged 是**最后一道防线**，强制保证所有提交到仓库的代码都符合规范和格式要求，防止不规范代码混入主干。

---

**总结：**  
- Prettier/ESLint 负责“怎么检查和修复”。
- Husky/lint-staged 负责“什么时候强制检查和修复（提交前）”。
- 推荐两者配合，保证开发体验和团队代码质量。
