---
comments: false
date: 2009-11-07 23:42:04
layout: post
published: false
slug: htc-dream-bricked-spl
title: 刷SPL致死的T-Mobile G1怎样恢复加速传感器
---

手贱不可怕，怕的是手贱以后没有办法挽救。

八月份我买了一台T-Mobile G1，当然看中的是Linux操作系统和强大的硬件配置，没想到问题也不是太少，总体来讲不太适合生产用手机，倒是适合折腾。

于是我在9月底折腾了刷SPL，当然radio没有刷成功，否则也就没有本文了。于是华丽的有了一块独一无二的砖头--启动不起来了。没办法，我可不要一块两千块钱的砖头，发深圳修吧。

260大元换了一个NAND芯片（业内人士成为_字库_），说的是加速度感应器没问题。拿到后确实没问题，但是固件是HiAPK.com的固件，我不是太喜欢这个固件，主要是慢，命令行工具少。选用了cynaogen mod 4.0.2，感觉比较稳定，以及他的recovery 1.4。刷了以后，加速度感应没有了。打电话给JS，JS说是不是刷机了，告诉我需要重新发去"做数据"。

我觉得不对头，反正硬件没坏，软件问题我这个半路出家的玩嵌入式Linux怎么也能搞定了吧，于是开始去xda-developer翻帖子。一个月的时间，中间还有个GRE(这也是我好久不写Blog的原因)，足够让我查到相关资料。

查资料的结果就是/system/bin/akmd是传感器的daemon process(还有个用户，compass)；/data/misc/akmd\_set.txt是akmd的配置文件，可以删除这个文件重新校准。但是我这里并没有这个文件，也许是因为换NAND的原因吧。说句题外话，我觉得根本不用换NAND Flash ROM的，重新插到编程器上烧录就好了吧。。。该死的Death SPL。nandroid显示的是一串0而不是以前的HT什么东西。估计是兼容性问题还是什么的，但是固件在那里放着的，JS确实可以"做数据"。我不甘心花来回40的顺丰快递换一个文件，于是决定自己折腾折腾。

反正硬件都是一样的，软件问题啦。最先怀疑的就是akmd，看见有些帖子说可以换老版本的akmd，抓紧去找了个HTC的1.1的ROM，提取出来了akmd，在console里替换掉，运行GPS Status果然可以正确的指南针了。考虑到众多G1用户有这样的问题，我把它发来了自己的服务器上。点[这个链接](http://www.huaxueba.com/alex/akmd.gz)就可以下载了。

文件信息：

    akmd.gz 48358 bytes

    akmd
    md5sum  517a87a4e6caa5e66f0520c68dcb7c0e   
    sha1sum b741df4aa075115192fe2659c472cd1f6bcd84a0<

下载回来文件，用gzip一类的文件（不见得你是用UNIX/Linux的，7-zip也不错）解压，拷贝到sd卡第一个分区。在console下（Home+Power开机，用cm-1.4的recovery选择Console）执行下面的命令：

    mount -a
    cd /system/bin
    cp ./akmd /sdcard/akmd_old  
    cp /sdcard/akmd /system/bin  
    tar cf /sdcard/misc.tar /data/misc/  
    rm /data/misc/akmd_set.txt  
    rm /data/misc/rild*  
    reboot

试试你的加速度感应器吧！

关键字：

Android Google G1 换字库 重力感应 SPL刷死 akmd
