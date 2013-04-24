---
layout: post
title: "cisco内部模拟器IOU"
description: ""
category: Network
tags: [cisco, 模拟器, IOU]
---
{% include JB/setup %}

以前做思科的实验一般都是用GNS3这个模拟器，主要缺点是太耗资源，计算好IDLE值开多几台路由器还是有些卡。前段时间研究了一下IOU这个CISCO内部使用的模拟器，感觉还不错。

现在用的版本是从[flyxj](flyxj.cn)的[Cisco L2/L3 IOU Rack V3.0](http://flyxj.cn/archives/cisco-l2-l3-iou-rack-v3)移植过来的，因为原来的版本是运行在虚拟机的Debian系统里面，也就是说做实验之前一定要打开虚拟机，感觉太麻烦，并且自己本身使用的也是Linux系统，干脆就把相关的文件移植过来直接运行在Arch Linux上。

拓扑图也根据自己的需要重新搭建,一共18台设备，其中12台路由器（一台模拟帧中继）,6台交换机

![iou](http://pic.yupoo.com/songtl/CLzxwLXT/medish.jpg)

全部设备开起来资源占用很少，CPU使用率也低,做了几个实验，感觉挺好。而且有了万能拓扑，做实验时直接开设备telnet上去，省去了GNS3里面搭拓扑,加模块的时间，非常方便！

现在可能比较多人用IOU Web Interface这个改进版，虽然可以通过浏览器自己搭拓扑,不过也意味着要开Httpd等更多的进程,而我是极力追求极简的，因此还是喜欢原版，一个万能拓扑基本上满足我现在的实验需求,实在不行可以自己再另外搭拓扑，毕竟IOU的拓扑搭建相对Dynamips来说很简单。
