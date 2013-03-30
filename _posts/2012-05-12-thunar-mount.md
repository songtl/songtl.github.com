---
layout: post
title: 解决thunar挂载分区无权限
categories: Linux
tags: [Arch, Linux, thunar, 挂载, 权限]
---

今天闲来无事更新了一下系统，感觉Archlinux的滚动更新确实是个不错的东西。更新完之后发现在Thunar侧边栏直接点击图标无法挂载分区了，提示

    Not authorized to perform operation.

手动mount则没有问题，虽然不是什么大问题，但对于喜欢折腾的人来说，看到Error总是不爽。把错误信息GOOGLE一下，发现也有不少人出现这个问题，很多帖子说解决方法是编辑文件

    vim /usr/share/polkit-1/actions/org.freedesktop.udisks.policy

找到

    

把其中的

    auth_admin_keep

改成

    yes

不过修改完后发现问题依旧。继续搜索发现有个帖子说是更新系统时把udisks更新到udisks2引起的,查看udisks软件

    pacman -Q | grep udisks

果然有两个版本

    udisks 1.0.4-3
    udisks2 1.94.0-1

看到这里基本可以确定是这个引起的了，上面提到的解决方法是修改udisks的配置，这里我们修改udisks2的配置

    vim /usr/share/polkit-1/actions/org.freedesktop.udisks2.policy

找到

    

把其中的

    auth_admin_keep

改成

    yes

保存之后再挂载，OK了
