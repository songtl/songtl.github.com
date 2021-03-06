---
layout: post
title: 防止无线路由器被蹭网的设置方法
categories: Security
tags: [安全, 无线路由, 蹭网]
---

现在笔记本、手机、平板大多数都有了WIFI功能，所以家里使用无线路由器变得非常普遍。当然，蹭别人网的人也多了。你能想象有些蹭网的人疯狂到什么程度吗？我曾在某无线论坛看到一个网友，花几百块买数个无线路由和网卡，把蹭到的带宽进行叠加，我想说，既然舍得投资几百块，还不如投资到宽带上呢？毕竟现在宽带的价格也降了，4M一年也就五百块左右甚至更便宜，这样自己用得还心安理得。虽然疯狂的人是少数，大多数人都是“文明”蹭网，不过这样的行为还是不招人待见的。今天就来谈谈家用无线路由如何设置才能防止被人蹭网（刷了OpenWRT、DD-WRT、TOMATO等固件的不在讨论范围）。

既然要防止别人蹭网，首先要了解别人是如何蹭的。要蹭网首先要拿到无线密码，难不成你还能直接拿根网线光明正大插到别人的路由器上？破解无线密码主要有2种方法：

**1.抓握手包跑字典**

**2.利用WPS漏洞，穷举PIN码，进而获取密码**


早年破解都是第一种方法，成功率跟密码的复杂度有很大关系，耗时一般比较长。2011年底爆出WPS漏洞，第二种方法成了破解无线密码的利器，只要开启了WPS功能，穷举出PIN（8位数字）码就能拿到密码，8位数字分为前4个，后4个两部分，后4个的最后一个是校验码。所以实际组合只有11000种，对于现在的电脑运行速度来说，破解只需几个钟，使用工具也比较简单:Linux Aircrack-ng Reaver-wps

下面总结一下预防方法（设置后可能要求重启，建议全部设置完再重启）：

##初级

**1.关闭WPS功能（TP-LINK、腾达、水星叫QSS） (强烈建议)**

这个功能是默认开启的，但在平时却几乎不会去使用它。而穷举PIN码是利用了WPS的漏洞，所以只要关闭了WPS功能就能防范PIN码破解。关闭方法：登陆到路由器管理页，点击`QSS安全设置`>>`关闭QSS`，注意要重启路由器才生效。

**2.无线密码使用强口令(强烈建议)**

虽然关闭WPS后不能用PIN码破解，还可能会被抓包跑字典，但只要无线密码要设置成强口令，跑出密码的可能性非常低，且耗时很长。所以无线密码建议12位以上，包含大小写字母、数字、特殊符号，使用WPA-PSK/WPA2-PSK加密,设置方法：`无线设置`>>`无线安全设置`。

**3.设置无线MAC过滤(强烈建议)**

虽然跑出密码的概率低，但不为0。所以还是要防御。把自己手机、平板、笔记本的MAC地址添加到过滤表，然后禁止不在过滤表中的无线设备连接到无线路由。设置方法：把自己的手机、平板、笔记本通过无线连接到路由器，然后进入`无线设置`>>`主机状态`，可以看到MAC地址，一个个复制，然后到 `无线设置`>>`无线MAC地址过滤`把刚才复制的MAC地址一个个添加到过滤表，添加完点`所有条目生效`。在上方选择`开启过滤`，并且过滤规则为`允许列表中生效的MAC地址访问本无线网络`.

完成这三步，基本可以抵御90%的蹭网了。一般设置这3步也够了。如果想要更安全继续往下看

##进阶#

MAC地址过滤的原理是MAC地址是全球唯一的，但MAC地址真的不能改吗？当然不是，现在有很多能够修改MAC地址的方法，比如路由器就有一个克隆MAC地址的功能用来突破上层的MAC地址绑定。所以要进行另外的设置

**4.关闭DHCP并改变局域网网段**

能够通过前3步连接到路由器的，说明做了MAC地址克隆，伪装成合法MAC地址。所以我们关闭DHCP自动分配IP地址功能:`DHCP服务器`>>`DHCP服务`>>`不启用`，就算连接上路由器也分配不到IP地址，自然就上不了网。当然我们自己的电脑就要手动设置IP了。既然我们可以自己设置静态IP，他也可以自己设置静态IP，因为路由器默认的LAN基本都是192.168.1.0/24网段，我们可以把网段改成一些不常见的IP：`网络参数`>>`LAN口设置`>>`IP地址` 比如172.30.1.1,掩码255.255.255.0这样他就难猜到了，只要他设置的IP不是同一个网段就上不了网。

**5.路由器出口设置局域网IP过滤**

万一他运气够好，猜对了网段，我们可以做外网过滤，设置只允许我们自己设置的几个IP通过路由器上网，其他IP（即使同一个网段）不能通过。设置方法：`安全设置`>>`防火墙设置`>>`开启防火墙，开启IP地址过滤`，缺省过滤规则:`凡是不符合已设IP地址过滤规则的数据包，禁止通过本路由器`,然后到 `安全设置`>>`IP地址过滤`中把你自己设置的静态IP地址添加进去（填在局域网IP那栏，而不是广域网IP），并`使所有条目生效`。

**6.修改登陆口令**

虽然做了IP过滤他上不了网，他可能会去登陆管理页面改变设置。默认用户名密码都是admin，所以我们要把默认用户名密码改掉。复杂一些不要紧。`在系统工具`>>`修改登陆口令` 里面可以修改。

做了那么多设置，被破解的几率大概已经很低了。当然不排除被破解的可能，毕竟黑客的才能是无限的..一天24小时都想着怎么破解人家无线蹭网的就不跟他玩了。。
