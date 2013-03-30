---
layout: post
title: 实验验证在路由器上发送ping包时的源IP地址
categories: Network
tags: [ping包,源IP]
---

在今天路由与交换的Trouble Shooting实验上，有一个疑问，当在路由器上ping一台设备时，会选择哪个物理网卡的IP作为ICMP ping包的源IP地址。因为老师的说法与书上不相同，因此搭建实验环境进行验证,毕竟实验才是检验真理的唯一标准。

按当时的实验拓扑配置好IP(拓扑图中忘了标R4的Loopback 0接口的IP地址：10.1.5.1/24)

![](http://songtl.com/wp-content/uploads/2012/09/Screenshot-09102012-101540-PM.png)

在R1，R2，R3，R4设置静态路由如下

R1

    ip route 10.0.0.0 255.0.0.0 192.168.1.34
    ip route 192.168.1.192 255.255.255.224 192.168.1.34

R2

    ip route 10.1.0.0 255.255.0.0 192.168.1.194
    ip route 192.168.1.0 255.255.255.224 192.168.1.65   //注意这里是故障排错实验的错误设置
    #ip route 192.168.1.0 255.255.255.192 192.168.1.65  //这才是正确的

R3

    ip route 10.1.0.0 255.255.0.0 10.4.5.1
    ip route 192.0.0.0 255.0.0.0 10.4.5.1

R4

    ip route 10.4.0.0 255.255.0.0 192.168.1.193
    ip route 192.168.1.0 255.255.255.0 192.168.1.193

我们在R2上输出数据包转发的信息

    R2#debug ip packet

然后在R1上ping路由器R4的Loopback 0接口

    R1#ping 10.1.5.1

在R2的Debug信息

	*Mar 1 00:18:15.503: IP: tableid=0, s=192.168.1.33 (Serial1/2), d=10.4.5.1 (Serial1/2), routed via RIB  
	*Mar 1 00:18:15.507: IP: s=192.168.1.33 (Serial1/2), d=10.4.5.1 (Serial1/2), len 100, rcvd 3  
	*Mar 1 00:18:15.511: IP: s=10.4.5.1 (local), d=192.168.1.33, len 100, unroutable  
	*Mar 1 00:18:17.507: IP: tableid=0, s=192.168.1.33 (Serial1/2), d=10.4.5.1 (Serial1/2), routed via RIB  
	*Mar 1 00:18:17.507: IP: s=192.168.1.33 (Serial1/2), d=10.4.5.1 (Serial1/2), len 100, rcvd 3  
	*Mar 1 00:18:17.507: IP: s=10.4.5.1 (local), d=192.168.1.33, len 100, unroutable  
	*Mar 1 00:18:19.507: IP: tableid=0, s=192.168.1.33 (Serial1/2), d=10.4.5.1 (Serial1/2), routed via RIB  
	*Mar 1 00:18:19.507: IP: s=192.168.1.33 (Serial1/2), d=10.4.5.1 (Serial1/2), len 100, rcvd 3  
	*Mar 1 00:18:19.507: IP: s=10.4.5.1 (local), d=192.168.1.33, len 100, unroutable  
	*Mar 1 00:18:21.479: IP: tableid=0, s=192.168.1.33 (Serial1/2), d=10.4.5.1 (Serial1/2), routed via RIB  
	*Mar 1 00:18:21.483: IP: s=192.168.1.33 (Serial1/2), d=10.4.5.1 (Serial1/2), len 100, rcvd 3  
	*Mar 1 00:18:21.487: IP: s=10.4.5.1 (local), d=192.168.1.33, len 100, unroutable  
	*Mar 1 00:18:23.495: IP: tableid=0, s=192.168.1.33 (Serial1/2), d=10.4.5.1 (Serial1/2), routed via RIB  
	*Mar 1 00:18:23.499: IP: s=192.168.1.33 (Serial1/2), d=10.4.5.1 (Serial1/2), len 100, rcvd 3  
	*Mar 1 00:18:23.503: IP: s=10.4.5.1 (local), d=192.168.1.33, len 100, unroutable  

其中S表示源地址（source address），D表示目的地址（destination address），可以看出源地址选择的是192.168.1.33而不是192.168.1.65，去往10.4.5.1的包转发了，但是回来192.168.1.33的包被丢弃了。因为在Trouble Shooting实验中R2的静态路由是故意设置错误的。

**最后结论：在路由器上PING另外的设备（如：终端、路由器）时，ICMP ping包选择与路由表项下一跳相连的出接口的IP地址作为源地址（即从哪个接口转发出去就选择哪个接口的IP作为源IP地址)**。
