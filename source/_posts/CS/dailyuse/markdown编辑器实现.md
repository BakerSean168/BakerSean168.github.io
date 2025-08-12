---
title: markdown编辑器实现
categories:
  - 前端
description: markdown编辑器实现
date: 2024-12-14 11:11:03
tags:
---

# 编辑器

## 布局

类似 vscode 布局

```
一、VS Code 布局结构解析

区域	功能描述
活动栏	左侧垂直图标栏（文件、搜索、Git、调试等入口），点击切换侧边栏内容
侧边栏	动态内容区（资源管理器、搜索、插件管理等），可折叠
编辑器区域	多标签页编辑器 + 主内容区
<!-- 面板区域	底部或右侧区域（终端、输出、问题面板等），支持拖拽调整高度/宽度 -->
状态栏	底部状态信息（Git 分支、编码格式、光标位置等）

二、技术选型

功能	推荐工具/库
布局框架	CSS Grid + Flexbox（原生实现）或 Splitpanes（拖拽分割）
状态管理	Pinia（Vue 3 官方推荐）
图标	Material Design Icons 或 Iconify
多标签页	自定义实现或 Vue Tabs
主题系统	CSS 变量 + 动态类名

三、项目结构与组件设计

src/
├── layouts/
│   └── VSCodeLayout.vue      # 整体布局容器
├── components/
│   ├── ActivityBar.vue       # 左侧活动栏（图标按钮）
│   ├── Sidebar.vue           # 侧边栏（动态内容）
│   ├── EditorTabs.vue        # 多标签页
│   ├── EditorArea.vue        # 编辑器主区域
│   ├── StatusBar.vue         # 底部状态栏
│   └── ResizeHandle.vue      # 拖拽分割条
├── stores/
│   └── layoutStore.ts        # 布局状态管理（侧边栏宽度、面板高度等）
└── styles/
    ├── themes/               # 主题变量
    └── layout.css            # 布局样式
```

每次打开时窗口大小不确定，调整区域使用 get
```ts
interface EditorLayoutState {
    activityBarWidth: number; //活动栏 固定
    sidebarWidth: number; //侧边栏 调整
    minSidebarWidth: number; //最小侧边栏 固定
    resizeHandleWidth: number; // resize条 固定
    minEditorWidth: number; //最小编辑器 固定
    editorTabWidth: number; //编辑器标签宽度 固定

    editorGroupsWidth: number; //编辑组区域 调整
}
```

每次打开编辑器时，初始化每个区域（editor-group）的长度  
然后监听窗口变化来修改每个区域的大小  

### resize

```ts
import { debounce } from 'lodash-es'

// Debounced resize handler
const handleResize = debounce(() => {
    editorLayoutStore.updateTotalWidth(window.innerWidth)
}, 200) // 200ms delay
```

web 自带的 API 用于监听窗口
window.addEventListener('resize', handler)

## markdown 编辑器

### 技术选择

- markdown-it  
    Markdown 解析  
