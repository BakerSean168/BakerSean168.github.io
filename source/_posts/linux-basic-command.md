---
title: linux basic command
date: 2024-11-13 16:48:16
tags:
categories: 
    - linux
description: linux 中的基础命令
---

# 任务管理工具

## 简单命令

| 命令 | 作用 |
| --- | --- |
| `&` | 加在一个命令的最后，可以把这个命令放在后台执行,如gftp& |
| `ctrl+z` | 可以将一个正在前台执行的命令放在后台，但是处于暂停状态，不可执行 |
| `ctrl+c` | 前台进程的中止 |
| `jobs` | 显示当前 shell 会话中所有的后台任务及其状态 |
| `bg` | 使一个在后台暂停的任务继续在后台运行 |
| `fg` | 将一个后台任务调到前台运行 |

# 文件管理工具

## ls

`ls [options] [files]`  
显示目录下的内容

常用参数：  
[-a] 显示所有文件，包括隐藏文件
[-l] 以长格式显示
[-R] 递归显示子目录内容

**长格式**
```bash
-rw-r--r-- 1 root root 337 Oct  4 11:31 /etc/hosts
```

每一项数据的意义
| 意义                | 命令中的样子               |
|---------------------|---------------------------|
| The file type       | -  eg: "-"表示文件，"d"表示文件夹 |
| The file permissions| rw-r--r--  eg：用户、组、其他|
| Number of hard links| 1                         |
| File owner          | root                      |
| File group          | root                      |
| File size           | 337                       |
| Date and Time       | Oct 4 11:31               |
| File name           | /etc/hosts                |

## less

`less [options] filename`
分页显示文件内容  

常用参数：
[-N] 显示行号  
[+F] 查看文件变化的内容，适合看日志文件

使用时命令
| 命令 | 作用                   |
|------|------------------------|
| 上,回车,e,j | 向前一行              |
| 下,y,k | 向后一行              |
| y    | 向后一行              |
| k    | 向后一行              |
| f    | 向前翻页              |
| g    | 向后翻页              |
| /    | 向前搜索              |
| ?    | 向后搜索              |
| n    | 重复之前的搜索        |
| N    | 重复之前的搜索，但反向|
| g    | 到第一行              |
| Ng   | 到第 N 行             |
| G    | 到最后一行            |
| P    | 到文件开始            |
| Np   | 到文件的 N%           |
| h    | 显示帮助              |
| q    | 退出                  |

**使用技巧**

ls /root | less 分页观看复杂的输出  

## head

`head [options] [files]`  
显示文件前十行（默认）内容，可同时显示多个文件  

参数：  
[-n]显示行数  
*tip:可以省略参数直接 -50 显示50行*  
[-c]显示字节数  

## tail

`tail [options] [files]`  
显示文件前十行（默认）内容，可同时显示多个文件  

参数：  
[-n]显示行数  
[-c]显示字节数  
[-f]监视文件更改  
Ctrl+C 中断监视  
[-F]重新创建文件时继续监视该文件  

## ln

**软连接和硬链接**
- 硬链接可以看作是文件的另一个名字，将多个名字链接向相同的 inode。同一文件系统或分区才能创建硬链接。一个文件可以有多个硬链接。  
- 软连接可以看作是windows中的快捷方式，是一种引用的文件类型。不同文件系统或分区也能创建软链接。  

`ln -s [options] fileToBeLinked linkedName`  
默认创建硬链接，[-s]用来创建软链接

[-f] 强制覆盖符号链接的目标路径

`unlink linkedName` 或 `remove linkedName` 删除链接

## chown

`chown [options] user[:group] file[s]`  
改变文件的所有者和组  

[-R] 递归更改子目录所有文件  
[-h] 如果包含软链接，使用它，更改符号链接本身的组所有权  

## chmod

`chmod [options] mode file[s]`  
`chmod [OPTIONS] [ugoa…][-+=]perms…[,…] FILE...`  
改变文件的权限  

**用户组参数属性说明**

| 参数 | 说明                               |
|------|------------------------------------|
| u    | The file owner.                    |
| g    | The users who are members of the group. |
| o    | All other users.                   |
| a    | All users, identical to ugo.       |

**权限参数说明**

| 参数 | 说明                                                                 |
|------|----------------------------------------------------------------------|
| -    | Removes the specified permissions.                                   |
| +    | Adds specified permissions.                                          |
| =    | Changes the current permissions to the specified permissions. If no permissions are specified after the = symbol, all permissions from the specified user class are removed. |

**数字参数**

当使用4位数字时，第一位数字具有以下含义：  

