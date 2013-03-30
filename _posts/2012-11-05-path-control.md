---
layout: post
title: 路径控制综合实验
categories: Network
tags: [path control, cisco]
---

这个实验是朱SIR 的CCNP教程里面特别提到的，是一个非常有意义和巧妙的一个实验。做这个实验花了不少时间，没有特别复杂的命令，不过有比较严格的限制，实现需求的时候必须瞻前顾后！

![](http://songtl.com/wp-content/uploads/2012/11/QQ½ØÍ¼20121101101514.png)

**需求一分析：**

只要把R5的Loopback口直连重分布到RIP进程里，在R2上把RIP、OSPF重分布到EIGRP进程中，R1就能够学习到23.0、24.0、100.0、以及R5的所有Loopback接口地址。把EIGRP分别重分布到RIP、OSPF，让R3、R4、R5学习到172.16.1.0/24、192.168.12.0/24的路由。

**需求二分析：**

在第一个需求完成后也相应完成了，因为R4的100.4/24接口不激活任何动态路由协议，所以R1只能通过R2重分布进EIGRP进程的RIP学习到去往R5下属Loopback口的路由,路径为 R1>>R2>>R3>>R5 ，同理R5也只从R3学习到172.16.1.0/24、192.168.12.0/24的路由，路径 R5>>R3>>R2>>R1

**需求三分析：**

当R2-R3之间的链路断掉后，R2路由表必然丢失了100.0/24和R5下属Loopback口的路由，重发布到EIGRP失败，R1没有去往100.0/24和R5下属Loopback口的路由。R5路由表也丢失172.16.1.0/24、192.168.12.0/24的路由，此时R1和R5不能通信！需要解决两个问题：

**1.R1去往R5的路由**

这里可以在R4设置去往172.16.65.0/24 、172.16.66.0/24、172.16.67.0/24的静态路由，并重分布到OSPF进程让R2学习。不过如果这样设置，在正常情况（R2-R3链路正常）下，R2将分别从R3（RIP）、R4（OSPF）学到172.16.65.0/24、172.16.66.0/24、172.16.67.0/24这三条路由，因为OSPF的AD值（110）小于RIP的AD值（120），因此R2只会选择通过R4（OSPF）学到的路由，并传递给R1。即正常情况下R1去R5的下属Loopback接口会走R1>>R2>>R4>>R5这个路径，与第二个需求冲突了。显然这样设置不合理！根据路由选路的最长匹配原则，可以在R4设置一条汇总路由172.16.64.0/22代替三条明细路由，并静态重分布进OSPF，此时R2通过R4（OSPF）学习到去往R5下属Loopback口的路由为172.16.64.0/22。这样R2就有两条去往R5下属Loopback口的路由，一条明细（从R3学习），一条汇总（从R4学习）。 根据最长匹配原则，正常情况R2优先选择掩码长的路由（明细），路径为R2>>R3>>R5 ,当R2-R3链路失效，明细路由也将丢失。此时就会选择汇总路由，即R2>>R4>>R5 这个路径。

**2.R5回来R1的路由**

R5通过设置去往172.16.1.0/24、192.168.12.0/24的静态路由将数据包丢给R4,让R4来选路。需要注意，静态路由AD值（1）比RIP的AD值（120）小，为了在正常情况下让R5去R1首选RIP路由（即R5>>R3>>R2>>R1路径），必须为静态路由指定一个比120大的AD值,如130。

**需求四分析：**

前半部分容易实现，只要把172.16.4.0/24 直连重分布到OSPF进程，R2就能学到4.0/24的路由并传递给R1，去的路径为R2>>R4,回来的路径为R4>>R2。如果R2-R4链路失效，R2学习到的4.0/24路由丢失，导致R1去往4.0/24的路由也丢失。R4学习到的1.0/24的路由也丢失。此时也必须解决两个问题

**1.R1去往R4的路由**

去的路由可以在R5上设置一条去往172.16.4.0/24的静态路由下一跳为100.4/24，并重分布到RIP进程，让R2学习到并传递给R1。但是R5是以100.5/24这个接口与R3交换RIP路由信息的，而这条静态路由的出接口也是100.5/24。根据水平分割原则，R5不会把这条路由更新给R3。解决方法为将R3、R5设置为单播通信（设置被动接口，手动指定邻居），R3才能学到这条路由并传递给R2、R1。 

**2.R4回来R1的路由**

可以在R4上设置172.16.1.0/24、192.168.12.0/24的下一跳为R3,让R3来选路。但静态路由AD（1）小于OSPF的AD值（110），为了让R4在正常情况（R2-R4链路正常）优先选择OSPF学到的路由（即路径为R2>>R4），需要为静态路由设置一个比110大的AD值，如120。只有OSPF学到的路由失效的时候才启用AD值比OSPF小的静态路由。

具体配置如下：

R1

    interface Loopback0
     ip address 172.16.1.1 255.255.255.0
    !         
    interface Serial0/0
     ip address 192.168.12.1 255.255.255.0
     no sh
     exit
    !
    router eigrp 100
     network 172.16.0.0
     network 192.168.12.0
     no auto-summary
    !

R2

    interface Serial0/0
     ip address 192.168.12.2 255.255.255.0
     no sh
    !         
    interface Serial0/1
     ip address 192.168.23.2 255.255.255.0
     no sh
    !         
    interface Serial0/2
     ip address 192.168.24.2 255.255.255.0
     no sh
     exit
    !
    router eigrp 100
     redistribute rip metric 1500 100 255 1 1500
     redistribute ospf 1 metric 1500 100 255 1 1500
     network 192.168.12.0
     exit
    !         
    router ospf 1
     redistribute eigrp 100 subnets
     network 192.168.24.2 0.0.0.0 area 0
     exit
    !         
    router rip
     version 2
     no auto-summary
     network 192.168.23.0
     redistribute eigrp 100 metric 2
     exit
    !

R3

    interface Serial0/1
     ip address 192.168.23.3 255.255.255.0
     no sh 
    !
    interface FastEthernet1/0
     ip address 100.100.100.3 255.255.255.0
     no sh 
     exit
    !
    router rip
     version 2
     passive-interface FastEthernet1/0
     network 100.0.0.0
     network 192.168.23.0
     neighbor 100.100.100.5
     no auto-summary
    !

R4

    interface Loopback0
     ip address 172.16.4.1 255.255.255.0
    !
    interface Serial0/2
     ip address 192.168.24.4 255.255.255.0
     no sh
    !
    interface FastEthernet1/0
     ip address 100.100.100.4 255.255.255.0
    !
    router ospf 1
     network 192.168.24.4 0.0.0.0 area 0
     redistribute connected subnets
     redistribute static subnets
    !
    ip classless
    ip route 172.16.1.0 255.255.255.0 100.100.100.3 120
    ip route 172.16.64.0 255.255.252.0 100.100.100.5
    ip route 192.168.12.0 255.255.255.0 100.100.100.3 120
    ip route 192.168.23.0 255.255.255.0 100.100.100.3
    !

R5

    interface Loopback0
     ip address 172.16.65.1 255.255.255.0
    !
    interface Loopback1
     ip address 172.16.66.1 255.255.255.0
    !         
    interface Loopback2
     ip address 172.16.67.1 255.255.255.0
    ! 
    interface FastEthernet1/0
     ip address 100.100.100.5 255.255.255.0
     no sh
     exit
    !
    router rip
     version 2
     passive-interface FastEthernet1/0
     network 100.0.0.0
     neighbor 100.100.100.3
     no auto-summary
     redistribute connected
     redistribute static metric 3 （注意metric一定要大于2）
    !
    ip classless
    ip route 172.16.1.0 255.255.255.0 100.100.100.4 130
    ip route 172.16.4.0 255.255.255.0 100.100.100.4
    ip route 192.168.12.0 255.255.255.0 100.100.100.4 130
    !

实验没有难懂命令，重点设置是R4、R5的几条静态浮动路由。注意R5设置静态路由重分布到RIP时一定要设置metric值并且要大于2。因为原本R3通过R2学习到的172.16.1.0/24、192.168.12.0/24这两条路由的metric值为2,下一跳是192.168.23.2！一旦R2-R3链路断掉后，这两条路由失效，R3转而从R5重分布进来的静态路由学习到这两条路由，下一跳是100.100.100.4/24。当R2-R3链路恢复后，R3会从R2重新收到这两条路由的更新，此时只有R2更新过来的路由的metric比从R5更新过来的路由的metric优才会选择R2作为下一跳,因此为了让R3正常情况下首选R2更新过来的路由，必须设置R5静态路由重分布时的metric值大于2（让R2过来的路由比R5过来的路由跳数少）！
