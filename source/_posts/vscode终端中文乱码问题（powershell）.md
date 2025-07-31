---
title: vscode终端中文乱码问题（powershell）
categories:
  - null
description: 在调试信息中使用了中文，结果终端中打印出来的中文都是乱码的，试了网上的改变终端编码的方法，结果没有用，最后找到了另外的解决方法。
date: 2025-07-10 11:42:00
tags:
---

## powershell 命令

```powershell
# 获取当前 powershell 的编码
chcp

# 获取当前 powershell 的输出编码
[Console]::OutputEncoding

# 设置 powershell 的输出编码为 UTF-8
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8

# 创建/打开 $PROFILE 文件
notepad $PROFILE
```

notepad $PROFILE

## 方法

先直接上方法：  

### 1. 临时测试

1. 在 powershell 终端中输入「获取当前输出编码」 的命令，查看输出编码，可以看到输出编码为「Chinese Simplified (GB2312)」，而非 UTF-8。
```powershell
EncodingName      : Chinese Simplified (GB2312)
WebName           : gb2312
HeaderName        : gb2312
BodyName          : gb2312
Preamble          : 
WindowsCodePage   : 
IsBrowserDisplay  : 
IsBrowserSave     : 
IsMailNewsDisplay : 
IsMailNewsSave    : 
IsSingleByte      : False
EncoderFallback   : System.Text.InternalEncoderBestFitFallback
DecoderFallback   : System.Text.InternalDecoderBestFitFallback
IsReadOnly        : False
CodePage          : 936
```

2. 输入「设置 powershell 的输出编码为 UTF-8」 的命令，查看输出编码，可以看到输出编码已经变为 UTF-8。
3. 测试打印的中文是否正常。

### 2. 永久解决

如果上面的此时通过了，可以用下面的方法修改配置文件，永久解决。

1. 创建/打开 $PROFILE 文件，在文件末尾添加如下内容：`[Console]::OutputEncoding = [System.Text.Encoding]::UTF8`
2. 保存、关闭文件。

## 总结

之所以 chcp 直接设置页面编码没用，应该是因为 PowerShell 不会自动根据 chcp 的值更改 [Console]::OutputEncoding，所以需要手动设置输出编码为 UTF-8。  