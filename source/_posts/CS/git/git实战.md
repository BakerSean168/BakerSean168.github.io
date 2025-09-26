---
title: git实战
date: 2024-12-03 19:58:09
tags:
    - git
categories: 
description: git 推送代码到远程仓库时遇到三种情况
---

## 开发新分支

流程：  
1. 在当前版本中创建新分支，命名为`feature/xxx`，

tip：  
- 如果有旧的该功能分支，最好也在新版本中创建新分支，再将旧分支合并到新分支

## 推送代码

应该先 pull 再 push

### 普通

```
git add .
git commit -m "message"
git remote add [name] [url] # 添加远程仓库
git push [name] [url]
```

### 远程仓库中已经有不需要的文件

使用 `git push --force [name] [url]` 强制推送，覆盖掉原有文件  

### 远程仓库中有需要保留的文件

```
git pull origin main
# 解决冲突并添加解决后的文件
git add <file>
git commit -m "Merge remote changes"
git push origin main
```

## 从仓库下载内容依赖失败

### 1. 使用 http 而非 ssh

```bash
# 修改 Git 全局配置，强制使用 HTTPS 代替 SSH
git config --global url."https://github.com/".insteadOf "git@github.com:"
git config --global url."https://".insteadOf "git://"
```