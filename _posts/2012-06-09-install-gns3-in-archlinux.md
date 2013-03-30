---
layout: post
title: ArchLinux安装GNS3笔记
categories: Linux
tags: [GNS3,Arch,Linux]
---

因为在路由与交换的学习上经常要做实验，所以CISCO模拟器是必不可少的。当然模拟器也有很多个选择，比如CISCO自家出的Packettracer、直接运行CISO系统的Dynamips，以及基于Dynamips的DynamipsGUI、GNS3。Packettracer、Dyanmips、GNS3都是跨平台的！以前一直使用的是Packettracer，对于初学者来说非常不错，不过在Linux平台上不太友好，使用软件自带的Lib库字体根本没法看，使用系统的Lib库后字体倒正常，不过工作无法保存。最新版Packettracer 5.3.3自带Lib库字体已经可以看清楚，不过离养眼还有些差距。再者很多命令跟不上真机，所以决定以后路由与交换的实验还是转到GNS3上来。

网上基于Linux的GNS3的安装教程倒不少，不过基本上都是Ubuntu、Fedora、Deepinlinux的，在Archlinux上安装的真没找到，万能的WIKI也没有记录。安装过程走了不少弯路。

GNS3在aur的源里，所以通过yaourt安装。由于GNS3是基于Dynamips的，所以有如下依赖关系

    Dynamips &gt;&gt; Dynagen  &gt;&gt; GNS3

好在yaourt会自动解决依赖关系，所以直接安装GNS3就可以

    yaourt -S GNS3

中间的交互过程，一路no，到Continue install？的时候yes，检查依赖环境会自动安装Dynagen，交互时继续no，continue install？的时候yes，再次检查依赖环境会自动安装Dynamips，下载源码安装，到Starting build()这步时出现错误

    /tmp/yaourt-tmp-root/aur-dynamips/./PKGBUILD: line 27: patch: command not found

原因是要用到patch这个命令，但系统没安装这个包，所以报错了。解决方法

    sudo pacman -S patch

安装好后重新yaourt安装就一路到底了,不过不知道为什么速度很慢，只有2~4 Kb/s，下载了很久。安装好之后启动GNS3

    sudo gns3

**2013/02/14 更新：**

关于右键菜单中没有图标的问题，参见GNS3官方论坛 

解决方法：打开终端，运行一下命令

    gconftool-2 --type Boolean --set /desktop/gnome/interface/menus_have_icons True

重启GNS3后就可以看到右键菜单的图标了。
