---
layout: post
title: ArchLinux安装与配置
categories: Linux
tags: [Arch,Linux]
---

一直使用Uubuntu10.04LTS,不过很多地方总是不合心意。正好Ubuntu在4月26发布了12.04LTS版本，于是打算升级上去看看10.04下的问题有没有改善。升级之后发现Unity 3D居然花屏，2D正常，10.04存在的问题依然没有得到好的解决。于是转投ArchLinux，安装过程坎坷，记下备忘。

正好买了U盘，选择U盘安装。插入U盘，在没有挂载的情况下制作启动盘

    dd if=/media/Others/archlinux-2011.12.core-i686.iso of=/dev/sdb

/media/Others/archlinux-2011.12.core.iso 是我下载的镜像存放位置，/dev/sdb 是我U盘在电脑里显示的盘符，可通过fdisk -l查询  
完成后重启，进BOIS里面设置U盘启动，由于我的U盘是量产成CD-ROM的，通过dd制作的是USB-HDD的，所以我的U盘启动有CD-ROM和USB-HDD两种选择，这里应该选USB-HDD启动，保存退出。  
现在应该可以进到ArchLinux的grub引导界面了，选择第一项Boot Arch Linux ,系统会启动到控制台界面，进入安装界面的方法

    /arch/setup

接下来是基本系统的安装，网上教程已经很多且很详细。我在安装的时候遇到两个问题

1.到分区这一步的时候选手动分区，cfdisk完成之后保存提示re-read错误，然后设置挂载点的时候读不到我新建的分区。是因为内核没有读取到新建的分区，重启解决。理论上Alt F2切换到tty2，登录root执行下面命令可以强制系统内核重读分区表信息

    partprobe

2.到最后安装grub开机引导到MBR总是出错，切换到tty7显示error 12，我在虚拟机里面安装也是一样问题。只能切换到tty2，通过grub命令安装

    grub
    root (hd0,0)
    setup (hd0)

或者

    grub-install /dev/sda

我的/boot是单独分区的，如果你只分了/分区，则命令如下

    grub-install --root-directory=/boot /dev/sda

安装完成后重启系统

    reboot

因为我在配置系统这一步是直接跳过选择DONE的，安装完基本系统后再配置，首先是网络，我的是有线连路由器上网，DHCP方式获取IP，编辑/etc/rc.conf

    vim /etc/rc.conf

找到

    interface=

改成

    interface=eth0

保存退出，获取IP

    dhcpcd

然后测试网络情况

    ping google.com

OK,有返回值网络已经通了，接下来修改软件源，编辑/etc/pacman.d/mirrorlist

    vim /etc/pacman.d/mirrorlist

找到#China，把下面的几个网址取消注释，如下

    ## China
    Server = http://mirrors.163.com/archlinux/$repo/os/$arch
    Server = http://mirror.bjtu.edu.cn/archlinux/$repo/os/$arch
    Server = http://mirror6.bjtu.edu.cn/archlinux/$repo/os/$arch
    Server = ftp://mirror.lzu.edu.cn/archlinux/$repo/os/$arch
    Server = http://mirror.lzu.edu.cn/archlinux/$repo/os/$arch
    Server = ftp://mirrors.stuhome.net/archlinux/$repo/os/$arch
    Server = http://mirrors.stuhome.net/archlinux/$repo/os/$arch

保存退出后更新源列表

    pacman -Syy

然后更新包管理软件pacman，要不然以后使用pacman时总提示你更新

    pacman -S pacman

提示错误的话加上参数f强制更新

    pacman -Sf pacman

接下来更新整个系统

    pacman -Syu

期间会下载一百多M的文件，幸好速度还比较快。完成后新建个普通用户了，使用root太危险

    adduser song

接下来要求输入一些信息，直接回车，到additional grounp的时候输入

    lp,audio,video,power,scanner,games,wheel,storage

