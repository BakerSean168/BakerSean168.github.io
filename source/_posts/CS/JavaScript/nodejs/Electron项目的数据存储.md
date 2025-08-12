---
title: Electron项目的数据存储
categories:
  - nodejs
description: Electron项目的数据存储
date: 2025-05-17 15:22:32
tags:
---

# 可以使用的数据存储方式

Electron 能够直接使用 nodejs API 对系统文件进行读写  
当使用 Electron + Vue3 时，存储文件API 有 Node.js + Web APIs  

## nodejs 本地存储

```ts
class StorageService {
  private store: Store;

  constructor() {
    this.store = new Store({
      name: 'user-data', // 文件名
      encryptionKey: 'your-key', // 可选加密
      defaults: {
        accounts: [],
        settings: {}
      }
    });
  }

  // 保存用户数据
  async saveUser(userData: any) {
    return this.store.set(`users.${userData.id}`, userData);
  }

  // 读取用户数据
  async getUser(userId: string) {
    return this.store.get(`users.${userId}`);
  }
}
```

## Web APIs

- localstorage 大小一般为 5-10 MB

```ts
sessionStorage.setItem('formData', JSON.stringify({
  lastTab: 'profile',
  scrollPosition: 200
}));

localStorage.setItem('theme', 'dark');
localStorage.setItem('language', 'zh-CN');

const autoSave = () => {
  localStorage.setItem('draft', editor.getValue());
};
setInterval(autoSave, 1000);
```

# 选择策略

用户数据、配置信息、大文件、敏感信息等使用 Electron Storage  
临时数据、会话数据、UI状态等 使用 WebStorage  