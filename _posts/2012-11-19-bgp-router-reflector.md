---
layout: post
title: 解决BGP路由黑洞：路由反射器
categories: Network
tags: [BGP,Router Reflector,路由黑洞]
---

BGP为了解决环路问题，设计了IBGP水平分割原则：从IBGP邻居收到的路由不再传递给其他IBGP邻居。但这产生了路由黑洞、路由学习不完整一系列问题。上一篇[《解决BGP路由黑洞：联邦》](http://songtl.com/bgp-confederation.html)通过联邦打破了IBGP水平分割，但解决问题并非只有一种方法，这里使用另一种方法：路由反射器。


![](http://songtl.com/wp-content/uploads/2012/11/Screenshot-11182012-020959-PM.png)

路由反射器（Router Reflector）是典型的C/S模型，RR作为服务器（Server），它的IBGP邻居可以设置为客户端（Client）和非客户端（Non-Client），规则如下：

1.RR通过EBGP邻居学习到的路由，会传递给其他EBGP邻居（如果有）、Client以及Non-Client  

2.RR通过Client（IBGP邻居）学习到的路由，会传递给EBGP邻居（如果有）、其他Client以及Non-Client  

3.RR通过Non-Client（IBGP邻居）学习到的路由，只会传递给EBGP邻居（如果有）和Client 
其实总结起来很简单：RR在传递路由时候，Client跟EBGP邻居具有相同的待遇。  
  
配置命令：

    bgp neighbor x.x.x.x router-reflector-client          //将本路由设置为RR，同时将x.x.x.x设置为Client

在这个拓扑中，只要将R3设置为RR，R2、R3任意一个设置为Client就能打破IBGP水平分割，但建议把R2,R3都设置成Client。

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
     bgp log-neighbor-changes
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
    router bgp 3
     no synchronization
     bgp router-id 2.2.2.2
     neighbor 3.3.3.3 remote-as 3
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
    router bgp 3
     no synchronization
     bgp router-id 3.3.3.3
     neighbor 2.2.2.2 remote-as 3
     neighbor 2.2.2.2 update-source Loopback0
     neighbor 2.2.2.2 route-reflector-client
     neighbor 4.4.4.4 remote-as 3
     neighbor 4.4.4.4 update-source Loopback0
     neighbor 4.4.4.4 route-reflector-client
     no auto-summary
    !
    

R4配置

    interface Loopback0
     ip address 4.4.4.4 255.255.255.0
    !
    interface Serial0/0
     ip address 192.168.34.4 255.255.255.0
     serial restart-delay 0
    !         
    interface Serial0/1
     ip address 192.168.45.4 255.255.255.0
    !
    router ospf 110
     router-id 4.4.4.4
      network 4.4.4.4 0.0.0.0 area 0
     network 192.168.34.4 0.0.0.0 area 0
    router bgp 3
     no synchronization
     bgp router-id 4.4.4.4
     neighbor 3.3.3.3 remote-as 3
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
     bgp log-neighbor-changes
     network 5.5.5.0 mask 255.255.255.0
     neighbor 192.168.45.4 remote-as 3
     no auto-summary
    !
