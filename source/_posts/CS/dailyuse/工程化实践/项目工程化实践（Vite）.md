---
title: 项目工程化实践（Vite）
categories:
  - null
description: 好好研究一下 Electron+Vue3 项目的工程化实践（Vite），Vite 的配置使用。
date: 2025-08-05 13:23:32
tags:
---

打包流程：

1. 打包 Vue3 前端代码，生成静态文件（Vite 来生成）
2. Electron 主进程入口指向 Vue3 静态文件
3. 运行 Electron 打包工具，生成 Electron 包（electron-builder）

## 原始代码分析

<details>
<summary>原始代码</summary>

```ts
import { defineConfig } from "vite";
import path from "node:path";
import electron from "vite-plugin-electron/simple";
import vue from "@vitejs/plugin-vue";
import monacoEditorPlugin from "vite-plugin-monaco-editor";
import fs from "node:fs";

// 自动发现插件目录
const pluginsDir = path.resolve(__dirname, "src/plugins");
const plugins = fs
  .readdirSync(pluginsDir)
  .filter((file) => fs.statSync(path.join(pluginsDir, file)).isDirectory())
  .filter((dir) => fs.existsSync(path.join(pluginsDir, dir, "index.html")));

// 构建入口配置
const buildInputs = {
  main: path.resolve(__dirname, "index.html"),
  ...Object.fromEntries(
    plugins.map((plugin) => [
      plugin,
      path.resolve(__dirname, `src/plugins/${plugin}/index.html`),
    ])
  ),
};

// 收集所有预加载脚本
const preloadInputs = {
  main_preload: path.join(__dirname, "electron/preload.ts"),
  login_preload: path.join(__dirname, "electron/preload/loginPreload.ts"),
  ...Object.fromEntries(
    plugins
      .filter((plugin) =>
        fs.existsSync(path.join(pluginsDir, plugin, "electron/preload.ts"))
      )
      .map((plugin) => [
        `${plugin}_preload`,
        path.join(pluginsDir, plugin, "electron/preload.ts"),
      ])
  ),
};

// 原生模块列表
const nativeModules = ["better-sqlite3", "bcrypt", "electron"];

// https://vitejs.dev/config/
export default defineConfig({
  resolve: {
    alias: {
      "~": path.resolve(__dirname, "./"),
      "@": path.resolve(__dirname, "./src"),
      src: path.resolve(__dirname, "./src"),
      "@electron": path.resolve(__dirname, "./electron"),
      "@common": path.resolve(__dirname, "./common"),
    },
  },
  base: "./",
  build: {
    rollupOptions: {
      input: buildInputs,
      output: {
        inlineDynamicImports: false,
        manualChunks: undefined,
      },
      // 在主构建中也排除原生模块
      external: nativeModules,
    },
  },
  plugins: [
    (monacoEditorPlugin as any).default({
      languageWorkers: ["editorWorkerService", "json"],
    }),
    vue(),
    electron({
      main: {
        entry: "electron/main.ts",
        vite: {
          resolve: {
            alias: {
              "@": path.resolve(__dirname, "./src"),
              "@electron": path.resolve(__dirname, "./electron"),
              "@common": path.resolve(__dirname, "./common"),
            },
          },
          build: {
            outDir: "dist-electron",
            rollupOptions: {
              external: nativeModules,
              output: {
                format: "es",
              },
            },
          },
          optimizeDeps: {
            exclude: nativeModules,
          },
        },
      },
      preload: {
        input: preloadInputs,
        vite: {
          build: {
            outDir: "dist-electron",
            rollupOptions: {
              external: nativeModules,
              output: {
                inlineDynamicImports: false,
                manualChunks: undefined,
                entryFileNames: "[name].mjs",
              },
            },
          },
          optimizeDeps: {
            exclude: nativeModules,
          },
        },
      },

      renderer: process.env.NODE_ENV === "test" ? undefined : {},
    }),
  ],
});
```

</details>

### `vite-plugin-electron` 插件

#### 具体来说，vite-plugin-electron 做了什么？

1. **主进程与预加载脚本的热更新和打包**

   - 让 Electron 主进程（如 main.ts）和 preload 脚本也能用 Vite 的开发体验（如热更新、TypeScript、别名等）。
   - 自动为主进程和 preload 启动独立的 Vite 构建流程，输出到指定目录。

2. **多入口管理**

   - 你可以像配置前端一样，配置主进程和 preload 的入口、输出、依赖排除等。

3. **开发时自动重启 Electron**

   - 当主进程代码变更时，自动重启 Electron 应用，提升开发效率。

4. **统一依赖管理和路径别名**
   - 让主进程、preload、renderer 都能用同一套路径别名和依赖排除配置。

