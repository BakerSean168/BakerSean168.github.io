---
title: 给Windows右键菜单添加创建markdown文件选项
categories:
  - null
description: 给Windows右键菜单添加创建markdown文件选项
date: 2025-07-30 14:34:02
tags:
---

[参考](https://blog.csdn.net/qq_43564374/article/details/109471694)



选择使用 Vscode 作为 md 编辑器  

在注册表的`计算机\HKEY_CLASSES_ROOT\Applications\`地址中找到编辑器的注册名称  

```reg
Windows Registry Editor Version 5.00
[HKEY_CLASSES_ROOT\.md]
@="Code.exe"
[HKEY_CLASSES_ROOT\.md\ShellNew]
"NullFile"=""
[HKEY_CLASSES_ROOT\Code.exe]
@="Markdown文档"
```

- @=“Code.exe” 指定.md文件的运行程序
- @=“Markdown文档” 指定右键新建的.md文件的默认名字, 也指定右键菜单新建相应选项名为 MarkDown

推荐不适用中文最为默认名字，因为中文会乱码（要使用中文的话注意将编辑器改为 ANSI 或 UTF-16 LE 编码，而非使用默认的 UTF-8 编码）  