意思是这个用户属于那些组，其中video audio wheel很重要，决定你能不能使用声音 视频 sudo权限，接下来继续回车，到后面设定密码就完了。接下来让普通用户可以使用Sudo，如果你安装基本系统时没选sudo这个包，那么现在安装

    pacman -S sudo

    visudo

系统会打开/etc/sudoers文件，找到

    #%wheel	ALL=(ALL) ALL PASSWD=NO

取消注释，变成

    %wheel	ALL=(ALL) ALL PASSWD=NO

保存退出，以后普通用户就可以使用sudo暂时取得root权限，并且无须密码。Ubuntu里面经常要输入密码很麻烦，当然如果你想要用密码验证的话取消注释这行代替上面的

    #%wheel	ALL=(ALL) ALL

注销root用户，以song用户登录安装图形界面X，先安装基础组件Xorg

    sudo pacman -S xorg-server xorg-xinit xorg-uitls xorg-server-utils

安装mesa使用3D效果

    sudo pacman -S mesa mesa-demos

安装DBUS

    sudo pacman -S dbus

设置dbus开机启动，修改/etc/rc.conf

    vim /etc/rc.conf

在DAEMONS数组最后加入dbus

    DAEMONS=(syslog-ng network crond dbus)

接着安装显卡驱动(NVIDIA)

    sudo pacman -S xf86-video-nouveau nouveau-dri

接下来是声卡

    sudo pacman -S alsa-utils

安装测试图形界面的软件

    sudo pacman -S xorg-twm xorg-xclock xterm

完成后，测试图形界面X能否正常运行

    startx

看到几个终端窗口和一个小时钟，说明图形界面X可以正常运行，安装桌面环境XFCE

    sudo pacman -S xfce4 xfce4-goodies

在家目录新建.xinitrc文件

    vim ~/.xinitrc

输入下面内容

    exec ck-launch-session startxfce4

保存退出后执行startx就能进入xfce4桌面了，接下来安装几个常用软件

    sudo pacman -S firefox unrar unzip p7zip flashplugin

打开firefox浏览器进优酷选个视频看看，没有声音，设置一下

    alsamixer

上下键调节数值，没打开的按M键打开，把alsa加入到DAEMONS数组，要不然每次开机都要设置

    DAEMONS=(syslog-ng network crond dbus alsa )

网页里得中文字体显示框框，安装中文字体,喜欢ubuntu的文泉字体

    sudo pacman -S wqy-zenhei

安装中文输入法ibus

    sudo pacman -S ibus ibus-pinyin

配置输入法，家目录新建.bashrc文件,输入下面内容

    export GTK_IM_MODULE=ibus
    export XMODIFIERS=@im=ibus
    export QT_IM_MODULE=ibus

设置桌面环境中文，编辑/etc/locale.gen 取消注释zh开头的4个

    zh_CN.UTF8 UTF-8
    zh_CN.GBK GBK
    zh_CN.GB2312 GB2312
    zh_CN.GB18030 GB1803

让系统更新可用语言

    locale-gen

启动桌面环境使用中文，编辑./xinitrc如下（注意顺序）

    export LANG=zh_CN.UTF-8
    export LC_ALL="zh_CN.UTF-8"
    exec ck-launch-session startxfce4

安装视频播放软件

    sudo pacman -S mplayer gnome-mplayer

安装ntfs-3g解决ntfs分区只读挂载问题

    sudo pacman -S ntfs-3g

一个可用的图形用户界面操作系统基本完成。

总结：确实比Ubuntu爽，原来ubuntu下chrome上淘宝老是崩溃，使用CISCO模拟软件Packettracer卡得要命，视频解码总感觉比不上windows，开机巨慢，怎么重装没用，估计对显卡（64MB独显）支持不好。现在ArchLinux下这些问题全部解决，开机不加载X的情况下10秒完成。就是安装过程复杂了些，ubuntu中文论坛有人埋怨说ubuntu是座空房子，flash插件什么都要自己装，底下有人回复说如果ubuntu是座空房子，ArchLinux就是堆砖头，自己慢慢砌！说得还挺在理。那Gentoo是什么？躺着也中枪
