---
layout: post
title: OSPF常见LSA类型及特殊区域
categories: Network
tags: [Area, LSA, OSPF]
---

初学OSPF很容易被各种LSA类型和特殊区域搞晕，但是为了更好的掌握OSPF这个广泛使用的动态路由协议，还是需要搞清楚了LSA的来源、包含信息和传播区域，以及各种特殊区域存在的LSA类型。

###LSA类型###

**LSA1：Router LSA**  

所有运行OSPF的路由器都产,描述了路由器所有接口、链路及他们的 Cost 值。只在本区域内传播

**LSA2：Network LSA**  

由MA类型网络内的DR、BDR产生，包含所有DRother的RID，也包含自己的RID。只在本区域内传播

**LSA3：Network Summary LSA**  

由ABR产生，通告其他区域的路由信息， Link ID 为目标网段的 ID。在整个OSPF域内传播

**LSA4：ASBR Summary LSA**  

由ABR产生，通告ASBR路由器的地址。在整个OSPF域内传播

**LSA5：AS External LSA**  

由ASBR产生，通告OSPF域外的路由信息。在整个OSPF域内传播

**LSA7：NSSA External LSA**  

由NSSA特殊区域内的ASBR产生，通告OSPF域外的路由信息。只在本区域内传播，经过ABR时转换为LSA5在整个OSPF域内传播。

###特殊区域###

**Stub Area**：阻拦4、5类LSA进入，并自动下发默认路由 ，不允许重发布外部路由

**Totally Stub Area**：阻拦3、4、5类LSA，并自动下发默认路由，不允许重发布外部路由

**Not-So-Stubby Area（NSSA）**：阻拦4、5类LSA，需手动下发默认路由，允许重发布外部路由

**Totally NSSA**：阻拦3、4、5类LSA，并自动下发默认路由，允许重发布外部路由

