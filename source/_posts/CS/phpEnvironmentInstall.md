---
title: Windows 下 php 环境配置
date: 2024-11-23 10:27:36
tags:
categories: 
    - php
description: Windows 下 php 环境配置
---

# 安装

## 1

[Windows php 安装地址](https://windows.php.net/download/)

与 Apache 一起使用，选择 TS（Thread Safe/线程安全） 版本。 这里选择 64 位 zip 形式。

解压后，将 php 文件夹添加到系统变量  

打开 cmd，使用 `php -v` 测试。返回版本信息则成功  

# 配置

[官方文档推荐配置](https://www.php.net/manual/zh/install.windows.recommended.php)

打开 OpCache。 此扩展默认已经包含到 PHP Windows 版本中。 它会自动编译和优化 PHP 脚本，并将它们缓存在内存中， 这样就不会在每次加载页面时动态编译它们。

```
opcache.enable=On
opcache.enable_cli=On
```


# 遇到的报错

## 1

### 错误形式

```cmd
PHP Warning:  'C:\WINDOWS\SYSTEM32\VCRUNTIME140.dll' 14.38 is not compatible with this PHP build linked with 14.41 in Unknown on line 0
```

### 解决方式

下载最新的 Microsoft Visual C ++ Redistributable 工具  

[最新受支持 x64 版本的永久链接](https://aka.ms/vs/17/release/vc_redist.x64.exe)