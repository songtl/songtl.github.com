---
layout: post
title: "MPLS VPN中MP-IBGP邻居建立的问题"
description: ""
category: "Network" 
tags: [MPLS ,VPN ,MP-IBGP ,Loopback /32]
---
{% include JB/setup %}
在MPLS VPN实验中有可能会出现CE路由器之间能学到对方的私网路由，但无法ping通的情况，这很有可能是使用不正确的接口建立MP-IBGP邻居导致的！

在PE之间建立MP-IBGP邻居传送VPNv4路由时需要注意

*	建议使用Loopback接口建立MP-IBGP邻居

*	如果MPLS内部运行OSPF协议，建立邻居用的Loopback接口掩码设为32位，或者改为点对点网络类型

![](http://pic.yupoo.com/songtl/CRZp2gA6/medish.jpg)

R1、R4使用各自的的物理接口S0/1建立MP-IBGP邻居传送VPNv4路由,会有如下提示：

	*May 18 06:58:24.295: %BGP-4-VPN_NH_IF: Nexthop 192.168.12.1 may not be reachable from neigbor 192.168.34.4 - not a loopback

此时MP-IBGP邻居关系还是能正常建立的，并且R7、R10之间也能正常学习到对方的内部网络路由，但双方之间是ping不通的。
	
	R7#sh ip route           
		Gateway of last resort is not set
		7.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
	C        7.7.7.0/24 is directly connected, Loopback1
		10.0.0.0/24 is subnetted, 1 subnets
	R        10.10.10.0 [120/2] via 192.168.17.1, 00:00:13, Serial1/2
	    192.168.17.0/24 is variably subnetted, 2 subnets, 2 masks
	C        192.168.17.0/24 is directly connected, Serial1/2
	R     192.168.40.0/24 [120/1] via 192.168.17.1, 00:00:13, Serial1/2

当R7发送给R10的数据包到达R1时是一个纯IP数据包，R1不会查询FLIB表，而是根据FIB表为该数据包打上两层MPLS标签（16,21），成为MPLS数据包发送给R2。

	R1#sh ip cef vrf MPLSVPN 10.10.10.0 detail 
	10.10.10.0/24, epoch 0, flags rib defined all labels
	  recursive via 192.168.34.4 label 21
		recursive via 192.168.34.0/24
		  nexthop 192.168.12.2 Serial1/1 label 16

此时在R2上开启`Debug mpls packet`命令可以看到R2收到一个带两层标签（16,21）的MPLS数据包，然后把外层标签（16）弹出再发送出去,此时的MPLS数据包就只剩内层标签（21）了。

	*May 18 07:57:10.095: MPLS les: Se1/1: rx: Len 112 Stack {16 0 254} {21 0 254} - ipv4 data s:192.168.17.7 d:10.10.10.10 ttl:254 tos:0 prot:1
	*May 18 07:57:10.095: MPLS les: Se1/0: tx: Len 108 Stack {21 0 253} - ipv4 data s:192.168.17.7 d:10.10.10.10 ttl:254 tos:0 prot:1

因为R2的FLIB中收到带16标签的MPLS数据包时的执行动作是Pop Label（弹出标签）

	R2#show mpls forwarding-table 192.168.34.0
	Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
	Label      Label      or Tunnel Id     Switched      interface              
	16         Pop Label  192.168.34.0/24  16636         Se1/0      point2point 

R3上同样开启Debug命令可以看到R3收到了R2发送来的带21标签的MPLS数据包，但并没显示有数据包被发送出去

	*May 18 07:57:10.100: MPLS les: Se1/1: rx: Len 108 Stack {21 0 253} - ipv4 data s:192.168.17.7 d:10.10.10.10 ttl:254 tos:C0 prot:1

因为R3的FLIB表中并没有相关表项说明当收到带21标签的MPLS数据包该如何处理，因此数据包被丢弃,根本无法到达R4,更不用说到达R10了。

	R3#show mpls forwarding-table 
	Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
	Label      Label      or Tunnel Id     Switched      interface              
	16         Pop Label  192.168.12.0/24  10703         Se1/0      point2point 
	17         17         1.1.1.1/32       0             Se1/0      point2point 
	18         Pop Label  2.2.2.2/32       0             Se1/0      point2point 
	19         Pop Label  4.4.4.4/32       0             Se1/1      point2point

其实根本原因是R1,R4使用物理接口s0/1建立MP-IBGP邻居，在R1上10.10.10.0/24的下一跳为192.168.34.4，R3是该地址的最后一跳路由器，而R2就是次末跳路由器，根据默认开启的PHP（Penultimate Hop Popping/次末跳弹出）原则，也就能解释为什么R2会弹出最外层标签了。如果把R1,R4的s0/0接口宣告进了MPLS的IGP内,用R1、R4的是s0/0来建立MP-IBGP邻居，这样应该是没问题的，当然思科官方的文档是建议使用Loopback接口来建立MP-IBGP邻居！

如果MPLS内部运行OSPF协议，R1、R4之间使用Loopback接口建立MP-IBGP邻居，但Loopback接口掩码为24位,这种情况路由器也会弹出提示

	*May 18 06:58:24.295: %BGP-4-VPN_NH_IF:  Nexthop 1.1.1.1 may not be reachable from neigbor 4.4.4.4 - not /32 mask

此时MP-IBGP邻居能正常建立，CE路由器之间也能学习到对方的路由，但不能ping通对方的内部网络

因为OSPF默认把Loopback接口通告为32位的主机路由，R1、R2、R3学到R4的Loopback口为4.4.4.4/32

	R3#show ip route | include 4.4.4 
	O        4.4.4.4 [110/65] via 192.168.34.4, 01:31:40, Serial1/1	

而R4本地的路由表中关于Loopback口的路由为4.4.4.0/24

	R4#sh ip route | include 4.4.4
	C        4.4.4.0/24 is directly connected, Loopback1

LDP是根据路由表来分发标签的，R3为4.4.4.4/32分发了标签19

	R3#show mpls ldp bindings                                                    
	lib entry: 4.4.4.4/32, rev 14
	   local binding:  label: 19
	   remote binding: lsr: 2.2.2.2:0, label: 19

R4没有4.4.4.4/32这条主机路由，所以不为它分配标签

	R4#show mpls ldp bindings 
	lib entry: 4.4.4.4/32, rev 17(no route)
	   remote binding: lsr: 3.3.3.3:0, label: 19

所以R3对收到标签为19的MPLS数据包时执行的动作是No Labels(低版本的IOS叫untag),即弹出所有标签，还原成纯IP数据包发出
	
	R3#show mpls forwarding-table | in 4.4.4 
	19         No Label   4.4.4.4/32       3696          Se1/1      point2point 

R4收到这个纯IP数据包，而它的公网路由表有没有10.10.10.0/24这个路由，所以数据包也被丢弃了。解决方法也很简单，把Loopback接口的掩码改为32位，或者把Loopback接口的网络类型改为Point-to-Point,或者MPLS间跑EIGRP，RIP等其他IGP协议。
