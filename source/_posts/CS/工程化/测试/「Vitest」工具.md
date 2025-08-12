---
title: 「Vitest」工具
categories:
  - null
description: 「Vitest」工具
date: 2025-08-07 22:15:07
tags:
---

[Vitest 官方文档](https://cn.vitest.dev/?from=home-page.cn)

## 安装与配置

`pnpm add -D vitest @vitest/ui @vitejs/plugin-vue vue-tsc`

```json
"scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui"
  },
```

```ts
export default defineConfig({
  test: {
    include: ['src/**/*.test.ts', 'electron/**/*.test.ts'],
    environment: 'node', // 默认 node
    environmentMatchGlobs: [
      ['src/**/*.test.ts', 'jsdom'],      // src 下测试用 jsdom
      ['electron/**/*.test.ts', 'node'],  // electron 下测试用 node
    ],
  },
})
```