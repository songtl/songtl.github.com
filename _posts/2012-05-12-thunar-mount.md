---
layout: post
title: 解决thunar挂载分区无权限
categories: Linux
tags: [Arch, Linux, thunar, 挂载, 权限]
---

今天把ArchLinux滚动更新了，更新完之后发现在Thunar侧边栏直接点击图标无法挂载分区了，提示

    Not authorized to perform operation.

手动mount没有问题，看到Error总是不爽，查了一下发现是Udisk2的配置问题，解决方法如下：


编辑`/usr/share/polkit-1/actions/org.freedesktop.udisks2.policy`这个文件，找到



    <action id="org.freedesktop.udisks2.filesystem-mount-system">
            省略
            <allow_active>auth_admin_keep</allow_active>
            省略
    </action>
    
把其中的auth_admin_keep改成yes,即

    <action id="org.freedesktop.udisks2.filesystem-mount-system">
            省略
            <allow_active>yes</allow_active>
            省略
    </action>



保存之后再挂载，应该就没问题了。
