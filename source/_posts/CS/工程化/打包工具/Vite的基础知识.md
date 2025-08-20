---
title: Vite的了解
categories:
  - null
description: Vite 和 Webpack 是同一类的打包工具，是前端工程化的基石，Vite 比 WebPack 更新，最先接触的也是 Vite，但没有详细了解过，这篇文章用来记录一下 Vite 的一些知识。
date: 2025-07-13 10:24:07
tags:
---

## 基本概念

[Vite](https://cn.vitejs.dev/guide/)

### Vite 的功能

> 打包工具的功能如下：

1. **将代码资源构建为浏览器能执行的静态资源**：通过依赖图构建来分析每个模块的依赖关系，然后通过资源转换、代码封装、依赖解析来生成静态资源。
2. **代码优化**：通过代码转换工具（如 swc）对代码进行优化，如转译、压缩、混淆、静态资源处理等等。
3. **性能优化**：代码分割、缓存策略、Tree-shaking、代码懒加载等等。
4. **开发环境**：通过开发服务器（如 vite）提供实时更新、热更新、代理、代理缓存等等。

上面是我对打工工具功能的理解，并以次来分析 Vite。

#### 开发环境

Vite 是一个开发环境，它提供了实时更新、热更新、代理、代理缓存等等功能。

1. Vite 使用了 esbuild 来预构建依赖（将以 CommonJS 或 UMD 形式提供的依赖项转换为 ES 模块；将超过 600 个内置模块的 lodash-es，预构建成单个模块），并且使用 esbuild 来快速转换源码（ts、jsx、tsx）
2. 使用了基于原生 ESM 的 bundleless 模式，不需要自己打包模块，而是让浏览器自己加载模块。
3. 使用了基于原生 ESM 的 HMR 模块热更新

#### 代码构建

Vite 的生产环境使用 rollup 进行构建，Vite 采用了 Rollup 灵活的插件 API 和基础建设。  
Rollup 能进行代码分割、tree-shaking、代码懒加载等等。

## 默认配置

通过动态导入会自动切分文件。

## 工程化配置

### 开发配置

开发端口、代理、输出、插件

```ts
export default defineConfig({
  plugins: [
    Vue(),
    {
      name: 'selfDefined',
      transform(code, id) {}
    }
  ],
  server: {
    port: 3232, // 指定本地开发服务器端口为 3232，启动后访问 http://localhost:3232
    proxy: {
      "/api": { // 代理所有以 /api 开头的请求
        target: "https://xxx.xx", // 目标服务器地址，实际请求会被转发到这里
        changeOrigin: true, // 修改请求头中的 origin 字段为目标地址，实现跨域
        rewrite: (path) => path.replace(/^\/api/, "/api"), // 重写路径（可做路径映射），这里是保持 /api 前缀
      },
    },
  },
  build: {
    outDir: "build",
    sourcemap: true,
    minfy: "esbuild"，
    clean: true
    rollupOptions: {
      output: {
        manualChunks: {
          'xxx': ['./src/file_name.js']
        },
        manualChunks(id) {
          if (id.includes('xxx')) {
            return 'xxxx'
          }
        }
      }
    }
  }
})
```

# 问题

## Vite 和 Webpack 有什么区别？详细说说 Vite7 工程化细节，常见配置和优化手段

Webpack

缺点：

1. 开发环境不好，打包构建后启动
2. 配置复杂

## 微内核设计？Vite7 插件化设计思想怎么完成个性化打包构建需求

## 说说你对 Vite 的理解，什么是 bundleless？

前置知识：**模块化**

1. 正是 esm 的浏览器支持，才有了 bundleless 方案。

bundleless 的理解：  
提倡少打包、不打包
主入口处引用其他的文件，其他的文件又引用另外的文件，此时，bundle 不需要将其他文件打包，而是直接通过请求来获取其他引用文件。

编码资源（源文件）有哪些：

- js（ts、tsx、jsx、vue）
- css
- png
- svg
  输出到一个指定目录

比如 webpack 需要模块化规范支持，构建模块依赖图。

2. 产物构建机制  
   产物构建从来不是依赖于打包工具，而是依赖于**编译工具**。

- 开发环境，应用就是代码资源：ts、tsx、jsx、vue、css、png、svg

js 不需要打包，其他资源需要打包（ts、tsx、jsx 用 esbuild；css 用 postcss）  
需要针对需要打包的资源做转换的处理（插件化机制：vite-plugin-xxx）

### 总结

esm 原生支持的 esmodule 让浏览器能直接解析 js 模块，不需要打包工具打包。

## Vite 构建过程了解吗，说说其实现原理？

## 如果让你设计一个打包工具，你怎么设计？

## 站在前端 Leader 的角度，从原理层面分析常规与新兴构建方案一同，详述 rust 重构工具链的优势

### 技术原理

常规的：

- js、webpack、tsc，工具间 ast 无法共享
- 生态成熟，兼容性低
- 性能较差

需要突破的是语言

新兴的：

基于 rust 的工具：

- swc
- turbo
- oxc

拥抱新兴构建工具的目的：

- 性能，单线程瓶颈，大型项目构建
- 统一化，oxc 编译、lint、format
- 内存安全稳定性
- 社区繁荣