![](http://pic.yupoo.com/songtl/CLfPYvdD/medish.jpg)

R1配置
    
    int f1/0
      ip add 192.168.12.1 255.255.255.0
      clock rate 64000
      no sh
    !
    router ospf 1
      router-id 1.1.1.1
      network 192.168.12.1 0.0.0 area 1
    !

R2配置

    int f1/0
      ip add 192.168.12.2 255.255.255.0
      no sh
    int s0/1
      ip add 192.168.23.2 255.255.255.0
      clock rate 64000
      no sh
    !
    router ospf 1
      router-id 2.2.2.2
      network 192.168.12.2 0.0.0.0 area 1
      network 192.168.23.2 0.0.0.0 area 0
    !

R3配置

    int s0/1
      ip add 192.168.23.3 255.255.255.0
      no sh
    int s0/0
      ip add 192.168.34.3 255.255.255.0
      clock rate 64000
      no sh
    !
    router ospf 1
      router-id 3.3.3.3
      network 192.168.23.3 0.0.0.0 area 0
      network 192.168.34.3 0.0.0.0 area 2
    !

R4配置

    int s0/0
      ip add 192.168.34.4 255.255.255.0
      no sh
    int lo 0
      ip add 4.4.4.4 255.255.255.0
    !
    router ospf 1
      router-id 4.4.4.4
      network 192.168.34.4 0.0.0.0 a 2
      redistribute connect subnet
    !
    

R1与R2通过F口连接属于MA网络，R1存在1、2、3、4、5类LSA

    R1#sh ip ospf da
    
               OSPF Router with ID (1.1.1.1) (Process ID 1)
    
    		Router Link States (Area 1)
    
    Link ID         ADV Router      Age         Seq#       Checksum Link count
    2.2.2.2         2.2.2.2         1111        0x80000007 0x003EEE 1
    
    		Net Link States (Area 1)
    
    Link ID         ADV Router      Age         Seq#       Checksum
    192.168.12.2    2.2.2.2         1111        0x80000003 0x008B21
    
    		Summary Net Link States (Area 1)
    
    Link ID         ADV Router      Age         Seq#       Checksum
    192.168.23.0    2.2.2.2         1111        0x80000003 0x001C56
    192.168.34.0    2.2.2.2         1111        0x80000003 0x002502
    
    		Summary ASB Link States (Area 1)
    
    Link ID         ADV Router      Age         Seq#       Checksum
    4.4.4.4         2.2.2.2         610         0x80000003 0x00871A 
    		Type-5 AS External Link States
    
    Link ID         ADV Router      Age         Seq#       Checksum Tag
    4.4.4.0         4.4.4.4         807         0x80000003 0x00DAA7 0

R1路由表存在3类LSA学到的路由(O IA) 和5类LSA学到的R4重发布到OSPF进程的外部路由4.4.4.0/24 (O E2)

    R1#sh ip ro
    ------------------Omit----------------------
    Gateway of last resort is not set
    
    C    192.168.12.0/24 is directly connected, FastEthernet1/0
         4.0.0.0/24 is subnetted, 1 subnets
    O E2    4.4.4.0 [110/20] via 192.168.12.2, 01:23:30, FastEthernet1/0
    O IA 192.168.23.0/24 [110/65] via 192.168.12.2, 01:30:29, FastEthernet1/0
    O IA 192.168.34.0/24 [110/129] via 192.168.12.2, 01:30:29, FastEthernet1/0
    

把Area 1设置成Stub区域后,R1上只剩下1、2、3类LSA，同时通过5类LSA学到的外部路由4.4.4.0/24消失，但多了一条通过三类LSA学到的默认路由（O\*IA）

    R1#sh ip os da
    
                OSPF Router with ID (192.168.12.1) (Process ID 1)
    
    		Router Link States (Area 1)
    
    Link ID         ADV Router      Age         Seq#       Checksum Link count
    2.2.2.2         2.2.2.2         5           0x8000000F 0x004CDA 1
    192.168.12.1    192.168.12.1    5           0x80000002 0x004712 1
    
    		Net Link States (Area 1)
    
    Link ID         ADV Router      Age         Seq#       Checksum
    192.168.12.2    2.2.2.2         5           0x80000001 0x0056E7
    
    		Summary Net Link States (Area 1)
    
    Link ID         ADV Router      Age         Seq#       Checksum
    0.0.0.0         2.2.2.2         158         0x80000001 0x0075C0
    192.168.23.0    2.2.2.2         143         0x80000006 0x00343D
    192.168.34.0    2.2.2.2         143         0x80000006 0x003DE8
    

    R1#sh ip ro
    ---------------Omit-------------------
    Gateway of last resort is 192.168.12.2 to network 0.0.0.0
    
    C    192.168.12.0/24 is directly connected, FastEthernet1/0
    O IA 192.168.23.0/24 [110/65] via 192.168.12.2, 00:00:58, FastEthernet1/0
    O IA 192.168.34.0/24 [110/129] via 192.168.12.2, 00:00:58, FastEthernet1/0
    O*IA 0.0.0.0/0 [110/2] via 192.168.12.2, 00:00:58, FastEthernet1/0
    

把Area 1设置为Totally Stub区域后，3类LSA从3条变成一条，同时路由信息也只有一条三类默认路由（O\*IA）

    R1#sh ip os da
    
                OSPF Router with ID (192.168.12.1) (Process ID 1)
    
    		Router Link States (Area 1)
    
    Link ID         ADV Router      Age         Seq#       Checksum Link count
    2.2.2.2         2.2.2.2         16          0x80000011 0x003EE7 1
    192.168.12.1    192.168.12.1    13          0x80000004 0x00391F 1
    
    		Net Link States (Area 1)
    
    Link ID         ADV Router      Age         Seq#       Checksum
    192.168.12.1    192.168.12.1    15          0x80000001 0x007C54
    
    		Summary Net Link States (Area 1)
    
    Link ID         ADV Router      Age         Seq#       Checksum
    0.0.0.0         2.2.2.2         539         0x80000001 0x0075C0
    

    R1#sh ip ro   
    -----------------------omit---------------------------
    Gateway of last resort is 192.168.12.2 to network 0.0.0.0
    
    C    192.168.12.0/24 is directly connected, FastEthernet1/0
    O*IA 0.0.0.0/0 [110/2] via 192.168.12.2, 00:02:07, FastEthernet1/0
    

将Area 1 设置为NSSA区域，R1存在1、2、3类LSA，R1没有学习到R4重发布的外部路由4.4.4.0/24，也没有学习到默认路由，因为NSSA区域的ABR不会自动下发默认路由，需要在ABR（R2）上手动下发默认路由

    R1#sh ip os da
    
                OSPF Router with ID (1.1.1.1) (Process ID 1)
    
    		Router Link States (Area 1)
    
    Link ID         ADV Router      Age         Seq#       Checksum Link count
    1.1.1.1         1.1.1.1         35          0x80000002 0x00290D 1
    2.2.2.2         2.2.2.2         28          0x80000015 0x00CD49 1
    
    		Net Link States (Area 1)
    
    Link ID         ADV Router      Age         Seq#       Checksum
    192.168.12.2    2.2.2.2         28          0x80000001 0x003573
    
    		Summary Net Link States (Area 1)
    
    Link ID         ADV Router      Age         Seq#       Checksum
    192.168.23.0    2.2.2.2         135         0x80000003 0x00C1AA
    192.168.34.0    2.2.2.2         135         0x80000003 0x00CA56
    

    R1#sh ip ro
    ------------------------Omit-----------------------
    Gateway of last resort is not set
    
    C    192.168.12.0/24 is directly connected, FastEthernet1/0
    O IA 192.168.23.0/24 [110/65] via 192.168.12.2, 00:03:05, FastEthernet1/0
    O IA 192.168.34.0/24 [110/129] via 192.168.12.2, 00:03:05, FastEthernet1/0
    

此时在R1上开启Loopback 0口 1.1.1.1/24 ，重分布到OSPF进程，R2的F1/0接口将收到一个7类LSA，并学到1.1.1.0/24的路由（O N2）

    R2#sh ip os da
    
                OSPF Router with ID (2.2.2.2) (Process ID 1)
    
    		Router Link States (Area 0)
    
    Link ID         ADV Router      Age         Seq#       Checksum Link count
    2.2.2.2         2.2.2.2         696         0x8000000B 0x00E196 2
    3.3.3.3         3.3.3.3         1701        0x80000006 0x0085F4 2
    
    		Summary Net Link States (Area 0)
    
    Link ID         ADV Router      Age         Seq#       Checksum
    192.168.12.0    2.2.2.2         582         0x80000001 0x00219D
    192.168.34.0    3.3.3.3         1701        0x80000004 0x0082DF
    
    		Summary ASB Link States (Area 0)
    
    Link ID         ADV Router      Age         Seq#       Checksum
    4.4.4.4         3.3.3.3         1188        0x80000004 0x00E4F7
    
    		Router Link States (Area 1)
    
    Link ID         ADV Router      Age         Seq#       Checksum Link count
    1.1.1.1         1.1.1.1         154         0x80000003 0x002D06 1
    2.2.2.2         2.2.2.2         591         0x80000015 0x00CD49 1
              
                    Net Link States (Area 1)
              
    Link ID         ADV Router      Age         Seq#       Checksum
    192.168.12.2    2.2.2.2         591         0x80000001 0x003573
              
                    Summary Net Link States (Area 1)
              
    Link ID         ADV Router      Age         Seq#       Checksum
    192.168.23.0    2.2.2.2         702         0x80000003 0x00C1AA
    192.168.34.0    2.2.2.2         703         0x80000003 0x00CA56
              
                    Type-7 AS External Link States (Area 1)
              
    Link ID         ADV Router      Age         Seq#       Checksum Tag
    1.1.1.0         1.1.1.1         160         0x80000001 0x00EB2D 0
              
                    Type-5 AS External Link States
              
    Link ID         ADV Router      Age         Seq#       Checksum Tag
    1.1.1.0         2.2.2.2         153         0x80000001 0x0062BC 0
    4.4.4.0         4.4.4.4         1446        0x80000004 0x00D8A8 0
    

    R2#sh ip ro   
    --------------------Omit-------------------------
    Gateway of last resort is not set
    
    C    192.168.12.0/24 is directly connected, FastEthernet1/0
         1.0.0.0/24 is subnetted, 1 subnets
    O N2    1.1.1.0 [110/20] via 192.168.12.1, 00:03:04, FastEthernet1/0
         4.0.0.0/24 is subnetted, 1 subnets
    O E2    4.4.4.0 [110/20] via 192.168.23.3, 00:03:04, Serial0/1
    C    192.168.23.0/24 is directly connected, Serial0/1
    O IA 192.168.34.0/24 [110/128] via 192.168.23.3, 00:03:04, Serial0/1
    

由于7类LSA是转换成5类LSA在OSPF域内传播，所以R4收到的是5类LSA，并且学到的路由1.1.1.0/24为 O E2类型

    R4#sh ip os database 
    
                OSPF Router with ID (4.4.4.4) (Process ID 1)
    
    		Router Link States (Area 2)
    
    Link ID         ADV Router      Age         Seq#       Checksum Link count
    3.3.3.3         3.3.3.3         1841        0x80000006 0x00411B 2
    4.4.4.4         4.4.4.4         1577        0x80000007 0x00E173 2
    
    		Summary Net Link States (Area 2)
    
    Link ID         ADV Router      Age         Seq#       Checksum
    192.168.12.0    3.3.3.3         723         0x80000001 0x0085F4
    192.168.23.0    3.3.3.3         1841        0x80000004 0x00FB71
    
    		Summary ASB Link States (Area 2)
    
    Link ID         ADV Router      Age         Seq#       Checksum
    2.2.2.2         3.3.3.3         833         0x80000001 0x0047A0
    
    		Type-5 AS External Link States
    
    Link ID         ADV Router      Age         Seq#       Checksum Tag
    1.1.1.0         2.2.2.2         290         0x80000001 0x0062BC 0
    4.4.4.0         4.4.4.4         1579        0x80000004 0x00D8A8 0
    

    R4#sh ip ro          
    ---------------------Omit---------------------
    Gateway of last resort is not set
    
    O IA 192.168.12.0/24 [110/129] via 192.168.34.3, 00:14:43, Serial0/0
         1.0.0.0/24 is subnetted, 1 subnets
    O E2    1.1.1.0 [110/20] via 192.168.34.3, 00:07:28, Serial0/0
         4.0.0.0/24 is subnetted, 1 subnets
    C       4.4.4.0 is directly connected, Loopback0
    O IA 192.168.23.0/24 [110/128] via 192.168.34.3, 02:05:39, Serial0/0
    C    192.168.34.0/24 is directly connected, Serial0/0
    

把Area 0 设置为Totally NSSA区域，R1只剩下1、2、3类LSA，自动学习到默认路由（O\*IA）

    R1#sh ip os da
    
                OSPF Router with ID (1.1.1.1) (Process ID 1)
    
    		Router Link States (Area 1)
    
    Link ID         ADV Router      Age         Seq#       Checksum Link count
    1.1.1.1         1.1.1.1         592         0x80000003 0x002D06 1
    2.2.2.2         2.2.2.2         1028        0x80000015 0x00CD49 1
    
    		Net Link States (Area 1)
    
    Link ID         ADV Router      Age         Seq#       Checksum
    192.168.12.2    2.2.2.2         1028        0x80000001 0x003573
    
    		Summary Net Link States (Area 1)
    
    Link ID         ADV Router      Age         Seq#       Checksum
    0.0.0.0         2.2.2.2         26          0x80000001 0x00FC31
    

    R1#sh ip ro
    --------------------Omit-------------------------
    Gateway of last resort is 192.168.12.2 to network 0.0.0.0
    
    C    192.168.12.0/24 is directly connected, FastEthernet1/0
         1.0.0.0/24 is subnetted, 1 subnets
    C       1.1.1.0 is directly connected, Loopback0
    O*IA 0.0.0.0/0 [110/2] via 192.168.12.2, 00:03:05, FastEthernet1/0
    

虽然Totally Stub和Totally NSSA区域的ABR会阻止其他区域过来的3、4、5类LSA，但在这两个区域内的路由器还是会存在一个3类LSA，这个3类LSA是用来发布默认路由的。
