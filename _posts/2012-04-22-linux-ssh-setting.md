---
layout: post
title: Linux实现SSH远程登录
categories: Linux
tags: [Arch, ssh, ubuntu]
---

远程登录方式有telnet和ssh两种方式，由于telnet使用的是明文传输，传输过程中系统帐号密码等重要信息容易被截获，安全性比不上SSH（secure shell），因此现在一般都使用SSH作为远程登录的工具。

其实很多linux版本如ubuntu已经内置了ssh-agent，这是一个远程连接的工具,通过ssh-agent可以发起远程连接，但是如果你要想实现在其他电脑远程登录自己的电脑，就必须安装openssh，可以通过以下命令  
ubuntu

    sudo apt-get install openssh

ArchLinux

    sudo pacman -S openssh

系统会自动下载并完成安装，完成后可通过以下命令查看ssh服务是否启动

    ps -ef | grep sshd

如果没有sshd这个进程，手动启动  
ubuntu

    sudo /etc/init.d/ssh start

ArchLinux

    sudo /etc/rc.d/sshd start

ArchLinux把sshd添加到DAEMONS数组开机启动

    DAEMONS=(syslog-ng network crond dbus alsa @openntpd sshd)

如果出现sshd进程，表示ssh服务已经启动，此时不出意外你在其他电脑上就可以通过ssh连接到自己的电脑上面了。关于ssh连接的工具，windows平台上面推荐Secure-CRT软件，这是一个非常受欢迎的软件，使用也比较简单。也可以选择Putty，不过建议下载官方英文版，前段时间汉化版的putty有后门的事在网上炒得沸沸扬扬。Linux系统因为已经自带ssh-agent所以比较方便，直接在terminal里面输入以下命令

    ssh username@ip

username是你的登录账户，ip即ip地址，当然你也可以使用域名

    ssh username@domain

此时系统会要求你输入密码进行验证，验证通过就能登录到远程主机.为了安全期间，需要进行一些简单的配置，否则日后你查看ssh日志文件的时候会发现大量ip的登录失败信息。其实是别人通过端口扫描软件扫描出开启来22（ssh默认）端口的主机，然后通过穷举法进行密码猜解，如果你使用的是弱口令，被猜解出的几率是非常高的。

配置信息保存在  
ubuntu

    /etc/ssh/ssh.conf

Archlinux

    /etc/ssh/sshd.config

我们可以通过编辑这个文件来进行配置。当你尝试登录别人的主机的时候你会以什么身份登录？当然是root用户，因为root是每个Linux系统都存在的用户。因此我们应该禁用root用户登录,找到

    PermitRootLogin yes

把yes改为no即可。 

端口扫描软件默认扫描22端口，因此我们也可以把端口改成其他端口，找到以下语句

    port 22

把其中的22改成你其他端口，比如1022之类的。

限制一下最大密码错误次数,3次吧，自己登录基本不会连续错3次，密码错误超过3次拒绝登录

    MaxAuthTries 3

修改之后保存退出,重启ssh服务  
ubuntu

    sudo /etc/init.d/ssh restart

ArchLinux

    sudo /etc/rc.d/sshd restart

注意，修改了端口之后用CLI模式登录时需要声明端口

    ssh -p 1022 username@ip

通过简单的配置后你会发现来自不明ip的失败登录明显的减少。忘了说，ssh登录日志保存在这个文件

    /var/log/auth.log

在Archlinux下这个文件的拥有者为root，群组为log，权限为640，为了方便普通用户查看日志，把用户加入到log组(不推荐others加权限)

    sudo gpasswd -a song log

平时可以cat一下这个文件看看登录记录，当然有时文件会比较长，特别是没禁用root用户之前，往往来自同一个ip的登录失败次数达到数千条，如果你一行行查看得看到什么时候，因此我们只输出root登录失败的记录(虽然禁用root登录，但有人尝试以root身份登录时系统仍会记录）

    grep "Failed password for root" /var/log/auth.log | awk {'print $11'} | uniq -c | sort -rn

系统会列出曾经登录失败的ip并且统计失败次数从高到底排列，当然你也可以输入登录成功的记录

    grep "Accepted password for " /var/log/auth.log

系统会列出曾经登录主机的记录，包括时间、ip地址等

另外，本人使用的是Android的手机，使用的是Linux的内核，所以希望也能从手机远程登录自己的电脑。首先，手机上安装支持ssh连接的软件，Cyanogenmod7版本的系统就自带一个Terminal，简直就是一个微型的linux终端，基本上很多基本的指令都能执行，比如：ls，cd，mount，cat，nano等等。接下来还有一个问题，我使用的是电信的ADSL，通过TP-link路由器pppoe拨号上网，每次拨号获取的ip都不一样，这样是无法ssh登录的，因为你不知道下一次获取的ip地址是多少。于是想到了windows下的花生壳客户端，可以动态解析域名，赶紧到花生壳网站申请了一个免费域名，不过当时还没发布linux的客户端（现在有源码安装），隐约好像记得TP-link有动态DNS功能，登录上去果不其然，在动态DNS的服务提供者选花生壳，账户密码就填花生壳网站的账户密码，登录之后就会把花生壳帐号绑定的域名解析到本地，但是当你通过域名ssh连接的时候发现是连接不上的，因为拨号上网的是路由器，花生壳会把域名解析到你的路由器，而不是你的电脑，因此还要做进一步设置。在TP-link路由器设置页面的转发规则里选DMZ主机，把DMZ状态设置启用，DMZ主机ip地址填你从路由器DHCP服务器获得ip，如果你的路由器比较多人上网可能每次从DHCP获得的ip也是不一样的，所以要把你的主机mac地址和某一个ip地址绑定，也就是把某一个ip地址保留给你的网卡，让你每次获取的都是同一个ip，把这个ip地址填到DMZ主机ip地址里面。再用域名来ssh连接发现已经可以正常链接了。

最后放上几张手机远程登录主机成功后的截图  
welcome to ubuntu  
 ![welcome](http://songtl.com/wp-content/uploads/2012/04/screenshot-1335180998557.png)
root目录
 ![root](http://songtl.com/wp-content/uploads/2012/04/screenshot-1335180450776.png)
ll 查看文件详情
 ![ll](http://songtl.com/wp-content/uploads/2012/04/screenshot-1335180473039.png)
