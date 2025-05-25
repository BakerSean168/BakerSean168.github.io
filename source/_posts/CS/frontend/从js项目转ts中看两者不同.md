---
title: 从js项目转ts中看两者不同
categories:
  - 前端
  - JS
  - TS
description: 从js项目转ts中看两者不同
date: 2025-05-19 16:46:29
tags:
---

将一个 express 的 js 项目 转为 ts 项目，要做哪些事

以下是根据你的需求补充的完整文章，结合了从 JavaScript 项目迁移到 TypeScript 的关键步骤和两者核心差异，并引用了多个来源的实践建议：

---

# 从 JS 项目转 TS 中看两者不同

## 引言
TypeScript（TS）作为 JavaScript（JS）的超集，通过静态类型系统和现代语法特性（如装饰器、接口）显著提升了代码的可维护性和开发体验。将 Express.js 项目从 JS 迁移到 TS 需要系统性调整，以下从**文件修改**、**配置迁移**和**核心差异**三部分展开。

---

## 一、文件修改：语法与结构的迁移

### 1. 文件后缀与模块语法
- **修改文件后缀**：将 `.js` 改为 `.ts`，TS 编译器会自动处理这些文件。  
- **模块系统转换**：  
  - **CommonJS → ES Modules**：  
    ```javascript
    // Before (CJS)
    const express = require('express');
    module.exports = app;
    
    // After (ESM)
    import express from 'express';
    export default app;
    ```  
    注意：ESM 要求导入路径包含扩展名（如 `'./utils.js'`），而 TS 允许省略（编译时自动补全）。

### 2. 类型注解与接口
- **参数类型**：为函数参数、返回值和变量添加类型注解：  
  ```typescript
  // JS
  function add(a, b) { return a + b; }
  
  // TS
  function add(a: number, b: number): number { return a + b; }
  ```  
- **接口与类型别名**：定义复杂数据结构：  
  ```typescript
  interface User {
    id: number;
    name: string;
  }
  const user: User = { id: 1, name: 'Alice' };
  ```

### 3. 第三方库类型支持
- 安装 `@types` 包：例如 `@types/express` 提供 Express 的类型定义。  
- 若库无官方类型，可手动声明：  
  ```typescript
  declare module 'untyped-library' {
    export function foo(): void;
  }
  ```

---

## 二、配置迁移：工具链与编译设置

### 1. 初始化 TS 配置
- **生成 `tsconfig.json`**：  
  ```bash
  npx tsc --init
  ```  
  关键配置项：  
  ```json
  {
    "compilerOptions": {
      "target": "ES2017",         // 编译目标版本
      "module": "commonjs",      // 模块系统（兼容 Node.js）
      "outDir": "./dist",        // 输出目录
      "strict": true,            // 启用严格类型检查
      "esModuleInterop": true    // 兼容 CJS/ESM 混合导入
    },
    "include": ["src/**/*.ts"],  // 仅编译 src 目录
    "exclude": ["node_modules"]
  }
  ```  
  推荐继承预设配置（如 `@tsconfig/node22`）以减少手动配置。

### 2. 开发与生产脚本
- **开发阶段**：使用 `ts-node` 或 `ts-node-dev` 实时编译执行：  
  ```json
  "scripts": {
    "dev": "ts-node-dev --respawn src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js"
  }
  ```  
  `ts-node-dev` 类似 `nodemon`，支持热重载。  
- **生产部署**：通过 `postinstall` 脚本自动编译：  
  ```json
  "postinstall": "npm run build"
  ```  
  确保部署时仅推送 `dist` 目录的 JS 文件。

---

## 三、TS 与 JS 的核心差异

### 1. 静态类型系统
- **编译时类型检查**：TS 在编译阶段捕获类型错误（如未定义属性、参数类型不匹配），而 JS 需运行时才能发现。  
- **类型擦除**：TS 的类型注解在编译后会被完全移除，不影响运行时性能。

### 2. 高级语言特性
- **装饰器**：通过 `@experimentalDecorators` 启用，用于类、方法或属性的元编程：  
  ```typescript
  @Controller('/users')
  class UserController {
    @Get('/')
    getAll() { /* ... */ }
  }
  ```  
  需在 `tsconfig.json` 中启用。  
- **泛型与工具类型**：提供类型复用能力：  
  ```typescript
  type Response<T> = { data: T; error: string };
  const res: Response<User> = { data: user, error: null };
  ```

### 3. 工程化优势
- **代码可维护性**：类型注解和接口显式定义契约，减少团队协作的歧义。  
- **工具链支持**：TS 与 VSCode 深度集成，提供智能提示、重构和跳转定义。

---

## ts 编译器的行为

TypeScript（TS）转换为 JavaScript（JS）的过程主要通过 TypeScript 编译器（tsc） 完成，其核心流程包括 静态类型检查、语法转换和模块处理。以下是详细的转换机制和步骤：  

1. **扫描器（Scanner）**
   - 输入：源代码
   - 输出：令牌（Token）流
   - 功能：识别关键字、标识符等语法单元

2. **解析器（Parser）**
   - 输入：令牌流
   - 输出：抽象语法树（AST）
   - 功能：分析代码结构，构建语法树

3. **绑定器（Binder）**
   - 输入：AST
   - 输出：符号表
   - 功能：建立符号（Symbol）与 AST 节点的关联

4. **类型检查器（Checker）**
   - 输入：AST 和符号表
   - 功能：验证类型是否符合声明
   - 作用：确保类型安全

5. **发射器（Emitter）**
   - 输入：经过类型检查的 AST
   - 输出：JavaScript 代码
   - 功能：
     - 移除类型注解
     - 转换 TS 特有语法（如接口、枚举）
     - 生成兼容的 JS 代码


TypeScript到JavaScript的转换本质是 静态类型检查 + 语法降级，通过编译器将高级类型语法转为浏览器或Node.js可执行的JS代码。  

## 总结
迁移到 TypeScript 不仅是语法转换，更是工程实践的升级。通过类型系统、现代语法和工具链整合，TS 能够显著提升 Express 项目的健壮性和开发效率。对于遗留项目，建议逐步迁移（如单个文件转换并验证），而非一次性重构。

> **关键区别总结**：  
> - **TS 是静态类型语言，JS 是动态类型语言**。  
> - **TS 编译为 JS 后类型信息消失**，但编译阶段已确保类型安全。  
> - **TS 支持更多高级特性**（如装饰器、泛型），适合大型应用开发。  

--- 

