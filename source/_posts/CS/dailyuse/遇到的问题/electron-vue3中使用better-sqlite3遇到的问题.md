---
title: electron+vue3中使用better-sqlite3遇到的问题
categories:
  - 前端
description: electron+vue3中使用better-sqlite3遇到的问题
date: 2025-05-24 22:43:18
tags:
---

使用 Sqlite 作为 Electron 的数据库时，使用了 better-sqlite3 库  
写了 config userModel.ts userService.ts userIpc.ts 然后注册 ipc 监听文件后，却报错了：  
`ReferenceError: __filename is not defined`  
使用 vscode 的搜索功能，确定这几个文件里面都没有使用到 `--filename`

# 错误1 __filename is not defined

在 main.ts 文件中使用 try catch 排错

```ts
try {
  console.log("testing database...");
  const db = await initializeDatabase();
  console.log("数据库初始化成功", db);
} catch (error) {
  console.error("UserModel 测试失败:", error);
}
```

问 AI 说可能是哪里用到了 `--filename`，可能是哪个库中用到了，可以使用如下代码来提供：  
```ts
// 提供 __filename 和 __dirname
const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

// 将这些添加到全局
globalThis.__filename = __filename;
globalThis.__dirname = __dirname;
```

这里就可以每个文件都尝试一下，来找到问题在哪了。  

此外，发现可以利用 error 抛出错误的顺序来判断真实错误的大致位置： 
```ts
// userService.ts
import path from "path";
import { fileURLToPath } from "url";
import type { TResponse } from "@/shared/types/response";
import type {
  TRegisterData,
  TLoginData,
  TUser,
} from "@/modules/Account/types/user";
import { UserModel } from "../models/userModel";
import bcrypt from "bcrypt";

const __dirname = path.dirname(fileURLToPath(import.meta.url));
``` 
userModel.ts import config  
userService.ts import userModel.ts  
当我在 userService 中制造一个错误时 错误信息 error info: __dirname is not defined in ES module scope 不是我制造的错误；当我在 userModel 中制造错误时，错误信息path is not defined，是我主动制造的错误，是不是说明 __dirname is not defined in ES module scope 错误在 我在userModel文件中制造错误的后面，userService 制造的错误的前面  

# 错误2

Could not dynamically require "xxx". Please configure the dynamicRequireTargets or/and ignoreDynamicRequires option of @rollup/plugin-commonjs appropriately for this require call to work.  

这个错误是因为 better-sqlite3 是一个原生模块（包含 .node 文件），在打包时 Rollup/Vite 无法正确处理动态 require。这是 Electron + Vite + 原生模块的常见问题。  

解决方法（修改vite配置）：  
排除原生模块  
```ts
export default defineConfig({
  resolve: {
    alias: {
      '~': path.resolve(__dirname, './'),
      '@': path.resolve(__dirname, './src'),
      'src': path.resolve(__dirname, './src'),
      '@electron': path.resolve(__dirname, './electron'),
    }
  },
  base: './',
  build: {
    rollupOptions: {
      input: buildInputs,
      output: {
        inlineDynamicImports: false,
        manualChunks: undefined
      },
      // 在主构建中也排除原生模块
      external: nativeModules
    }
  },
  plugins: [
    (monacoEditorPlugin as any).default({
      languageWorkers:['editorWorkerService', 'json']
    }),
    vue(),
    electron({
      main: {
        entry: 'electron/main.ts',
        vite: {
          build: {
            outDir: 'dist-electron',
            rollupOptions: {
              external: nativeModules,
              output: {
                format: 'es'
              }
            }
          },
          optimizeDeps: {
            exclude: nativeModules
          }
        }
      },
      preload: {
        input: preloadInputs,
        vite: {
          build: {
            outDir: 'dist-electron',
            rollupOptions: {
              external: nativeModules,
              output: {
                inlineDynamicImports: false,
                manualChunks: undefined,
                entryFileNames: '[name].mjs'
              }
            }
          },
          optimizeDeps: {
            exclude: nativeModules
          }
        }
      },
      
      renderer: process.env.NODE_ENV === 'test'
        ? undefined
        : {},
    }),
  ],
})
```