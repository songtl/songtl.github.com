---
layout: post
title: CISCO综合实验
categories: Network
tags: [CISCO,RIP,VLAN,VTP,单臂路由]
---

昨天闲逛51CTO论坛，看到个同是网络专业学生发的一个[“求救啊！本人菜鸟，求各位高手赐教 拓扑图问题”](http://bbs.51cto.com/thread-933954-1.html) 帖子，想必应该是期末的课程设计之类的吧。正好没什么事，再者路由与交换的知识自从考了国家四级网络工程师之后忘得也差不多了。于是就配来玩玩，权当是复习吧！


原帖要求有三：

**1、网络管理要求：（20分）**  
所有的交换机及路由器都需要设置ENABLE密码，密码为自己的完整学号，所有的交换机及路由器都要求能进行远程管理（启用设备的VTY线路），并且设置远程管理登录口令为自己的完整学号。

**2、VLAN划分要求：（25分）**

请在学生网段划分三个VLAN（VLAN2、VLAN3、VLAN4、），并使用VTP域对学生网的VLAN信息进行统一管理，VTP服务器为学生网主交换机，Switch0、Switch1、Switch2三台交换机为VTP客户端，VTP域名设置为student，VTP密码为自己的完整学号。将Switch0、Switch1、Switch2的F0/2号端口划分到VLAN2，将Switch0、Switch1、Switch2的F0/3号端口划分到VLAN3，将Switch0、Switch1、Switch2的F0/4号端口划分到VLAN4。

**3、路由要求：（35分）**

使用RIP动态路由协议及单臂路由方式实现整个网络的互联。

 

当然我就是玩玩，所以路由器、交换机设置密码、远程管理（VTY）就免了，虽然也就一两行命令的事。所以第一条要求忽略掉。照着原帖的拓扑图片在Packettracer里搭好拓扑

![](http://songtl.com/wp-content/uploads/2012/06/截图-2012年06月12日-15时03分47秒.png)

IP已经规划好

![](http://songtl.com/wp-content/uploads/2012/06/截图-2012年06月12日-15时28分21秒.png)

接下来就是配置过程了：

其实需要配置的也只有Switch 0、Swtich 1、Switch 2、学生主干网交换机、Router 0、Router 1、Router 2

**1.按照ip规划表先给终端PC、SERVER配IP**

**2.VTP域的配置**  
**学生主干网交换机：**

    switch>en                                   
    switch#conf t
    switch(config)#vtp domain myvtp                                
    switch(config)#vtp mode server                              
    switch(config)#int f0/2/                                
    switch(config-if)#switchport mode trunk                       
    switch(config-if)#int f0/3
    switch(config-if)#switchport mode trunk
    switch(config-if)#int f0/4
    switch(config-if)#switchport mode trunk
    switch(config-if)#int g0/1
    switch(config-if)#switchport mode trunk
    switch(config-if)#exit

**Switch 0、Swtich 1、Switch 2：**

    switch>en
    switch#conf t
    switch(config)#vtp domain myvtp                                  
    switch(config)#vtp mode client                                  
    switch(config)#int g0/1 
    switch(config-if)#switchport mode trunk                    
    switch(config-if)#exit

**3.划分VLAN**  
**学生主干网交换机：**

    switch#vlan database
    switch(vlan)#vlan 2                                               
    switch(vlan)#vlan 3                                              
    switch(vlan)#vlan 4  
    

**Switch 0、Switch 1、Switch 2：**

    switch(config)#int f0/2
    switch(config-if)#switchport access vlan 2                         
    switch(config-if)#int f0/3
    switch(config-if)#switchport access vlan 3                            
    switch(config-if)#int f0/4
    switch(config-if)#switchport access vlan 4         
    

**4.Router 0设置单臂路由、IP地址、RIP协议**

    router>en
    router#conf t
    router(config)#int s0/2/1
    router(config-if)#ip address 192.168.1.1 255.255.255.0             
    router(config-if)#no shutdown                                     
    router(config-if)#int f0/0 
    router(config-if)#no shutdown
    router(config-if)#int f0/0.2                                        
    router(config-if)#encapsulation dot1q 2                               
    router(config-if)#ip address 172.16.2.254 255.255.255.0          
    router(config-if)#int f0/0.3
    router(config-if)#encapsulation dot1q 3
    router(config-if)#ip address 172.16.3.254 255.255.255.0              
    router(config-if)#int f0/0.4
    router(config-if)#encapsulation dot1q 4 
    router(config-if)#ip address 172.16.4.254 255.255.255.0           
    router(config-if)#exit
    router(config)#router rip                                            
    router(config-rip)#network 172.16.0.0                           
    router(config-rip)#network 192.168.1.0
    router(config-rip)#version 2                                      
    router(config-rip)#no auto-summary                                

**5.Router 1设置ip地址、RIP协议**

    router>en
    router#conf t
    router(config)#int s0/2/1
    router(config-if)#ip address 192.168.1.2 255.255.255.0
    router(config-if)#clock rate 64000                                    
    router(config-if)#no shutdown
    router(config-if)#int f0/0
    router(config-if)#ip address 172.16.8.254 255.255.255.0
    router(config-if)#no shutdown
    router(config-if)#int f0/1
    router(config-if)#ip address 172.16.7.254 255.255.255.0
    router(config-if)#no shutdown
    router(config-if)#int s0/2/0
    router(config-if)#ip address 192.168.2.1 255.255.255.0
    router(config-if)#no shutdown
    router(config-if)#exit
    router(config)#router rip
    router(config-rip)#network 192.168.1.0
    router(config-rip)#network 192.168.2.0
    router(config-rip)#network 172.16.8.0
    router(config-rip)#network 172.16.0.0
    router(config-rip)#version 2
    router(config-rip)#no auto-summary

**6.Router 2配置IP地址、RIP**

    router>en
    router#conf t
    router(config)#int f0/1
    router(config-if)#ip address 172.16.6.254 255.255.255.0
    router(config-if)#no shutdown
    router(config-if)#int f0/0
    router(config-if)#ip address 172.16.5.254 255.255.255.0
    router(config-if)#no shutdown
    router(config-if)#int s0/2/0
    router(config-if)#ip address 192.168.2.2 255.255.255.0
    router(config-if)#clock rate 64000
    router(config-if)#no shutdown
    router(config-if)#exit
    router(config)#router rip
    router(config-rip)#network 172.16.0.0
    router(config-rip)#network 192.168.2.0
    router(config-rip)#version 2
    router(config-rip)#no auto-summary

配置完成，远程管理计算机ping VLAN 2 PC1测试  

![](http://songtl.com/wp-content/uploads/2012/06/截图-2012年06月13日-09时19分49秒.png)
