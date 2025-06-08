---
title: 配置ssh密钥来连接虚拟机（ubuntu），并使用vscode远程连接开发
categories:
  - null
description: 配置ssh密钥来连接虚拟机
date: 2025-06-07 14:55:05
tags:
---

# 流程

1. 下载 vscode 的 Remote Development 插件
2. 获取虚拟机的 ip 地址，并在本机主机看能不能 ping 通
3. 在本地主机生成 ssh 密钥对，并将公钥传到虚拟主机的 ~/.ssh 文件中
4. 配置本地主机的 ssh 配置文件

# 具体操作（部分）

## 生成 ssh 密钥

生成 ssh 密钥需要 ssh 相关服务，一般在下载 git 后会自带 openssh。

ssh-keygen

用于为“ssh”生成、管理和转换认证密钥，它支持RSA和DSA两种认证密钥。  

选项：  
```
-b：指定密钥长度；
-e：读取openssh的私钥或者公钥文件；
-C：添加注释；
-f：指定用来保存密钥的文件名；
-i：读取未加密的ssh-v2兼容的私钥/公钥文件，然后在标准输出设备上显示openssh兼容的私钥/公钥；
-l：显示公钥文件的指纹数据；
-N：提供一个新密语；
-P：提供（旧）密语；
-q：静默模式；
-t：指定要创建的密钥类型。
```

eg：  
`ssh-keygen -t rsa -b 4096 -f ubu2004` 

windows 下生成在用户目录下，建议剪贴到 .ssh 中统一保存  
linux 生成在用户目录的 .ssh 文件中  

## 配置虚拟机

1. 确保虚拟机有 ssh 服务

```bash
# 查看本机的 ssh 服务状态
# On the remote Linux machine
sudo systemctl status ssh
# or
sudo systemctl status sshd
```

如果没有服务，则安装：  
```bash
sudo apt update
sudo apt install openssh-server
```

## 配置本地主机的 ssh 配置文件

vscode 提供了图形化操作：  
点击 ssh 行的右侧 ⚙️ 选项，选择当前用户下的 .ssh\config 文件，进行配置  
配置项如下：  
```
Host Ubuntu20.04
    HostName 192.168.97.125
    User zhangsan
    port 22
    IdentityFile "C:\Users\zhangsan\.ssh\ubu2004"
```

此时左侧 ssh 下出现了配置的主机项，可以直接连接

# 高级

端口映射