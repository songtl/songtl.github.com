---
layout: post
title: 长沙蓝狐两周实训考核项目整理
categories: Network
tags: [DHCP, HSRP, OSPF, VLAN, 蓝狐]
---

自25号从长沙回来已有十多天的时间了，今天把整个考核项目重新做一遍，记录下来。这个项目据蓝狐的老师说是湖南某公司的内部局域网项目工程，当然这不重要，重要的是在搭建的过程中能学习到的知识。因为涉及的配置会比较多，又懒得分成几个篇幅来写，所以文章比较长一些。

首先看下整体的局域网拓扑图  

![](http://songtl.com/wp-content/uploads/2012/08/未命名20.jpg)

##项目要求如下：##

* 1、使用VLAN 技术进行业务二层隔离，生产业务划分到VLAN50，办公业务划分到VLAN60，奇数编号的PC属于生产业务，偶数编号的PC属于办公业务；网管VLAN使用VLAN300；

* 2、为保障网络的高可靠性，总部采用双核心架构，SW1 与SW2互为主备，提供网关冗余并且实现流量的负载均衡，HSRP网关备份要求请参照IP地址分配表

* 3、总部运行STP，杜绝二层环路，STP根网桥、备份根网桥按流量的最佳转发路径合理配置，末端接口必须配置portfast；

* 4、为实现VLAN外流量互通，总部部署三层交换，两个分部部署单臂路由；

* 5、OSPF配置要求：

     在任何不需要形成OSPF邻居的接口上，配置OSPF被动接口；  
     配置接口bandwith与物理带宽一致，以确保OSPF Cost能反映真实的链路带宽；  
     配置点对点以太网的OSPF网络类型为点对点，以加快收敛速度；

* 6、总部核心交换机SW1、SW2的OSPF对接：S1—S2间使用Trunk链路，在该Trunk链路上使用1个点对点L3 Vlan（vlan 901）建立OSPF邻居，（该Vlan只允许在该Trunk链路上通过，本网络点对点L3 Vlan（互联VLan）规划的Vlan号码段：901——1000）

* 7、总部核心交换机SW1、SW2做DHCP Server，为总部用户分配IP地址；分部路由器RT5、RT6做DHCP Server，为所在分部用户分配IP地址.

**IP地址规划表**

![](http://songtl.com/wp-content/uploads/2012/08/未命名211.jpg)

**因为要配置的设备比较多，所以我的配置过程分为四个部分：**

* 第一部分、先配置益阳分部的局域网，实现内部互通  
* 第二部分、接着配置衡阳分部的局域网，实现内部互通  
* 第三部分、然后配置长沙总部的局域网，实现内部互通  
* 第四部分、最后配置OSPF动态路由协议，实现总部与分部互通

整个配置过程使用Dynagen SecureCRT VPCS来模拟，全网配置涉及Vlan，Trunk，单臂路由，三层交换，PVST,HSRP,DHCP,链路跟踪，链路捆绑，OSPF等知识。

###第一部分、益阳分部配置###

**1、基本信息配置**  
SW8，SW9，RT6

    router>en
    router#conf t
    router(config)#hostname SW8/SW9/RT6
    SW8(config)#no ip domain lookup
    SW8(config)#line console 0
    SW8(config-line)#logging synchronous
    SW8(config-line)#exit

**2、SW8,SW9 VLAN划分，Access，Trunk模式设置**  
SW8

    SW8#vlan database
    SW8(vlan)#vlan 50
    SW8(vlan)#vlan 60
    SW8(vlan)#vlan 300
    SW8(vlan)#exit
    SW8#conf t
    SW8(config)#int f0/3
    SW8(config-if)#switchport mode access
    Sw8(config-if)#switchport access vlan 50
    SW8(config-if)#no shutdown
    Sw8(config-if)#int f0/4
    SW8(config-if)#switchport mode access
    Sw8(config-if)#switchport access vlan 60
    Sw8(config-if)#exit
    SW8(config)#int range f0/0 ,f0/2
    SW8(config-range)#switchport trunk encapsulation dot1q
    SW8(config-range)#switchport mode trunk
    SW8(config-range)#no shutdown

SW9

    SW9#vlan database
    SW9(vlan)#vlan 50
    SW9(vlan)#vlan 60
    SW9(vlan)#vlan 300
    SW9(vlan)#exit
    SW9#conf t
    SW9(config)#int f0/3
    SW9(config-if)#switchport mode access
    SW9(config-if)#switchport access vlan 50
    SW9(config-if)#no shutdown
    SW9(config-if)#int f0/4
    SW9(config-if)#switchport mode access
    SW9(config-if)#switchport access vlan 60
    SW9(config-if)#int f0/0
    SW9(config-if)#switchport trunk encapsulation dot1q
    SW9(config-if)#switchport mode trunk
    SW9(config-if)#no shutdown

**3、SW8,SW9,RT6网管地址及单臂路由配置**  
SW8

    SW8(config)#no ip routing
    SW8(config)#int vlan 300
    SW8(config-if)#ip address 10.4.3.4 255.255.255.128
    SW8(config-if)#no shutdown
    SW8(config-if)#exit
    SW8(config)#ip default-gateway 10.4.3.1
    

SW9

    SW9(config)#no ip routing
    SW9(config)#int vlan 300
    SW9(config-if)#ip address 10.4.3.5 255.255.255.128
    SW9(config-if)#no shutdown
    SW9(config-if)#exit
    SW9(config)#ip default-gateway 10.4.3.1
    

RT6

    RT6(config)#int loopback 0
    RT6(config)#ip address 10.4.0.6 255.255.255.255
    RT6(config)#no shutdown
    RT6(config)#int f1/0
    RT6(config-if)#no shutdown
    RT6(config-if)#int f1/0.50
    RT6(confif-if)#encapsulation dot1q 50
    RT6(config-if)#ip address 10.68.3.1 255.255.255.0
    RT6(config-if)#no shutdown
    RT6(config-if)#int f1/0.60
    RT6(confif-if)#encapsulation dot1q 60
    RT6(config-if)#ip address 10.132.3.1 255.255.255.0
    RT6(config-if)#no shutdown
    RT6(config-if)#int f1/0.300
    RT6(confif-if)#encapsulation dot1q 300
    RT6(config-if)#ip address 10.4.3.1 255.255.255.128
    RT6(config-if)#no shutdown

**4、RT6 DHCP服务器配置**

    RT6(config)#service dhcp
    RT6(config)#ip dhcp pool vlan50
    RT6(dhcp-config)#network 10.68.3.0 255.255.255.0
    RT6(dhcp-config)#default-router 10.68.3.1
    RT6(dhcp-config)#dns-server 8.8.8.8
    RT6(dhcp-config)#exit
    RT6(config)#ip dhcp pool vlan60
    RT6(dhcp-config)#network 10.132.3.0 255.255.255.0
    RT6(dhcp-config)#default-router 10.132.3.1
    RT6(dhcp-config)#dns-server 8.8.8.8
    RT6(dhcp-config)#exit
    RT6(config)#ip dhcp exclude-address 10.68.3.1
    RT6(config)#ip dhcp exclude-address 10.68.3.255
    RT6(config)#ip dhcp exclude-address 10.132.3.1
    RT6(config)#ip dhcp exclude-address 10.132.3.255

###第二部分、衡阳分部配置###

**1、SW7，RT5基本信息配置**

    router>en
    router#conf t
    router(config)#hostname SW7/RT5
    SW7(config)#no ip domain lookup
    SW7(config)#line console 0
    SW7(config-line)#logging synchronous
    SW7(config-line)#exit

**2、SW7 Vlan划分，Access、Trunk模式设置**

    SW7#vlan database
    SW7(vlan)#vlan 50
    SW7(vlan)#vlan 60
    SW7(vlan)#vlan 300
    SW7(vlan)#exit
    SW7#conf t
    SW7(config)#int f0/3
    SW7(config-if)#switchport mode access
    SW7(config-if)#switchport access vlan 50
    SW7(config-if)#no shutdown
    SW7(config-if)#int f0/4
    SW7(config-if)#switchport mode access
    SW7(config-if)#switchport access vlan 60
    SW7(config-if)#int f0/0
    SW7(config-if)#switchport trunk encapsulation dot1q
    SW7(config-if)#switchport mode trunk
    SW7(config-if)#no shutdown

**3、SW7，RT5网管地址、单臂路由配置**  
SW7

    SW7(config)#no ip routing
    SW7(config)#int vlan 300
    SW7(config-if)#ip address 10.4.2.4 255.255.255.128
    SW7(config-if)#no shutdown
    SW7(config-if)#exit
    SW7(config)#ip default-gateway 10.4.2.1
    

RT5

    RT5(config)#int loopback 0
    RT5(config-if)#ip address 10.4.0.5 255.255.255.255
    RT5(config-if)#no shutdown
    RT5(config-if)#int f1/0
    RT5(config-if)#no shutdown
    RT5(config-if)#int f1/0.50
    RT5(confif-if)#encapsulation dot1q 50
    RT5(config-if)#switchport mode trunk
    RT5(config-if)#ip address 10.68.2.1 255.255.255.0
    RT5(config-if)#no shutdown
    RT5(config-if)#int f1/0.60
    RT5(confif-if)#encapsulation dot1q 60
    RT5(config-if)#switchport mode trunk
    RT5(config-if)#ip address 10.132.2.1 255.255.255.0
    RT5(config-if)#no shutdown
    RT5(config-if)#int f1/0.300
    RT5(confif-if)#encapsulation dot1q 300
    RT5(config-if)#switchport mode trunk
    RT5(config-if)#ip address 10.4.2.1 255.255.255.0
    RT5(config-if)#no shutdown

**4、RT5 DHCP服务器配置**

    RT5(config)#service dhcp
    RT5(config)#ip dhcp pool vlan50
    RT5(dhcp-config)#network 10.68.2.0 255.255.255.0
    RT5(dhcp-config)#default-router 10.68.2.1
    RT5(dhcp-config)#dns-server 8.8.8.8
    RT5(dhcp-config)#exit
    RT5(config)#ip dhcp pool vlan60
    RT5(dhcp-config)#network 10.132.2.0 255.255.255.0
    RT5(dhcp-config)#default-router 10.132.2.1
    RT5(dhcp-config)#dns-server 8.8.8.8
    RT5(dhcp-config)#exit
    RT5(config)#ip dhcp exclude-address 10.68.2.1
    RT5(config)#ip dhcp exclude-address 10.68.2.255
    RT5(config)#ip dhcp exclude-address 10.132.2.1
    RT5(config)#ip dhcp exclude-address 10.132.2.255

###第三部分、长沙总部配置###

**1、SW1，SW2，SW3基本信息配置**

    router>en
    router#conf t
    router(config)#hostname SW1/SW2/SW3
    SW1(config)#no ip domain lookup
    SW1(config)#line console 0
    SW1(config-line)#logging synchronous
    SW1(config-line)#exit

**2、SW1,SW2,SW3 VLAN划分、Acess、Trunk模式配置**  
SW3

    SW3#vlan database
    SW3(vlan)#vlan 50
    SW3(vlan)#vlan 60
    SW3(vlan)#vlan 300
    SW3(vlan)#exit
    SW3#conf t
    SW3(config)#int f0/3
    SW3(config-if)#switchport mode access
    Sw3(config-if)#switchport access vlan 50
    SW3(config-if)#no shutdown
    Sw3(config-if)#int f0/4
    SW3(config-if)#switchport mode access
    Sw3(config-if)#switchport access vlan 60
    Sw3(config-if)#exit
    SW3(config)#int range f0/0 ,f0/1
    SW3(config-range)#switchport trunk encapsulation dot1q
    SW3(config-range)#switchport mode trunk
    SW3(config-range)#no shutdown

SW1

    SW1#vlan database
    SW1(vlan)#vlan 50
    SW1(vlan)#vlan 60
    SW1(vlan)#vlan 300
    SW1(vlan)#vlan 901
    SW1(vlan)#exit
    SW1#conf t
    SW1(config)#int f0/0
    SW1(config-if)#switchport trunk encapsulation dot1q
    SW1(config-if)#switchport mode trunk
    SW1(config-if)#no shutdown
    SW1(config-if)#exit
    SW1(config)#int range f0/2 ,f0/3
    SW1(config-range)#channel-group 1 mode on
    SW1(config-range)#exit
    SW1(config)#int po1
    SW1(config-if)#switchport encapsulation dot1q
    SW1(config-if)#switchport mode trunk
    SW1(config-if)#no shutdown

SW2

    SW2#vlan database
    SW2(vlan)#vlan 50
    SW2(vlan)#vlan 60
    SW2(vlan)#vlan 300
    SW2(vlan)#vlan 901
    SW2(vlan)#exit
    SW2#conf t
    SW2(config)#int f0/0
    SW2(config-if)#switchport trunk encapsulation dot1q
    SW2(config-if)#switchport mode trunk
    SW2(config-if)#no shutdown
    SW2(config-if)#exit
    SW2(config)#int range f0/2 ,f0/3
    SW2(config-range)#channel-group 1 mode on
    SW2(config-range)#exit
    SW2(config)#int po1
    SW2(config-if)#switchport encapsulation dot1q
    SW2(config-if)#switchport mode trunk
    SW2(config-if)#no shutdown

**3、SW1,SW2 PVST配置**  

SW1

    SW1(config)#spanning-tree vlan 50 priority 0
    SW1(config)#spanning-tree vlan 60 priority 4096
    SW1(config)#spanning-tree vlan 300 priority 0
    SW1(config)#int f0/0
    SW1(config)#spanning-tree portfast
    

SW2

    SW2(config)#spanning-tree vlan 50 priority 4096
    SW2(config)#spanning-tree vlan 60 priority 0
    SW2(config)#spanning-tree vlan 300 priority 4096
    SW2(config)#int f0/0
    SW2(config)#spanning-tree portfast
    

**4、SW1,SW2,SW3网管地址、三层交换、HSRP配置**  

SW3

    SW3(config)#no ip routing
    SW3(config)#int vlan 300
    SW3(config-if)#ip address 10.4.1.132 255.255.255.128
    SW3(config-if)#no shutdown
    SW3(config-if)#exit
    SW3(config)#ip default-gateway 10.4.1.129
    

SW1

    SW1(config)#int lookback 0
    SW1(config-if)#ip address 10.4.0.1 255.255.255.255
    SW1(config-if)#no shutdown
    SW1(config-if)#int vlan 50
    SW1(config-if)#ip address 10.68.1.2 255.255.255.0
    SW1(config-if)#stanby 50 ip 10.68.1.1
    SW1(config-if)#stanby 50 preempt
    SW1(config-if)#no shutdown
    SW1(config-if)#int vlan 60
    SW1(config-if)#ip address 10.132.1.2 255.255.255.0
    SW1(config-if)#standby 60 ip 10.132.1.1
    SW1(config-if)#standby 60 priority 200
    SW1(config-if)#standby 60 preempt
    SW1(config-if)#standby 60 track FastEthernet0/1 120
    SW1(config-if)#no shutdown
    SW1(config-if)#int vlan 300
    SW1(config-if)#ip address 10.4.1.130 255.255.255.128
    SW1(config-if)#standby 3 ip 10.4.1.129
    SW1(config-if)#standby 3 preempt
    SW1(config-if)#no shutdown

SW2

    SW2(config)#int lookback 0
    SW2(config-if)#ip address 10.4.0.2 255.255.255.255
    SW2(config-if)#no shutdown
    SW2(config-if)#int vlan 50
    SW2(config-if)#ip address 10.68.1.3 255.255.255.0
    SW2(config-if)#stanby 50 ip 10.68.1.1
    SW2(config-if)#standby 50 priority 200
    SW2(config-if)#stanby 50 preempt
    SW2(config-if)#standby 50 track FastEthernet0/1 120
    SW2(config-if)#no shutdown
    SW2(config-if)#int vlan 60
    SW2(config-if)#ip address 10.132.1.3 255.255.255.0
    SW2(config-if)#standby 60 ip 10.132.1.1
    SW2(config-if)#standby 60 preempt
    SW2(config-if)#no shutdown
    SW2(config-if)#int vlan 300
    SW2(config-if)#ip address 10.4.1.131 255.255.255.128
    SW2(config-if)#standby 3 ip 10.4.1.129
    SW2(config-if)#standby 3 priority 200
    SW2(config-if)#standby 3 preempt
    SW2(config-if)#standby 3 track FastEthernet0/1 120
    SW2(config-if)#no shutdown

**4、SW1,SW2 DHCP服务器配置**  

SW1

    SW1(config)#service dhcp
    SW1(config)#ip dhcp pool vlan50
    SW1(dhcp-config)#network 10.68.1.0 255.255.255.0
    SW1(dhcp-config)#default-router 10.68.1.1 
    SW1(dhcp-config)#dns-server 8.8.8.8
    SW1(dhcp-config)#exit
    SW1(config)#ip dhcp pool vlan60
    SW1(dhcp-config)#network 10.132.1.0 255.255.255.0
    SW1(dhcp-config)#default-router 10.132.1.1 
    SW1(dhcp-config)#dns-server 8.8.8.8
    SW1(dhcp-config)#exit
    SW1(config)#ip dhcp excluded-address 10.68.1.1 10.68.1.3
    SW1(config)#ip dhcp excluded-address 10.132.1.1 10.132.1.3
    SW1(config)#ip dhcp excluded-address 10.68.1.129 10.68.1.255
    SW1(config)#ip dhcp excluded-address 10.132.1.129 10.132.1.255

SW2

    SW2(config)#service dhcp
    SW2(config)#ip dhcp pool vlan50
    SW2(dhcp-config)#network 10.68.1.0 255.255.255.0
    SW2(dhcp-config)#default-router 10.68.1.1 
    SW2(dhcp-config)#dns-server 8.8.8.8
    SW2(dhcp-config)#exit
    SW2(config)#ip dhcp pool vlan60
    SW2(dhcp-config)#network 10.132.1.0 255.255.255.0
    SW2(dhcp-config)#default-router 10.132.1.1 
    SW2(dhcp-config)#dns-server 8.8.8.8
    SW2(dhcp-config)#exit
    SW2(config)#ip dhcp excluded-address 10.68.1.1 10.68.1.128
    SW2(config)#ip dhcp excluded-address 10.132.1.1 10.132.1.128
    SW2(config)#ip dhcp excluded-address 10.68.1.255
    SW2(config)#ip dhcp excluded-address 10.132.1.255

###第四部分、全网OSPF配置###

SW1

    SW1(config)#int f0/1
    SW1(config-if)#no switchport 
    SW1(config-if)#ip address 10.4.1.1 255.255.255.252
    SW1(config-if)#ip ospf network point-to-point
    SW1(config-if)#no shutdown
    SW1(config-if)#int vlan 901
    SW1(config-if)#ip address 10.4.1.9 255.255.255.252
    SW1(config-if)#ip ospf network point-to-point
    SW1(config-if)#int f0/0
    SW1(config-if)#switchport trunk allowed vlan remove 901
    SW1(config-if)#exit
    SW1(config)#router ospf 1
    SW1(config-router)#router-id 10.4.0.1
    SW1(config-router)#network 10.4.0.1 0.0.0.0 area 0
    SW1(config-router)#network 10.4.1.0 0.0.0.3 area 0
    SW1(config-router)#network 10.4.1.8 0.0.0.3 area 0
    SW1(config-router)#network 10.4.1.128 0.0.0.127 area 0
    SW1(config-router)#network 10.68.1.0 0.0.0.255 area 0
    SW1(config-router)#network 10.132.1.0 0.0.0.255 area 0
    SW1(config-router)#passive-interface Vlan50
    SW1(config-router)#passive-interface Vlan60
    SW1(config-router)#passive-interface Vlan300

SW2

    SW2(config)#int f0/1
    SW2(config-if)#no switchport 
    SW2(config-if)#ip address 10.4.1.5 255.255.255.252
    SW2(config-if)#ip ospf network point-to-point
    SW2(config-if)#no shutdown
    SW2(config-if)#int vlan 901
    SW2(config-if)#ip address 10.4.1.10 255.255.255.252
    SW2(config-if)#ip ospf network point-to-point
    SW2(config-if)#int f0/0
    SW2(config-if)#switchport trunk allowed vlan remove 901
    SW2(config-if)#exit
    SW2(config)#router ospf 1
    SW2(config-router)#router-id 10.4.0.2
    SW2(config-router)network 10.4.0.2 0.0.0.0 area 0
    SW2(config-router)network 10.4.1.4 0.0.0.3 area 0
    SW2(config-router)network 10.4.1.8 0.0.0.3 area 0
    SW2(config-router)network 10.4.1.128 0.0.0.127 area 0
    SW2(config-router)network 10.68.1.0 0.0.0.255 area 0
    SW2(config-router)network 10.132.1.0 0.0.0.255 area 0
    SW2(config-router)passive-interface Vlan50
    SW2(config-router)passive-interface Vlan60
    SW2(config-router)passive-interface Vlan300

RT5

    RT5(config)#int s0/0
    RT5(config-if)#ip address 10.4.1.14 255.255.255.252
    RT5(config-if)#no shutdown
    RT5(config-if)#exit
    RT5(config)#router ospf 1
    RT5(config-router)#router-id 10.4.0.5
    RT5(config-router)#network 10.4.0.5 0.0.0.0 area 0
    RT5(config-router)#network 10.4.1.12 0.0.0.3 area 0
    RT5(config-router)#network 10.4.2.0 0.0.0.127 area 0
    RT5(config-router)#network 10.68.2.0 0.0.0.255 area 0
    RT5(config-router)#network 10.132.2.0 0.0.0.255 area 0
    RT5(config-router)#passive-interface FastEthernet1/0.50
    RT5(config-router)passive-interface FastEthernet1/0.60
    RT5(config-router)passive-interface FastEthernet1/0.300

RT6

    RT6(config)#int s0/0
    RT6(config-if)#ip address 10.4.1.18 255.255.255.252
    RT6(config-if)#no shutdown
    RT6(config-if)#exit
    RT6(config)#router ospf 1
    RT6(config-router)#router-id 10.4.0.6
    RT6(config-router)#passive-interface FastEthernet1/0.50
    RT6(config-router)#passive-interface FastEthernet1/0.60
    RT6(config-router)#passive-interface FastEthernet1/0.300
    RT6(config-router)#network 10.4.0.6 0.0.0.0 area 0
    RT6(config-router)#network 10.4.1.16 0.0.0.3 area 0
    RT6(config-router)#network 10.4.3.0 0.0.0.127 area 0
    RT6(config-router)#network 10.68.3.0 0.0.0.255 area 0
    RT6(config-router)#network 10.132.3.0 0.0.0.255 area 0

RT4

    RT4(config)#int f1/0
    RT4(config-if)#ip address 10.4.1.2 255.255.255.252
    RT4(config-if)#no shutdown
    RT4(config-if)#int f2/0
    RT4(config-if)#ip address 10.4.1.6 255.255.255.252
    RT4(config-if)#no shutdown
    RT4(config-if)#int s0/0
    RT4(config-if)#ip address 10.4.1.13 255.255.255.252
    RT4(config-if)#no shutdown
    RT4(config-if)#int s0/1
    RT4(config-if)#ip address 10.4.1.17 255.255.255.252
    RT4(config-if)#no shutdown
    RT4(config-if)#int loopback 0
    RT4(config-if)#ip address 10.4.0.4 255.255.255.255
    RT4(config-if)#no shutdown
    RT4(config-if)#exit
    RT4(config)#router ospf 1
    RT4(config-router)#router-id 10.4.0.4
    RT4(config-router)#network 10.4.0.4 0.0.0.0 area 0
    RT4(config-router)#network 10.4.1.0 0.0.0.3 area 0
    RT4(config-router)#network 10.4.1.4 0.0.0.3 area 0
    RT4(config-router)#network 10.4.1.12 0.0.0.3 area 0
    RT4(config-router)#network 10.4.1.16 0.0.0.3 area 0

终于配置完了，看下结果吧

通过dhcp获取ip地址

![]( http://songtl.com/wp-content/uploads/2012/08/未命名28.jpg)

**PC1 PING PC8**

![](http://songtl.com/wp-content/uploads/2012/08/未命名5.jpg)

**PC1 PING PC3**

![]( http://songtl.com/wp-content/uploads/2012/08/未命名6.jpg)

**SW3上show spanning-tree**

![](http://songtl.com/wp-content/uploads/2012/08/未命名7.jpg)

![](http://songtl.com/wp-content/uploads/2012/08/未命名8.jpg)

![](http://songtl.com/wp-content/uploads/2012/08/未命名9.jpg)

**SW1上show stanby brief**

![](http://songtl.com/wp-content/uploads/2012/08/未命名11.jpg)

**SW2上show stanby brief**

![](http://songtl.com/wp-content/uploads/2012/08/未命名12.jpg)

**RT4上show ip ospf neighbor**

![](http://songtl.com/wp-content/uploads/2012/08/未命名13.jpg)

**RT5上show ip ospf neighbor**

![](http://songtl.com/wp-content/uploads/2012/08/未命名14.jpg)

**RT6上show ip ospf neighbor**

![](http://songtl.com/wp-content/uploads/2012/08/未命名15.jpg)

**SW1上show ip ospf neighbor**  

![](http://songtl.com/wp-content/uploads/2012/08/未命名16.jpg)

**SW2上show ip ospf neighbor**

![](http://songtl.com/wp-content/uploads/2012/08/未命名17.jpg)

**SW1上show ip route**

![](http://songtl.com/wp-content/uploads/2012/08/未命名18.jpg)

整个网络是全部互通的，配置上没有什么大的问题，花好几个钟得时间终于写完这篇文章，虽然检查了几次，但文中的命令有些是自己一行行敲下来，有些是从show running复制的，不排除笔误部分,我自己在配置的过程中基本是严格按照OSI七层模型层次化配置的，但如果文章按照配置顺序写可能会显得比较乱，所以我稍微改变了一些顺序才贴出来比较好利于理解，虽然配置命令比较多，不过配命令都比较简单，所以具体的注释也没有写（写了排版不太好看，chrome里看起来整齐到了firfox就乱了，反之也一样），再者配置前都有一个说明这是配置什么。

总之，只要严格按照层次化配置，细心一些，应该都能配通整个网络。
