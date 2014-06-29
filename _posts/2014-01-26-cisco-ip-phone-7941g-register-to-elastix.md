---
layout: post
title: "Cisco IP Phone 7941G自动注册Elastix"
description: ""
category: "network" 
tags: [Cisco , IP Phone , Elastix , SIP]
---

现有Cisco IP Phone 7941G两台，Cisco 2960交换机一台，需要实现两台IP Phone能自动注册到Elastix服务器上，并且能够互拨分机号进行通话。

##刷SIP固件版本##

由于两台Cisco IP Phone 7941G都是SCCP的固件版本，所以需要先刷成SIP固件，这里刷的是SIP41.8-3-1S版本，大致过程是IP Phone重启进入刷机模式，通过DHCP服务器拿到一个IP地址及TFTP服务器地址，然后自动去TFTP服务器下载固件进行升级。

所以，至少要搭一个DHCP服务器及一个TFTP服务器。

由于用tftpd32做DHCP Server一直没有成功，所以在Cisco 2960交换机上做DHCP Pool为IP Phone分配IP，并通过Option 150为IP Phone 指定TFTP服务器地址

	interface FastEthernet0/1
		description Connect_to_IP_Phone7941G_1
		switchport access vlan 3
		switchport voice vlan 2
		spanning-tree portfast

	interface FastEthernet0/2
		description Connect_to_IP_Phone7941G_2
		switchport access vlan 3
		switchport voice vlan 2
		spanning-tree portfast

	interface FastEthernet0/3
		description Connect_to_Elastix_Server
		switchport access vlan 2
		spanning-tree portfast

	interface Vlan2
		ip address 192.168.2.254 255.255.255.0

	ip dhcp pool  IP_Phone
		network 192.168.2.0 255.255.255.0
		default-router 192.168.2.254 
		option 150 ip 192.168.2.251


TFTP Server这里直接使用Elastix里的in.tftpd，如果你只是要刷SIP固件而不需要注册到Elastix，可以使用tftpd32或者其他TFTP Server。

把下载的SIP固件解压到TFTP Server的根目录，此时TFTP Server根目录有如下文件

	apps41.8-3-0-50.sbn
	cnu41.8-3-0-50.sbn
	cvm41sip.8-3-0-50.sbn
	dsp41.8-3-0-50.sbn
	Jar41sip.8-3-0-50.sbn
	SIP41.8-3-1S.loads
	term41.default.loads
	term61.default.loads

注意`Jar41sip.8-3-0-50.sbn`首字母J是大写，有可能解压出来时是小写的，如果不进行更改，IP Phone请求`Jar41sip.8-3-0-50.sbn`这个文件时TFTP Server会因为不存在这个文件而报错。

然后把`XMLDefault.cnf.xml`这个文件也放到TFTP Server根目录，里面的这个语句告诉IP Phone去读取哪个固件

	<loadInformation115 model="CP-7941">SIP41.8-3-1S</loadInformation115>

如果你的IP Phone型号不同或者你刷的SIP固件版本不一样，注意变更！比如7941G-GE的就需要变成这样

	<loadInformation309 model="Cisco 7941G-GE">SIP41.8-3-1S</loadInformation309>

此时TFTP Server根目录有如下文件

	apps41.8-3-0-50.sbn
	cnu41.8-3-0-50.sbn
	cvm41sip.8-3-0-50.sbn
	dsp41.8-3-0-50.sbn
	Jar41sip.8-3-0-50.sbn
	SIP41.8-3-1S.loads
	term41.default.loads
	term61.default.loads
	XMLDefault.cnf.xml

然后就可以开始升级了，把IP Phone断电，按住#号键通电，直到显示屏右边的Line信号灯开始闪烁时松开#号键，并依次输入123456789*0#进入刷机模式，然后IP Phone会自动拿到一个DHCP Server分配的IP地址，并自动去Option 150指定的TFTP Server下载固件进行升级，期间可能会多次重启！

具体IP Phone到TFTP Server里下载了什么文件，可以在刷固件之前开启TFTP日志进行查看

	tailf /var/log/message

刷完后进入系统菜单中的 `Status` > `Firmware Version`查看，如果有显示SIP字样说明已经刷成功了。

##自动注册Elastix##

首先需要在Elastix里面建立Extension（分机号），需要注意的一点是分机号设置里的`nat`和`quality`这两个选项都要设置为no，否则IP Phone可能无法成功注册。

然后需要将`SEPxxxxxxxxxxxxx.cnf.xml`这个文件放到TFTP根目录，xxx需要换成你IP Phone的MAC地址（都是大写），比如我的就是`SEP001E4A5FBF14.cnf.xml` ，文件里面遇到`_username_`或`_password_`等这种样式的地方说明要根据自己的分机号参数进行替换。

接着将`dialplan.xml`这个文件放到TFTP根目录。此时TFTP Server包含的文件如下

	apps41.8-3-0-50.sbn
	cnu41.8-3-0-50.sbn
	dsp41.8-3-0-50.sbn
	SIP41.8-3-1S.loads
	term41.default.loads
	term61.default.loads
	XMLDefault.cnf.xml
	SEP001E4A5FBF14.cnf.xml
	SEP001E4A5FBDBC.cnf.xml
	dialplan.xml

最后，IP Phone进入设置菜单，输入\*\*#\*\*重启，IP Phone会到TFTP Server里读取配置文件（SEPxxxxxxxxxxxxx.cnf.xml），根据配置文件里的分机号信息自动注册到Elastix服务器。

**注意：**

*	如果`SEPxxxxxxxxxxxxx.cnf.xml` 文件里面的配置有错，或者Elastix上分机号设置里的的`nat`和`quality`选项没有设置为no，可能IP Phone屏幕右上角会显示号码，但一直在Registering，此时需要检查一下配置文件和分机号配置是否正确。

*	如果缺少`diaplan.xml`这个文件，可能IP Phone能够接电话，但无法拨出电话。

最后附上[相关的配置文件](http://pan.baidu.com/s/1Bx4BG)供参考，须知刷机有风险，变砖不负责，祝好运！

**参考资料：**

 [1. Reflash your Cisco 7940, 7941, 7960 or 7961 phone to SIP]( http://greenwirecommunications.com/phone-systems/cisco-ip-phones/reflash-cisco-7940-7941-7960-7961-phone-sip/)

 [2. Elastix @ Cisco 7941 7970 @ SIP](http://holisticware.net/holisticware/know-how/system-integration/ip-telephony-voip/equipment/ip-phones/cisco-linksys/elastix--cisco-7941-7970--sip)

 [3. Setup Cisco 7941 or 7961 with Asterisk]( http://www.selbytech.com/2009/10/setup-cisco-7941-or-7961-with-asterisk)
