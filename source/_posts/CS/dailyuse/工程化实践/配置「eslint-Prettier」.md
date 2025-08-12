---
title: 配置「eslint+Prettier」
categories:
  - null
description: 配置「eslint+Prettier」
date: 2025-08-05 22:04:24
tags:
---

下面是适用于 TS 项目的 ESLint/Prettier 配置方案，并对每个配置项添加了详细注释。

---

### 1. 安装依赖

```bash
pnpm add -D eslint prettier @typescript-eslint/eslint-plugin @typescript-eslint/parser eslint-plugin-vue eslint-config-prettier eslint-plugin-prettier
```
> 安装 ESLint、Prettier 及其 TypeScript、Vue 支持插件和相关集成。

---

### 2. 配置 ESLint

新建 `.eslintrc.js`：

```javascript
module.exports = {
  root: true, // 指定当前为根配置文件，防止 ESLint 查找父级配置
  env: {
    node: true,    // 启用 Node.js 全局变量和作用域
    browser: true, // 启用浏览器全局变量
  },
  extends: [
    "eslint:recommended", // 使用 ESLint 推荐规则
    "plugin:vue/vue3-recommended", // 启用 Vue3 推荐规则
    "plugin:@typescript-eslint/recommended", // 启用 TypeScript 推荐规则
    "plugin:prettier/recommended", // 启用 Prettier 推荐规则，解决与 ESLint 的冲突
  ],
  parser: "vue-eslint-parser", // 使用 Vue ESLint 解析器，支持 .vue 文件
  parserOptions: {
    parser: "@typescript-eslint/parser", // 指定 TypeScript 解析器
    ecmaVersion: 2020, // 支持的 ECMAScript 版本
    sourceType: "module", // 使用 ES 模块
  },
  rules: {
    // 可自定义规则
    "@typescript-eslint/no-explicit-any": "warn", // 使用 any 类型时警告
  },
};
```

eslint9 开始，ESLint 默认配置文件为 `eslint.config.js`。

`pnpm create @eslint/config` 回答问题来创建一个 ESLint 配置文件。*用的 ts 文件，有坑*

```ts
import globals from "globals";
import tseslint from "typescript-eslint";
import pluginVue from "eslint-plugin-vue";

export default tseslint.config([ // 使用 ts-eslint，否则下面的`tseslint.configs.recommended`会报类型不匹配错误
  {
    files: ["**/*.{js,mjs,cjs,ts,mts,cts,vue}"],
    languageOptions: { globals: { ...globals.browser, ...globals.node } },
    
  },
  
  tseslint.configs.recommended,
  pluginVue.configs["flat/essential"],
  { files: ["**/*.vue"],
    languageOptions: { parserOptions: { parser: tseslint.parser } },
    rules: {
      'vue/multi-word-component-names': 'off'
    }
  },
  {
    rules: {
      '@typescript-eslint/no-explicit-any': 'off',
      "@typescript-eslint/no-unused-vars": "off" // 不能正确识别Vue组件用大写引入，小写使用的组件，所以关了
    }
  },
]);
```



---

### 3. 配置 Prettier

新建 `.prettierrc`：

```json
{
  "semi": true,           // 句末加分号
  "singleQuote": true,    // 使用单引号
  "printWidth": 100       // 每行最大长度为 100，超出自动换行
}
```

`From ESLint v9.0.0, the default configuration file is now eslint.config.js.`


### 4. TypeScript 配置建议

确保 tsconfig.json 开启严格模式：

```json
// filepath: tsconfig.json
{
  "compilerOptions": {
    "strict": true,                        // 启用所有严格类型检查选项
    "noImplicitAny": true,                 // 禁止隐式 any 类型
    "strictNullChecks": true,              // 严格的 null 检查
    "module": "ESNext",                    // 使用 ESNext 模块系统
    "target": "ESNext",                    // 编译为 ESNext 语法
    "moduleResolution": "Node",            // Node 风格的模块解析
    "esModuleInterop": true,               // 允许默认导入非 ES 模块
    "skipLibCheck": true,                  // 跳过库文件的类型检查，加快编译速度
    "forceConsistentCasingInFileNames": true // 文件名大小写一致性检查
  }
}
```

---

### 5. 脚本与 VSCode 配置

- package.json 脚本：

  ```json
  {
    "scripts": {
      "lint": "eslint --ext .js,.ts,.vue src electron", // 检查 src 和 electron 目录下的 JS/TS/Vue 文件
      "format": "prettier --write \"src/**/*.{js,ts,vue}\"" // 格式化 src 目录下的 JS/TS/Vue 文件
    }
  }
  ```

- VSCode 推荐设置（settings.json）：

  ```json
  {
    "editor.formatOnSave": true, // 保存时自动格式化
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": true // 保存时自动修复所有 ESLint 问题
    }
  }
  ```