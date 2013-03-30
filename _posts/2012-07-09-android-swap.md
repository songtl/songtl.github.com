---
layout: post
title: Android系统Swap分区小疑问
categories: Android
tags: [Android,swap]
---

因为手机内存只有256MB，而Android出自Linux，所以可以使用虚拟内存swap来改善性能，虽然这不是什么新鲜的事情，不过今天将手机内存重新规划为Cache-20MB   System-180MB    Oem-0MB Data-240MB之后，SD卡的swap分区居然出现了问题，记录如下。。

手机刷了带内存分区功能的T卡升级包（只为重新分区）之后，还原原来备份的系统。在终端里面输入Free发现Swap居然为0，输入df查询已挂载分区，发现应挂载为Swap的分区/dev/block/mmcblk0p2（手机显示）被挂载为了sd-ext分区，而且还可以存储数据。一顿折腾无果。后来终于发现/system/etc/init.d下有个名为05mountsd的shell script，内容如下


    #!/system/bin/sh
    #
    # mount ext partition from sd card
    &nbsp;
    BB="logwrapper busybox";
    &nbsp;
    if &#91; "$SD_EXT_DIRECTORY" = "" &#93;;
    then
        SD_EXT_DIRECTORY=/sd-ext;
    fi;
    &nbsp;
    # find SD Card
    for MMC_NUM in `seq  9`;
    do
        MMC_TYPE=`cat /sys/block/mmcblk$MMC_NUM/device/type`
        if &#91; "$MMC_TYPE" = "SD" &#93;;
        then
            # 2nd partition of sdcard should be the sd-ext if exist
            SD_EXT_PART=/dev/block/mmcblk${MMC_NUM}p2
            break
        fi
    done
    &nbsp;
    if &#91; -b "$SD_EXT_PART" &#93;;
    then
        log -p i -t mountsd "Checking filesystems..";
    &nbsp;
        # fsck the sdcard filesystem first
        if &#91; -x `which e2fsck` &#93;;
        then
            e2fsck -y $SD_EXT_PART
            e2fsk_exitcode=$?
        else
            echo "executable e2fsck not found, assuming no filesystem errors"
            e2fsk_exitcode=
        fi
    &nbsp;
        # set property with exit code in case an error occurs
        setprop cm.e2fsck.errors $e2fsk_exitcode;
        if &#91; "$e2fsk_exitcode" -lt 2 &#93;;
        then
            # mount and set perms
            $BB mount -o noatime,nodiratime,barrier=1 -t ext3 $SD_EXT_PART $SD_EXT_DIRECTORY;
            if &#91; "$?" =  &#93;;
            then
                $BB chown 1000:1000 $SD_EXT_DIRECTORY;
                $BB chmod 771 $SD_EXT_DIRECTORY;
                log -p i -t mountsd "$SD_EXT_DIRECTORY successfully mounted";
            else
                log -p e -t mountsd "Unable to mount filesystem for $SD_EXT_DIRECTORY!";
            fi
        else
            log -p e -t mountsd "Unable to repair filesystem, disabling apps2sd";
        fi
    fi

玩Linux的应该知道/etc/init.d目录存放的是开机启动的服务，这个shell script的作用在第三行里的注释已经说明了“# mount ext partition from sd card”，意思是从SD卡挂载sd-ext分区，这就是把该挂载为swap的分区挂载为sd-ext的罪魁祸首。仔细看18、19行，18行注释说只要SD卡存在第二个分区就认为是sd-ext分区，一般人SD卡分区通常都会分三个区，即FAT（正常存储）、sd-ext、swap，系统识别顺序应该分别为mmcblk0p1、mmcblk0p2、mmcblk0p3，因为我系统的Data区已经基本够用，所以就没分sd-ext区，因此swap就被系统识别为了mmcblk0p2，而上面的脚本的第19行直接把mmcblk0p2识别为sd-ext分区了，第43行直接就把mmcblk0p2挂载成了ext3格式的sd-ext分区了。找到原因就容易解决了，直接把这个脚本移出/system/etc/init.d就行了。

不过让我郁闷的是我当初给SD卡划分Swap分区时确切指明为Linux swap / Solaris格式，怎么就能被重新挂载为ext3格式呢?另，Android的分区识别确实有点死板，当初在Linux下给SD卡分区的时候把swap分区放在前面，FAT分区放在后面，手机倒是能识别，不过进入recovery刷机模式的时候就不能识别，总是mount SDcard error！原来也是死认SDcard第一分区为存储区，弄了很久都不明原因。最后在XDA论坛才知道真正的原因。
