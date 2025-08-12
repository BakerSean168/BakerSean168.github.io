---
title: tsc文件大小写报错
categories:
  - null
description: tsc文件大小写报错
date: 2025-08-11 21:24:16
tags:
---

### ts 编译器缓存，导致文件大小写报错

`cd packages/domain-server && Remove-Item -Recurse -Force node_modules -ErrorAction SilentlyContinue; Remove-Item -Recurse -Force dist -ErrorAction SilentlyContinue`
用上面的命令清除缓存再 install