| 数字 | 含义                |
|------|---------------------|
| 4    | setuid              |
| 2    | setgid              |
| 1    | sticky              |
| 0    | no changes          |

其余三位：  

| 字母 | 含义                |
|------|---------------------|
| r    | read (4)            |
| w    | write (2)           |
| x    | execute (1)         |
| -    | no permissions (0)  |

## du

`du [options] [files]`  
查看磁盘使用量 diskUsage  

[-s] 全部加起来的使用量  
[-h] 人性化显示  
[--appart-size] 实际数据量  
[-c] 通配符查询  

# 文件系统管理工具

## df

`df [options] filesystem`  
显示文件系统的磁盘空间使用情况  

```bash
Filesystem     1K-blocks      Used Available Use% Mounted on
dev              8172848         0   8172848   0% /dev
run              8218640      1696   8216944   1% /run
/dev/nvme0n1p3 222284728 183057872  27865672  87% /
tmpfs            8218640    150256   8068384   2% /dev/shm
tmpfs            8218640         0   8218640   0% /sys/fs/cgroup
tmpfs            8218640        24   8218616   1% /tmp
/dev/nvme0n1p1    523248    107912    415336  21% /boot
/dev/sda1      480588496 172832632 283320260  38% /data
tmpfs            1643728        40   1643688   1% /run/user/1000
```

[-h] 人性化显示  
[-t] 显示类型/过滤指定类型  
[-i] 打印有关文件系统inode使用情况的信息

## mount

`mount -t [type] [device] [dir]`  
挂载文件系统到指定目录  

## fdisk

`fdisk [options] [files]`  
一个用于创建分区方案的命令行工具  

[-l] 列出分区  

## mkfs

`mkfs -t [fs type] [target device]`  
`mkfs.[fs type] [target device]`  
格式化选择的某个文件系统中的磁盘或分区 "make file system"  

## lsblk

`lsblk [options] [device(s)]`  
获取有关 Linux 系统上的驱动器和块设备的信息 "list block devices"  

[-a] 列出所有块设备，包括空设备和cdrom
[-f] 显示每个设备的文件系统信息  
[-m] 打印每个设备的权限  
[-o] 指定要在输出中显示的字段  

# 网络管理工具

## ip

`ip [ OPTIONS ] OBJECT { COMMAND | help }`  
配置网络接口的强大工具  

addr对象最常用的命令是：show、add和del  
对象有
- link (l) - Display and modify network interfaces.
- address (a) - Display and modify IP Addresses.
- route (r) - Display and alter the routing table.
- neigh (n) - Display and manipulate neighbor objects (ARP table).

`ip addr show`  

## dig

### 输出说明

dig是一个命令行工具，用于查询DNS信息和解决DNS相关问题。

示例：  
`dig linux.org`  
输出结果：  
![alt text](assert/linux-basic-command_dig.png)

 输出的第一行打印安装的dig版本和查询的域名。第二行显示全局选项（默认情况下，只有cmd）。[+nocmd]可以不显示，得是第一个参数。
```bash
; <<>> DiG 9.13.3 <<>> linux.org
;; global options: +cmd
```

下一节包括有关从请求的机构（DNS服务器）接收的应答的技术细节。头部显示操作码（由dig执行的操作）和操作的状态。在本例中，状态为NOERROR，这意味着请求的授权机构在没有任何问题的情况下处理了查询。 [+nocomments]不显示。 
```bash
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37159
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 2, ADDITIONAL: 5
```

“OPT”伪段仅在较新版本的dig实用程序中显示。[+noedns]不显示   
```bash
;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
```

在“QUESTION”部分中，`dig`显示了查询（问题）。默认情况下，dig请求A记录。[+nopquestion]  
```bash
;; QUESTION SECTION:
;linux.org.			IN	A
```

“答案”部分为我们提供了问题的答案。正如我们已经提到的，默认情况下，dig将请求A记录。在这里，我们可以看到域linux.org指向104.18.59.123IP地址。[+nosonwise]  
```bash
;; ANSWER SECTION:
linux.org.		300	IN	A	104.18.59.123
linux.org.		300	IN	A	104.18.58.123
```

“附加”部分为我们提供了有关权威部分中显示的权威DNS服务器的IP地址的信息。[+noadditional]  
```bash
;; ADDITIONAL SECTION:
lia.ns.cloudflare.com.	84354	IN	A	173.245.58.185
lia.ns.cloudflare.com.	170762	IN	AAAA	2400:cb00:2049:1::adf5:3ab9
mark.ns.cloudflare.com.	170734	IN	A	173.245.59.130
mark.ns.cloudflare.com.	170734	IN	AAAA	2400:cb00:2049:1::adf5:3b82
```

