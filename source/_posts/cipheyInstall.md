---
title: ciphey 在 Windows 下的安装
date: 2024-11-28 09:33:17
tags:
categories: 
    - tools_document
description: ciphey 在 Windows 下的安装
---

# 环境准备

⚠️ Ciphey doesn't work on Windows with Python 3.9+, use Python 3.7 or Python 3.8. Other platforms are fine with Python 3.9, Python 3.10 doesn't work.  
⚠️ Windows Python defaults to install 32-bit. Ciphey only supports 64-bit. Make sure you're using 64-bit Python.  

# 安装

20241128 执行该命令安装成功  
`pip install ciphey  -i http://pypi.mirrors.ustc.edu.cn/simple/ --trusted-host pypi.mirrors.ustc.edu.cn`  
-i 指定安装源，避免 ssl 错误  
```powershell
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1131)'))': /simple/ciphey/
```
--trusted-host，避免不信任错误  
```
WARNING: The repository located at pypi.douban.com is not a trusted or secure host and is being ignored. If this repository is available via HTTPS we recommend you use HTTPS instead, otherwise you may silence this warning and allow it anyway with '--trusted-host pypi.douban.com'.
```

# 其他疑问

## 使用 python3 命令没反应

```
(python38) PS D:\Environment\Python> python3 -m pip install -U ciphey  -i http://pypi.aliyun.com/simple --trusted-host pypi.aliyun.com
```

## 使用 https 协议时，指定软件源仍然报 ssl 错误

```powershell
 pip install ciphey  -i https://pypi.mirrors.ustc.edu.cn/simple/
Looking in indexes: https://pypi.mirrors.ustc.edu.cn/simple/
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1131)'))': /simple/ciphey/
WARNING: Retrying (Retry(total=3, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1131)'))': /simple/ciphey/
WARNING: Retrying (Retry(total=2, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1131)'))': /simple/ciphey/
WARNING: Retrying (Retry(total=1, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1131)'))': /simple/ciphey/
WARNING: Retrying (Retry(total=0, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1131)'))': /simple/ciphey/
Could not fetch URL https://pypi.mirrors.ustc.edu.cn/simple/ciphey/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.mirrors.ustc.edu.cn', port=443): Max retries exceeded with url: /simple/ciphey/ (Caused by SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:1131)'))) - skipping
```