---

#### 为什么不用 Vite 直接打包主进程？

- Vite 默认只支持浏览器环境（前端），主进程和 preload 需要 Node 环境支持，且打包方式不同。
- vite-plugin-electron 帮你桥接了 Vite 和 Electron 的差异，让开发和打包 Electron 应用变得更简单、统一。

---

#### 总结

- **Vite**：只管前端（Vue、React 等）代码。
- **vite-plugin-electron**：让 Vite 也能管理和打包 Electron 主进程、预加载脚本，实现一体化开发体验。

是的，这些配置（如 `main`、`preload` 下的 `vite` 配置）都是 `vite-plugin-electron` 插件提供的专用配置入口。插件会自动帮你用 Vite 启动和打包 Electron 主进程、预加载脚本，并支持热更新、路径别名、依赖排除等。

---

#### 如果**不用插件**，你需要手动做这些事：

1. **主进程和 preload 脚本的打包**

   - 你要单独写 Vite 或 Rollup 配置文件（如 `vite.main.config.ts`、`vite.preload.config.ts`），分别指定入口、输出、依赖排除、别名等。
   - 用命令行分别运行这些配置进行打包。

2. **开发时热重载**

   - 需要自己写脚本监听主进程/preload 文件变动，自动重启 Electron 应用（比如用 `nodemon` 或自定义 watcher）。

3. **路径别名和依赖排除**

   - 需要在每个配置文件里重复写别名、external 配置，维护成本高。

4. **多入口和多 preload 管理**
   - 需要自己处理多入口的输入输出映射，不能像插件一样自动发现和管理。

---

#### 举例：不用插件时主进程的 Vite 配置

```typescript
// vite.main.config.ts
import { defineConfig } from "vite";
import path from "node:path";

export default defineConfig({
  build: {
    outDir: "dist-electron",
    lib: {
      entry: "electron/main.ts",
      formats: ["cjs"],
    },
    rollupOptions: {
      external: ["electron", "better-sqlite3", "bcrypt"],
      output: {
        entryFileNames: "main.js",
      },
    },
  },
  resolve: {
    alias: {
      "@electron": path.resolve(__dirname, "./electron"),
      "@common": path.resolve(__dirname, "./common"),
    },
  },
});
```

然后用命令行分别打包：

```bash
vite build --config vite.main.config.ts
vite build --config vite.preload.config.ts
vite build           # 前端
```

---

**总结：**  
不用插件时，你需要为主进程、preload、前端分别维护配置和打包流程，开发体验和自动化程度都不如用 `vite-plugin-electron`。插件就是帮你把这些流程集成到一起，极大简化了 Electron + Vite 项目的开发和维护。

## 我还能做什么

都说 vite 这类打包工具能做很多事来优化这个项目工程，但我现在好像就用了路径别名，我还能做什么呢

### 自动引入组件/函数

使用 unplugin-auto-import 和 unplugin-vue-components 实现自动导入 Vue 组件、常用函数，无需手写 import。  
感觉在学习的过程中，会影响学习，暂时不考虑。

### 代码分割与懒加载

说是利用动态 `import()`，让大页面或大组件按需加载，减少首屏包体积。  
我确实已经在 vue-router 中使用了 `import()` 来动态加载各个页面了。

### 环境变量与多环境配置

利用 `.env` 文件和 `import.meta.env`，实现开发、测试、生产环境的自动切换和配置隔离。

### 静态资源优化

- 配置 vite-plugin-imagemin 自动压缩图片。
- 使用 Vite 的 assetsInclude 处理自定义静态资源类型。

### 代码质量保障

- 集成 ESLint、Prettier 插件，自动格式化和检查代码风格。
- 配置 TypeScript 严格模式，提升类型安全。

[已配置，配置文档](/source/_posts/CS/dailyuse/工程化实践/配置「eslint-Prettier」.md)

### Mock 数据与接口代理

- 使用 vite-plugin-mock 实现本地 mock 数据，前后端并行开发。
- 配置 server.proxy，解决本地开发跨域问题。

### 构建分析与性能优化

- 使用 rollup-plugin-visualizer 分析打包体积，定位大包来源。
- 配置 build.target，让输出代码更兼容或更小。

### 自动化与 CI/CD

- 配合 Husky、lint-staged 实现提交前自动检查。
- 配置 GitHub Actions 等自动化测试、构建和发布。

[Husky 配置参考](/source/_posts/CS/dailyuse/工程化实践/「Husky-lint-staged」.md)

### PWA 支持

- 使用 vite-plugin-pwa 让你的应用支持离线访问和桌面安装。

### 多页面应用（MPA）支持

- Vite 支持多入口配置，适合多页面场景。