dig输出的最后一部分包括有关查询的统计信息。[+nostats]  
```bash
;; Query time: 58 msec
;; SERVER: 192.168.1.1#53(192.168.1.1)
;; WHEN: Fri Oct 12 11:46:46 CEST 2018
;; MSG SIZE  rcvd: 212
```

### 使用 dig 查询域名

**常用形式：**  
`dig +nocmd [domain] [type] +noall +answer`  
type:
- a
- cname
- txt
- mx
- ns
- any

### 反向 DNS 查询

使用[-x] , 查询 ip 地址对应的域名

`dig -x [ip] +noall +answer`

### 批量查询

使用[-f]

`dig -f domains.txt +short`

### dig 行为控制（.digrc File  .digrc文件）

dig命令的行为可以通过在${HOME}/.digrc文件中设置每个用户的选项来控制。  
例如，如果只想显示答案部分，请打开文本编辑器并创建以下~/.digrc文件：  
![alt text](assert/linux-basic-command_dig2.png)
+nocmd +noall +answer

# 文本查询工具

## 正则表达式

| 元字符 | 意义                           | 举例       | 举例意义                          | 可匹配结果                       |
|--------|--------------------------------|------------|-----------------------------------|----------------------------------|
| ^      | 行开头                         | ^aa        | 以aa开头                          | aa aaa aa_ aa123 …               |
| $      | 行结尾                         | aa$        | 以aa结尾                          | aa 312aa …                       |
| ^$     | 空行                           |            |                                   |                                  |
| ^aa$   | 整行只有aa                     | ^a$        | 匹配只有a的行，aba也不行          | a                                |
| ?      | 前面的字符出现1次以下          | ab?        | 出现0次或1次，就是不确定的意思    | a ab                             |
| +      | 前面的字符出现1次以上          | ab+        | b出现1次以上                      | ab abb abbb ……                   |
| {}     | 指定前面的字符出现次数范围     | ab{2,}c    | b出现2次以上,{,5}这是5次以下      | abbc abbbc ……                    |
| *      | 前面的字符出现任意次           | ab*        | b出现0次或多次                    | a ab abb abbb ……                 |
| .      | 除换行符外任意一个字符         | gr.p       | gr后接一个任意字符然后是p         | grep grap gr@p gr1p ……           |
| .{}    | 指定任意字符出现次数           | ab.{2,4}   | b后面出现2到4个任意字符           | ab#$ ab123 ab!4rq ab…            |
| .*     | 代表任意个任意字符             |            |                                   | ……                               |
| |      | 或                             | a\|b       | 包含a或b                          |                                  |
| []     | 文字字符域，单个字符匹配       | [abc]      | 包含a、b、c任意一个，等同a\|b\|c  | awd _23rc bbb                    |
| [^]    | 取反，只要一行里有一个字符符合条件整行都会匹配 | [^abc] | 不包含abc中任意一个字符          | qwe a13 ……                       |
| \s     | 空白符(空格、tab)              | a\sb       | a和b之间有空白                    | a b、a    b                      |
| \<、\b | 单词开头                       | \<re       | 单词以re开头                      | remove reverse review ……         |
| \>、\b | 单词结尾                       | tion\>     | 单词以tion结尾                    | function revolution ……           |
| ()     | 实现多个字符分组               | f(oo)*     | oo可以出现任意次                  | f foo foooo ……                   |

## grep

`grep [options] parttern [files]`  
返回文件中符合格式的内容  
 Global search Regular Expression and Print out the line

[-v] 返回不符合的内容  
[-R] 递归（包括软链接）  
[-l] 只返回文件名  
[-i] 不区分大小写  
[-w] 匹配完整的单词，不再匹配单词内字符串  
[-n] 显示行号  
[-c] 计数  
[-o] 只匹配指定内容,输出也是指定内容  
[-A] 多显示匹配内容之后的n行 --after
[-B] 多显示匹配内容之前的n行 --before
[-C] 多显示匹配内容前后的n行 --center

## egrep

和 `grep -E` 相同

其主要是对正则表达式中增加了以下元字符：

1. | 表示或，例如 egrep ‘a|b’ 表示匹配a或b
2. + 表示一次或多次重复，例如 egrep ‘a+’ 表示匹配一个或多个a
3. ? 表示零次或一次重复，例如 egrep ‘a?’ 表示匹配零个或一个a
4. () 表示分组，例如 egrep ‘(ab)+’ 表示匹配ab的一个或多个重复
5. {} 表示出现次数，例如 egrep ‘a{3}’ 表示匹配a出现3次