- [Monaco](https://wf0.github.io/)  
    "monaco-editor": "^0.52.2" -monaco 编辑器核心  
    "monaco-editor-vue3": "^0.1.10" -组件化 monaco 编辑器，方便在 vue 中使用  
    "vite-plugin-monaco-editor": "^1.1.0" -方便 vite 配置 Monaco  
- DOMPurify  
    安全渲染，防止 XSS 攻击

### monaco-editor-vue3 配置

// const editorOptions = {
//   minimap: { enabled: true },
//   wordWrap: 'on',
//   lineNumbers: 'on',
//   renderWhitespace: 'boundary',
//   scrollBeyondLastLine: false,
//   automaticLayout: true,
//   fontSize: 14,
//   padding: { top: 16 }
// }

### Monaco Editor 实例的获取和使用

在组件中使用 @mounted 获取

```ts
// Monaco Editor 实例的常用方法和属性示例
const handleEditorDidMount = (instance: any) => {
  editor.value = instance;  // 保存编辑器实例
  
  // 常用方法示例
  instance.getValue();              // 获取编辑器内容
  instance.setValue('new content'); // 设置编辑器内容
  instance.getPosition();          // 获取当前光标位置
  instance.setPosition({           // 设置光标位置
    lineNumber: 1,
    column: 1
  });
  
  // 事件监听
  instance.onDidChangeModelContent(() => {
    // 内容变化时触发
  });
  
  instance.onDidChangeCursorPosition(() => {
    // 光标位置变化时触发
  });
  
  // 编辑操作
  instance.executeEdits('source', [{
    range: new monaco.Range(1, 1, 1, 1),
    text: 'inserted text'
  }]);
  
  // 获取选中内容
  const selection = instance.getSelection();
  const selectedText = instance.getModel()?.getValueInRange(selection);
}
```

## 粘贴图片 功能

- 直接将图片转化为 base64 嵌入代码中  
- 将图片保存到相应目录，通过链接显示  

### 1.编辑器监听 paste 事件，当 paste 为图片时进行相应处理  
  `monacoEditor.value.onDidPaste` Monaco Editor 貌似有自带监听 paste 的方法  

#### 监听粘贴事件的方法

*使用 markRaw，告诉 Vue 不要将编辑器实例转换为响应式对象，否则执行 executeEdits 会卡住*

```ts
const handleEditorDidMount = (instance: any) => {
  editor.value = markRaw(instance)
  
  // 方法1: 使用 onDidPaste
  // Monaco Editor 的原生事件
  // 在粘贴完成后触发
  // 提供粘贴的文本内容
  // onDidPaste 返回的 e 对象好像没有粘贴的数据
  editor.value.onDidPaste((e: any) => {
    console.log('Paste event:', e)
    console.log('Pasted text:', e.text)
  })

  // 方法2: 使用 onKeyDown 监听粘贴快捷键
  // 可以捕获粘贴快捷键
  // 在粘贴发生前触发
  // 可以阻止默认行为
  editor.value.onKeyDown((e: any) => {
    if ((e.ctrlKey || e.metaKey) && e.keyCode === 86) { // 86 是 'V' 键的keyCode
      console.log('Paste shortcut detected')
    }
  })

  // 方法3: 添加命令监听
  // 添加自定义命令
  // 可以绑定特定快捷键
  // 更灵活的控制
  editor.value.addCommand(monaco.KeyMod.CtrlCmd | monaco.KeyCode.KeyV, () => {
    console.log('Paste command triggered')
  })

  // 方法4: 使用事件监听器
  // DOM 原生事件
  // 可以访问完整的剪贴板数据
  // 可以处理多种格式（文本、HTML、图片等）
  const editorDomElement = editor.value.getDomNode()
  editorDomElement.addEventListener('paste', (e: ClipboardEvent) => {
    e.preventDefault() // 阻止默认粘贴行为
    
    const clipboardData = e.clipboardData
    if (!clipboardData) return

    // 打印所有可用的格式
    console.log('Available formats:', clipboardData.types)

    // 获取文本内容
    if (clipboardData.types.includes('text/plain')) {
      const text = clipboardData.getData('text/plain')
      console.log('Plain text:', text)
    }

    // 获取HTML内容
    if (clipboardData.types.includes('text/html')) {
      const html = clipboardData.getData('text/html')
      console.log('HTML:', html)
    }

    // 处理图片
    const items = clipboardData.items
    for (const item of items) {
      if (item.type.startsWith('image/')) {
        const file = item.getAsFile()
        if (file) {
          console.log('Image:', {
            type: file.type,
            size: file.size,
            lastModified: new Date(file.lastModified)
          })
        }
      }
    }
  })
}
```



## Other

### 如何将选中的仓库的路径传给文件管理器 

在 RepositoryStore 中添加获取方法  
利用 URL 与 title 有关来获取  
```ts
currentRepositoryPath: (state) => {
            const currentRepo = state.repositories.find(
                repo => repo.title === window.location.hash.split('/').pop()
            );
            return currentRepo?.path || '';
        }
```
