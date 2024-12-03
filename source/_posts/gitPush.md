---
title: gitPush
date: 2024-12-03 19:58:09
tags:
categories: 
    - tools_document
description: git 推送代码到远程仓库时遇到三种情况
---

# 普通

```
git add .
git commit -m "message"
git remote add [name] [url] # 添加远程仓库
git push [name] [url]
```

# 远程仓库中已经有不需要的文件

使用 `git push --force [name] [url]` 强制推送，覆盖掉原有文件  

# 远程仓库中有需要保留的文件

```
git pull origin main
# 解决冲突并添加解决后的文件
git add <file>
git commit -m "Merge remote changes"
git push origin main
```
