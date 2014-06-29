---
layout: post
title: 系统收到两个ARP Reply时的处理机制
categories: Network
tags: [ARP Reply, Linux, windows]
---

当主机发送ARP Requst（请求）包后同时收到两个ARP Reply（应答）包包含不同的MAC地址时主机会怎么处理呢？今天在公司测试时就碰到了这个问题。

    ping 172.16.14.1

用Wireshark抓到了ARP Requst、ARP Reply交互的过程，发现主机收到了两个几乎同时到达的ARP Reply报文  

![](http://pic.yupoo.com/songtl/CLgdfLla/medish.jpg)

测试发现：

*    Windows系统以最后收到的ARP Reply的内容更新ARP缓存表


*    Linux系统以最先收到的ARP Reply的内容更新ARP缓存表

windows系统中收到第一个ARP Reply时将它写入ARP缓存表，接着又收到第二个ARP Reply，于是把ARP缓存表中对应的旧表项覆盖了，这是符合逻辑的。但为什么Linux系统中ARP表项不会被第二个ARP Reply覆盖？

类Unix系统中有这么一个文件：`/proc/sys/net/ipv4/neigh/DEV/locktime`

找到的一个文档是这样解释的

    An ARP/neighbor entry is only replaced with a new one if the old is at least locktime old. This prevents ARP cache thrashing.

以上面为例，当收到第二个ARP Reply报文时，距离上次更新的时间小于locktime（Archlinux默认99秒），因此系统不会更新ARP缓存表。
