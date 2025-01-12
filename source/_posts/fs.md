---
title: fs
categories:
  - nodejs
description: fs
date: 2025-01-01 20:45:43
tags: nodejs
---

[官方文档](https://nodejs.org/api/fs.html)
[中文](https://nodejs.cn/api/fs.html)

### 创建目录

fs.mkdir(path[, options], callback)

### 创建文件

fs 模块提供了几种创建文件的方法。最常用的是 writeFile：  
不同的创建文件方法：  

writeFile - 创建新文件或覆盖已有文件：
```js
// 创建文件并写入内容
await fs.writeFile(filePath, 'Hello World', 'utf8');
```

appendFile - 追加内容或创建新文件：
```js
// 如果文件不存在则创建，存在则追加内容
await fs.appendFile(filePath, 'New Content', 'utf8');
```

open - 低级文件操作：
```js
// 以写入模式打开文件，如果不存在则创建
const fileHandle = await fs.open(filePath, 'w');
await fileHandle.close();
```

推荐使用 writeFile，因为：  
- 接口简单直观
- 自动处理文件不存在的情况
- 支持编码设置
- 原子操作，更安全

### 删除文件和目录

删除文件 - unlink：  
```js
// fs.unlink(path, callback)
// 删除单个文件
ipcMain.handle('deleteFile', async (_event, filePath: string) => {
  await fs.unlink(filePath);
});
```

删除目录 - rmdir：  
```js
// 删除空目录
ipcMain.handle('deleteEmptyDir', async (_event, dirPath: string) => {
  await fs.rmdir(dirPath);
});
```

递归删除 - rm：  
```js
// 递归删除目录及其内容
ipcMain.handle('deleteDir', async (_event, dirPath: string) => {
  await fs.rm(dirPath, { recursive: true, force: true });
});
```

选项说明：  
- recursive: true: 递归删除子目录和文件
- force: true: 忽略不存在的文件和权限错误

建议实现：  
```js
// 统一的删除接口
ipcMain.handle('delete', async (_event, path: string, isDirectory: boolean) => {
  if (isDirectory) {
    await fs.rm(path, { recursive: true, force: true });
  } else {
    await fs.unlink(path);
  }
});
```

### 复制文件或文件夹

cp  
```js
// 复制文件或文件夹
ipcMain.handle('copy', async (_event, sourcePath: string, targetPath: string, isDirectory: boolean) => {
  try {
    if (isDirectory) {
      // 复制文件夹及其内容
      await fs.cp(sourcePath, targetPath, { 
        recursive: true,  // 递归复制子目录
        force: false,     // 如果目标存在则不覆盖
        preserveTimestamps: true  // 保留时间戳
      });
    } else {
      // 复制单个文件
      await fs.copyFile(sourcePath, targetPath);
    }
    return true;
  } catch (error) {
    console.error('Copy error:', error);
    throw error;
  }
});
```

选项说明：  
- recursive: true: 递归复制子目录和文件
- force: false: 目标存在时不覆盖
- preserveTimestamps: 保留原始时间戳


```js
// 仅复制文件
await fs.copyFile(sourcePath, targetPath);
```

```js
// 复制文件夹（旧方法）
async function copyDir(src: string, dest: string) {
  await fs.mkdir(dest, { recursive: true });
  const entries = await fs.readdir(src, { withFileTypes: true });

  for (const entry of entries) {
    const srcPath = path.join(src, entry.name);
    const destPath = path.join(dest, entry.name);

    if (entry.isDirectory()) {
      await copyDir(srcPath, destPath);
    } else {
      await fs.copyFile(srcPath, destPath);
    }
  }
}
```

```js
```

### 读取文件

fs.readFile(path[, options], callback)

### 写入文件

fs.writeFile(file, data[, options], callback)


### 读取目录

fs.readdir(path[, options], callback)

### 重命名文件或目录

```js
fs.rename(oldPath, newPath, callback)

// 重命名文件或文件夹
ipcMain.handle('rename', async (_event, oldPath: string, newPath: string) => {
  try {
    // 检查新路径是否已存在
    const exists = await fs.access(newPath)
      .then(() => true)
      .catch(() => false);

    if (exists) {
      // 如果目标已存在，弹出确认对话框
      const { response } = await dialog.showMessageBox({
        type: 'question',
        buttons: ['覆盖', '取消'],
        defaultId: 1,
        title: '确认覆盖',
        message: '目标已存在，是否覆盖？',
        detail: `目标路径: ${newPath}`
      });

      if (response === 1) {
        return false;
      }
    }

    // 执行重命名
    await fs.rename(oldPath, newPath);
    return true;
  } catch (error) {
    console.error('Rename error:', error);
    throw error;
  }
});
```

### 获取文件状态

fs.stat(path, callback)

fs.access

// fs.access - 仅检查文件/目录的访问权限
await fs.access(path)  // 如果文件存在且可访问则成功，否则抛出错误

// fs.stat - 获取文件/目录的详细信息
const stats = await fs.stat(path)
console.log({
  isFile: stats.isFile(),          // 是否是文件
  isDirectory: stats.isDirectory(), // 是否是目录
  size: stats.size,                // 文件大小
  mtime: stats.mtime,             // 最后修改时间
  // ... 其他信息
})