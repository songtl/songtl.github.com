---
author: song
layout: post
title: 解决Fcitx某些GTK2程序无法激活
categories: Linux
tags: [Fcitx, GTK2]
---

最近换成Fcitx输入法后，在英文Locale下某些GTK2程序总是无法通过Ctrl Space正常激活，如GNS3、阿里旺旺、永中Office等，不过切换到中文Locale就能够正常激活，这确实是个非常烦人的问题。

[官方wiki也提到了这个问题：](https://wiki.archlinux.org/index.php/Fcitx#Troubleshooting)

*   Ctrl Space fail to work in GTK2 programs

This problem sometimes happens when locale is set as English. From the official FAQ, simply use:

    # gtk-query-immodules-2.0 > /etc/gtk-2.0/gtk.immodules

then edit gtk.immodules，and change the corresponding line as below:

    "xim" "X Input Method" "gtk20" "/usr/share/locale" "en:ko:ja:th:zh"

按照wiki的方法修改了配置文件后不但不起效果，反而在更多的程序中无法激活Fcitx，而Google出来的解决方法都是照搬wiki的方法修改GTK2配置文件。

不过注意到wiki中有一段关于Fcitx在Emacs中不能输入是这么说的：

*   Emacs

If your LC\_CTYPE is English, you may not be able to use input method in emacs due to a old emacs’ bug. You can set your LC\_CTYPE to something else such as “zh_CN.UTF-8″ before emacs starts to get rid of this problem.

考虑到在中文Local下可以正常激活，有可能跟这个有关，于是将/etc/gtk-2.0/gtk.immodules文件还原成修改前，在～/.xinitrc中加上一句

    export LC_CTYPE=zh.CN.UTF-8

重新启动X，问题解决。 由此看来官方wiki写得也不是特别严谨，不过问题总算解决！
