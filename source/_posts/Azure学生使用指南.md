---
title: Azure学生使用指南
categories:
  - null
description: Azure学生使用指南
date: 2025-07-12 18:17:05
tags:
---

Azure 是 Microsoft 的云平台，在学生认证后，可以使用 Azure 的免费服务（100$/年）。  

下面主要记录一下 Azure 中服务器创建的注意事项。  

免费资源：
- 1台 Windows 和 1台 Linux 虚拟机 
- 2v1g 或 1v1g 配置
- 64G 磁盘

已经不支持基础 SKU（动态IP）了，所以只能使用标准 SKU 了（变免费了吧）。  
直接通过 Education 界面显示的可使用资源中的创建虚拟机入口导致了部署失败（可提供 ip 地址为 0），可能就是这个原因。所以需要自己通过标准入口创建虚拟机。  

