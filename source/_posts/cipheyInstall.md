---
title: ciphey 在 Windows 下的安装和使用
date: 2024-11-28 09:33:17
tags:
categories: 
    - tools_document
description: ciphey 在 Windows 下的安装和使用
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

# 使用

There are 3 ways to run Ciphey.

1. File Input ciphey -f encrypted.txt
2. Unqualified input ciphey -- "Encrypted input"
3. Normal way ciphey -t "Encrypted input"


## 报错
`UnicodeDecodeError: 'gbk' codec can't decode byte 0xbf in position 695: illegal multibyte sequence`  

解决方式：  

### 1

修改该文件  
`..\python安装的路径\Lib\site-packages\pywhat\regex_identifier.py`  

```python
import json
import os
import re


class RegexIdentifier:
    def __init__(self):
        path = "Data/regex.json"
        fullpath = os.path.join(os.path.dirname(os.path.abspath(__file__)), path)
        # 改下面这句里的 r 的后面加上 ", encoding = 'UTF-8'" , 显示指定 UTF-8 编码方式打开
        with open(fullpath, "r") as myfile:
            self.regexes = json.load(myfile)
```

### 2

自己添加检测文件编码类型的功能  
```python
import chardet

with open('filename.txt', 'rb') as f:
    raw_data = f.read()
    result = chardet.detect(raw_data)
    encoding = result['encoding']

with open('filename.txt', 'r', encoding=encoding) as f:
    content = f.read()
```