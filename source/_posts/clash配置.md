---
title: clash配置
categories:
  - 网络
description: clash配置
date: 2025-05-03 20:47:14
tags:
---

信息来源：[Clash 知识库](https://clash.wiki/)

# 什么是 Clash

Clash 是一个跨平台的基于规则的代理工具, 在网络和应用层运行。

Clash 使用 YAML (YAML Ain't Markup Language) 作为配置文件格式。

## Clash 原理（两个关键部分）：

- Inbound 入站
  Inbound 入站是在本地端监听的部分, 它通过打开一个本地端口并监听传入的连接来工作. 当连接进来时, Clash 会查询配置文件中配置的规则, 并决定连接应该去哪个 Outbound 出站。
- Outbound 出站  
  Outbound 出站是连接到远程端的部分. 根据配置的不同, 它可以是一个特定的网络接口、一个代理服务器或一个策略组。

## 基于规则的路由

Clash 支持基于规则的路由, 这意味着您可以根据各种规则将数据包路由到不同的出站. 规则可以在配置文件的 rules 部分中定义。  
有许多可用的规则类型, 每种规则类型都有自己的语法. 规则的一般语法是:

```bash
# 类型,参数,策略(,no-resolve)
TYPE,ARGUMENT,POLICY(,no-resolve)
```

# 配置

## 配置文件

主配置文件名为 config.yaml. 默认情况下, Clash 会在 $HOME/.config/clash 目录读取配置文件.

-d 指定配置目录  
-f 指定配置文件

## 特殊语法

### IPv6 地址

用方括号（[]）包裹 IPv6 地址

### DNS 通配符域名匹配

任何包含这些字符的域名都应该用单引号 (') 包裹. 例如, '_.google.com'. 静态域名的优先级高于通配符域名 (foo.example.com > _.example.com > .example.com) .

使用星号 (\*) 来匹配单级通配符子域名  
使用点号 (.) 来匹配多级通配符子域名（不包括根域名）  
使用加号 (+) 来匹配多级通配符子域名（包括根域名）

# Rules 规则

[Clash.wiki.Rules](https://clash.wiki/configuration/rules.html)

## 策略

目前有四种策略类型,:

- DIRECT: 通过 interface-name 直接连接到目标 (不查找系统路由表)
- REJECT: 丢弃数据包
- Proxy: 将数据包路由到指定的代理服务器
- Proxy Group: 将数据包路由到指定的策略组

## 规则

| 规则类型       | 语法示例                                | 说明                   |
| -------------- | --------------------------------------- | ---------------------- |
| DOMAIN         | `DOMAIN,www.google.com,policy`          | 精确匹配域名           |
| DOMAIN-SUFFIX  | `DOMAIN-SUFFIX,youtube.com,policy`      | 匹配域名后缀           |
| DOMAIN-KEYWORD | `DOMAIN-KEYWORD,google,policy`          | 匹配域名关键字         |
| GEOIP          | `GEOIP,CN,policy`                       | 匹配 IP 地理位置       |
| IP-CIDR        | `IP-CIDR,127.0.0.0/8,DIRECT`            | 匹配 IPv4 地址段       |
| IP-CIDR6       | `IP-CIDR6,2620:0:2d0:200::7/32,policy`  | 匹配 IPv6 地址段       |
| SRC-IP-CIDR    | `SRC-IP-CIDR,192.168.1.201/32,DIRECT`   | 匹配源 IP 地址段       |
| SRC-PORT       | `SRC-PORT,80,policy`                    | 匹配源端口             |
| DST-PORT       | `DST-PORT,80,policy`                    | 匹配目标端口           |
| PROCESS-NAME   | `PROCESS-NAME,nc,DIRECT`                | 匹配进程名称           |
| PROCESS-PATH   | `PROCESS-PATH,/usr/local/bin/nc,DIRECT` | 匹配进程路径           |
| IPSET          | `IPSET,chnroute,policy`                 | 匹配 IP 集合(仅 Linux) |
| RULE-SET       | `RULE-SET,my-rule-provider,DIRECT`      | 匹配规则集(仅 Premium) |
| SCRIPT         | `SCRIPT,script-path,DIRECT`             | 执行脚本(仅 Premium)   |
| MATCH          | `MATCH,policy`                          | 全匹配(最后规则)       |

# Clash DNS

由于 Clash 的某些部分运行在第 3 层 (网络层) , 因此其数据包的域名是无法获取的, 也就无法进行基于规则的路由.  
Enter fake-ip: 它支持基于规则的路由, 最大程度地减少了 DNS 污染攻击的影响, 并且提高了网络性能, 有时甚至是显著的.  

## fake-ip

"fake IP" 的概念源自 RFC 3089:  
  一个 "fake IP" 地址被用于查询相应的 "FQDN" 信息的关键字.  
fake-ip 池的默认 CIDR 是 198.18.0.1/16 (一个保留的 IPv4 地址空间, 可以在 dns.fake-ip-range 中进行更改).  

当 DNS 请求被发送到 Clash DNS 时, Clash 内核会通过管理内部的域名和其 fake-ip 地址的映射, 从池中分配一个 空闲 的 fake-ip 地址.  
以使用浏览器访问 http://google.com 为例：  
1. 浏览器向 Clash DNS 请求 google.com 的 IP 地址  
2. Clash 检查内部映射并返回 198.18.1.5  
3. 浏览器向 198.18.1.5 的 80/tcp 端口发送 HTTP 请求  
4. 当收到 198.18.1.5 的入站数据包时, Clash 查询内部映射, 发现客户端实际上是在向 google.com 发送数据包  
5. 根据规则的不同:  
  Clash 可能仅将域名发送到 SOCKS5 或 shadowsocks 等出站代理, 并与代理服务器建立连接  
  或者 Clash 可能会基于 SCRIPT、GEOIP、IP-CIDR 规则或者使用 DIRECT 直连出口查询 google.com 的真实 IP 地址  