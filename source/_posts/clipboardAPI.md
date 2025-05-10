---
title: clipboardAPI
categories:
  - 前端
description: clipboardAPI
date: 2025-05-10 11:07:25
tags: js
---
# clipboard API

## 1.ClipboardEvent 接口

```ts
interface ClipboardEvent extends Event {
  readonly clipboardData: DataTransfer | null;
}
```

## 2.DataTransfer 接口  

```ts
interface DataTransfer {
  // 获取剪贴板中的数据
  getData(format: string): string;
  
  // 设置数据到剪贴板
  setData(format: string, data: string): void;
  
  // 可用的数据格式列表
  readonly types: ReadonlyArray<string>;
  
  // 文件列表
  readonly files: FileList;
  
  // 剪贴板项目列表
  readonly items: DataTransferItemList;
}
```

## 3.常见用法示例

基本文本操作
```ts
const handlePaste = (e: ClipboardEvent) => {
  const clipboardData = e.clipboardData;
  if (!clipboardData) return;

  // 获取纯文本
  const text = clipboardData.getData('text/plain');
  
  // 获取 HTML
  const html = clipboardData.getData('text/html');
  
  // 获取 URL
  const url = clipboardData.getData('text/uri-list');
}
```

处理图片
```ts
const handleImagePaste = (e: ClipboardEvent) => {
  const clipboardData = e.clipboardData;
  if (!clipboardData) return;

  for (const item of clipboardData.items) {
    if (item.type.startsWith('image/')) {
      const file = item.getAsFile();
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          const imageDataUrl = e.target?.result as string;
          console.log('Image data:', imageDataUrl);
        };
        reader.readAsDataURL(file);
      }
    }
  }
}
```

获取所有可用格式
```ts
const logClipboardFormats = (e: ClipboardEvent) => {
  const clipboardData = e.clipboardData;
  if (!clipboardData) return;

  console.group('Clipboard Content Types');
  clipboardData.types.forEach(type => {
    console.log(`${type}:`, clipboardData.getData(type));
  });
  console.groupEnd();
}
```

## 4.现代 Clipboard API

```ts
// 异步 Clipboard API
const modernClipboardOps = {
  // 写入文本
  writeText: async (text: string) => {
    await navigator.clipboard.writeText(text);
  },
  
  // 读取文本
  readText: async () => {
    return await navigator.clipboard.readText();
  },
  
  // 读取所有内容（包括图片等）
  read: async () => {
    return await navigator.clipboard.read();
  }
};
```

## 5.事件类型

```ts
// 剪贴板事件监听
element.addEventListener('copy', (e: ClipboardEvent) => {
  // 处理复制
});

element.addEventListener('cut', (e: ClipboardEvent) => {
  // 处理剪切
});

element.addEventListener('paste', (e: ClipboardEvent) => {
  // 处理粘贴
});
```

## 6.安全注意事项

```ts
const secureClipboardAccess = async () => {
  try {
    // 检查权限
    const permission = await navigator.permissions.query({
      name: 'clipboard-read' as PermissionName
    });

    if (permission.state === 'granted') {
      // 可以访问剪贴板
      const text = await navigator.clipboard.readText();
      return text;
    } else {
      throw new Error('No clipboard permission');
    }
  } catch (error) {
    console.error('Clipboard access error:', error);
    return null;
  }
};
```

