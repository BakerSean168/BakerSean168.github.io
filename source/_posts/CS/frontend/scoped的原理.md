---
title: scoped的原理
categories:
  - Vue
description: scoped的原理
date: 2025-06-04 09:58:21
tags:
---

# Scoped 样式的作用域原理

Vue 中的 scoped 属性是通过 PostCSS 插件在编译阶段对 CSS 进行特殊处理，实现组件级样式隔离的核心机制。其工作原理可以概括为以下几个关键步骤：  
1. 唯一属性标记生成  
  Vue Loader 会基于组件文件路径生成一个唯一的哈希值，如 `data-v-f3242`  
2. DOM 元素标记
  在编译时，这个唯一属性会被自动添加到组件模板的所有 HTML 元素上  
3. CSS 选择器转换
  为组件内所有 CSS 选择器追加 [data-v-xxxxxx] 属性选择器

**核心思想就是使类名（选择器）唯一**  
所以有些时候要用到深度选择器 `:deep()` 来控制样式，比如第三方组件内置样式修改  

# 其他方案

- CSS Modules:  
  编译时生成唯一类名来实现  
- BEM 命名约定

