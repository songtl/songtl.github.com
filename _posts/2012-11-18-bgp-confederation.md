---
layout: post
title: 解决BGP路由黑洞：联邦
categories: Network
tags: [BGP, confederation, 联邦, 路由黑洞]
---

BGP为了解决环路问题，设计了IBGP水平分割原则：从IBGP邻居收到的路由不再传递给其他IBGP邻居。但这产生了路由黑洞、路由学习不完整一系列问题。我们可以通过联邦有条件的打破IBGP水平分割，避免路由黑洞问题！

![](http://pic.yupoo.com/songtl/CLfPJuBA/medish.jpg)

正常情况下，邻居关系：R1与R2为EBGP ，R2与R3为IBGP，R3与R4为IBGP，R4与R5为EBGP，R1通告的AS1内部的路由通过R2传递给R3后因为水平分割原则不会继续传递给R4,同理R5通告的AS5内部的路由通过R4传递给R3后也不会继续传递给R2。这就导致了R1没有AS5的路由，R5没有AS1的路由。

通过联邦，把AS3分割成两个小的AS，R2、R3属于AS64512 ，R4属于AS64513。但在R1,R5看来他们还是属于一个大的AS3，此时AS3内部的邻居关系：R2与R3为IBGP ，R3与R4为EBGP

因为R3与R4是EBGP邻居关系，没有了IBGP水平分割的约束，会将从R2学习到AS1内部的路由传递给R4,从R4学习到的AS5内部的路由传递给R2。R1和R5会学习到对方AS的路由。

配置联邦主要有三个步骤：  
  
1.以小AS号运行BGP  
2.声明所属的大AS号  
3.小AS之间互指peers  


配置的区别主要在R2、R3、R4上

R1配置

    interface Loopback0
     ip address 1.1.1.1 255.255.255.0
    !
    interface Serial0/0
     ip address 192.168.12.1 255.255.255.0
    
    ! 
    router bgp 1
     no synchronization
     bgp router-id 1.1.1.1
     network 1.1.1.0 mask 255.255.255.0
     neighbor 192.168.12.2 remote-as 3
     no auto-summary
    !

R2配置

    interface Loopback0
     ip address 2.2.2.2 255.255.255.0
    !
    interface Serial0/0
     ip address 192.168.12.2 255.255.255.0
    
    !         
    interface Serial0/1
     ip address 192.168.23.2 255.255.255.0
    !
    router ospf 110
     router-id 2.2.2.2
     network 2.2.2.2 0.0.0.0 area 0
     network 192.168.23.2 0.0.0.0 area 0
    router bgp 64512
     no synchronization
     bgp router-id 2.2.2.2
     bgp confederation identifier 3
     neighbor 3.3.3.3 remote-as 64512
     neighbor 3.3.3.3 update-source Loopback0
     neighbor 3.3.3.3 next-hop-self
     neighbor 192.168.12.1 remote-as 1
     no auto-summary
    !
    

R3配置

    interface Loopback0
     ip address 3.3.3.3 255.255.255.0
    !
    interface Serial0/0
     ip address 192.168.34.3 255.255.255.0
    !         
    interface Serial0/1
     ip address 192.168.23.3 255.255.255.0
    !
    router ospf 110
     router-id 3.3.3.3
      network 3.3.3.3 0.0.0.0 area 0
     network 192.168.23.3 0.0.0.0 area 0
     network 192.168.34.3 0.0.0.0 area 0
    router bgp 64512
     no synchronization
     bgp router-id 3.3.3.3
     bgp confederation identifier 3
     bgp confederation peers 64513 
     neighbor 2.2.2.2 remote-as 64512
     neighbor 2.2.2.2 update-source Loopback0
     neighbor 4.4.4.4 remote-as 64513
     neighbor 4.4.4.4 ebgp-multihop 255
     neighbor 4.4.4.4 update-source Loopback0
     no auto-summary
    !

R4配置

    interface Loopback0
     ip address 4.4.4.4 255.255.255.0
    !
    interface Serial0/0
     ip address 192.168.34.4 255.255.255.0
     !         
    interface Serial0/1
     ip address 192.168.45.4 255.255.255.0
    !
    router ospf 110
     router-id 4.4.4.4
     network 4.4.4.4 0.0.0.0 area 0
     network 192.168.34.4 0.0.0.0 area 0
    router bgp 64513
     no synchronization
     bgp router-id 4.4.4.4
     bgp confederation identifier 3
     bgp confederation peers 64512
     neighbor 3.3.3.3 remote-as 64512
     neighbor 3.3.3.3 ebgp-multihop 255
     neighbor 3.3.3.3 update-source Loopback0
     neighbor 3.3.3.3 next-hop-self
     neighbor 192.168.45.5 remote-as 5
     no auto-summary
    !

R5配置

    interface Loopback0
     ip address 5.5.5.5 255.255.255.0
    !            
    interface Serial0/1
     ip address 192.168.45.5 255.255.255.0
    !
    router bgp 5
     no synchronization
     bgp router-id 5.5.5.5
     network 5.5.5.0 mask 255.255.255.0
     neighbor 192.168.45.4 remote-as 3
     no auto-summary
    !

结果就不贴了，下一篇是解决BGP路由黑洞的另一种方法：[路由反射器(Router Reflector)](http://songtl.com/bgp-router-reflector.html)
