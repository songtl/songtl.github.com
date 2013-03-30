---
layout: post
title: 三层交换与DHCP中继实验
categories: Network
tags: [DHCP, 三层交换]
---

**一、实验目标：**

**1、验证VLAN通过三层交换实现全网互通，掌握三层交换的基本配置。  
2、验证PC能通过DHCP中继向其它网段的DHCP服务器获取IP，掌握DHCP中继与服务器的基本配置。**  
  
**二、实验过程：**

**1、首先在GNS3里面搭好拓扑，给S1，S2，S3分别添加一个NM-16ESW交换模块，给R1加一个NM-1FE-TX快速以太网口**

![](http://songtl.com/wp-content/uploads/2012/08/未命名.jpg)

**2、S1、S2、S3、R1基本信息配置**

    router>en
    router#conf t
    router(config)#hostname S1/S2/S3/R1
    s1(config)#no ip domain lookup
    s1(config)#line console 0
    s1(config-line)#logging synchronous
    s1(config-line)#exit
    

**3、s1、s2、s3 VLAN划分（s1为例）**

    s1#vlan database
    s1(vlan)#vlan 10
    s1(vlan)#vlan 11
    

**4、Access、Trunk模式设置**
s1的配置

    s1(config)#int f0/1
    s1(config-if)#switchport mode access
    s1(config-if)#switchport access vlan 10
    s1(config-if)#no shutdown
    s1(config-if)#int f0/2
    s1(config-if)#switchport mode access
    s1(config-if)#switchport access vlan 11
    s1(config-if)#no shutdown
    s1(config-if)#exit
    s1(config)#int range f0/0 ,f0/3
    s1(config-range)# switchport trunk encapsulation dot1Q
    s1(config-range)#switchport mode trunk
    s1(config-range)#no shutdown
    

s2的配置

    s2(config)#int f0/1
    s2(config-if)#switchport mode access
    s2(config-if)#switchport access vlan 10
    s2(config-if)#no shutdown
    s2(config-if)#int f0/2
    s2(config-if)#switchport mode access
    s2(config-if)#switchport access vlan 11
    s2(config-if)#no shutdown
    S2(config-if)#int f0/3
    s2(config-if)# switchport trunk encapsulation dot1Q
    s2(config-if)#switchport mode trunk
    s2(config-if)#no shutdown
    

s3的配置

    s3(config)#int f0/0
    s3(config-if)#switchport trunk encapsulation dot1Q
    s3(config-if)#switchport mode trunk
    S3(config-if)#no shutdown
    

**5、S3 IP地址、DHCP中继配置**

    s3(config)#int vlan 10
    s3(config-if)#ip address 192.168.1.1 255.255.255.0
    s3(config-if)#ip helper-address 192.168.3.2
    s3(config-if)#no shutdown
    s3(config-if)#int vlan 11
    s3(config-if)#ip address 192.168.2.1 255.255.255.0
    s3(config-if)#ip helper-address 192.168.3.2
    s3(config-if)#no shutdown
    s3(config-if)#int f0/1
    s3(config-if)#no switchport
    s3(config-if)#ip address 192.168.3.1 255.255.255.0
    s3(config-if)#no shutdown
    s3(config)#service dhcp
    

**6、R1 IP地址、DHCP服务器设置**

    R1(config)#int f0/0
    R1(config-if)#ip address 192.168.3.2 255.255.255.0
    R1(config-if)#no shutdown
    R1(config-if)#exit
    R1(config-if)#service dhcp
    R1(config-if)#ip dhcp pool vlan10
    R1(dhcp-config)#network 192.168.1.0 255.255.255.0
    R1(dhcp-config)#default-router 192.168.1.1
    R1(dhcp-config)#dhcp-server 8.8.8.8
    R1(dhcp-config)#exit
    R1(config)#ip dhcp pool vlan11
    R1(dhcp-config)#network 192.168.2.0 255.255.255.0
    R1(dhcp-config)#default-router 192.168.2.1
    R1(dhcp-config)#dhcp-server 8.8.8.8
    R1(dhcp-config)#exit
    R1(config)#ip dhcp exclude-address 192.168.1.1
    R1(config)#ip dhcp exclude-address 192.168.2.1
    R1(config)#ip default-gateway 192.168.3.1
    R1(config)#ip router 0.0.0.0 0.0.0.0 192.168.3.1
    

**三、实验结果**

**配置完成，打开VPC，通过DHCP获取ip地址**  
![](http://songtl.com/wp-content/uploads/2012/08/未命名1.jpg)

**全部获得IP地址，show查看一下**  
![](http://songtl.com/wp-content/uploads/2012/08/未命名21.jpg)

**PC1 ping PC4**  
![](http://songtl.com/wp-content/uploads/2012/08/未命名3.jpg)


三层交换与DHCP中继实验到此结束。
