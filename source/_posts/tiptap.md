---
title: tiptap
categories:
  - null
description: tiptap
date: 2025-09-17 11:50:27
tags:
---

## 简介
Tiptap 是一个基于 ProseMirror 的开源富文本编辑器框架，用于构建自定义的富文本编辑器。它提供了灵活的 API，支持多种前端框架（如 Vue、React、Svelte），并通过扩展系统实现功能定制。Tiptap 专注于可扩展性和性能，适用于博客、CMS、协作工具等场景。

## 核心特性
- **基于 ProseMirror**：强大的底层引擎，支持复杂编辑操作。
- **扩展系统**：模块化扩展，如文本格式、链接、图片等。
- **框架支持**：原生支持 Vue、React 等。
- **实时协作**：可集成 Yjs 等协作库。
- **可定制**：高度可配置的 UI 和行为。

## 安装
使用 npm 或 yarn 安装：
```bash
npm install @tiptap/vue-3 @tiptap/pm @tiptap/starter-kit
# 或
yarn add @tiptap/vue-3 @tiptap/pm @tiptap/starter-kit
```

## 基本用法
### Vue 示例
```vue
<template>
  <editor-content :editor="editor" />
</template>

<script setup>
import { useEditor, EditorContent } from '@tiptap/vue-3'
import StarterKit from '@tiptap/starter-kit'

const editor = useEditor({
  content: '<p>Hello World!</p>',
  extensions: [
    StarterKit,
  ],
})
</script>
```

### React 示例
```jsx
import { useEditor, EditorContent } from '@tiptap/react'
import StarterKit from '@tiptap/starter-kit'

const editor = useEditor({
  content: '<p>Hello World!</p>',
  extensions: [
    StarterKit,
  ],
})

return <EditorContent editor={editor} />
```

## 常用扩展
- **StarterKit**：基础扩展包，包括段落、标题、粗体等。
- **TextStyle**：文本样式，如颜色、字体大小。
- **Link**：链接插入和编辑。
- **Image**：图片上传和显示。
- **Table**：表格支持。
- **Collaboration**：实时协作。

## 配置选项
- **content**：初始内容（HTML 或 JSON）。
- **extensions**：启用扩展列表。
- **editable**：是否可编辑。
- **onUpdate**：内容更新回调。

## 示例：添加扩展
```javascript
import { useEditor } from '@tiptap/vue-3'
import StarterKit from '@tiptap/starter-kit'
import Link from '@tiptap/extension-link'

const editor = useEditor({
  extensions: [
    StarterKit,
    Link.configure({
      openOnClick: false,
    }),
  ],
})
```

## 更多资源
- [官方文档](https://tiptap.dev/)
- [GitHub 仓库](https://github.com/ueberdosis/tiptap)
- [社区扩展](https://tiptap.dev/extensions)