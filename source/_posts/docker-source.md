---
title: Windows 下 php 环境安装
date: 2024-11-16 13:06:21
tags:
categories: 
    - php
description: Windows 下 php 环境安装
---

# 1

[Windows php 安装地址](https://windows.php.net/download/)

与 Apache 一起使用，选择 TS（Thread Safe/线程安全） 版本。  这里选择 64 位 zip 形式。

# 2

解压后，将 php 文件夹添加到系统变量  

打开 cmd，使用 `php -v` 测试。  
返回版本信息则成功  

# 遇到的报错

## 1

### 错误形式

```cmd
PHP Warning:  'C:\WINDOWS\SYSTEM32\VCRUNTIME140.dll' 14.38 is not compatible with this PHP build linked with 14.41 in Unknown on line 0
```

### 解决方式

下载最新的 Microsoft Visual C ++ Redistributable 工具  

[最新受支持 x64 版本的永久链接](https://aka.ms/vs/17/release/vc_redist.x64.exe)