---
comments: true
date: 2010-06-18 20:16:30
layout: post
published: true
slug: ibm-server-sucks
title: IBM服务器——谁用谁知道
---

用电脑这么多年，还真没见过这样的电脑，更别说服务器了。

![](http://difan.org.cn/BlogIMG/x255ov1.gif)

这是一台IBM eServer xSeries 255，1个4核Xeon MP，7个U的大家伙。有个Adaptec AIC-7899 SCSI控制器，装了12块300GB硬盘，两个光卡一个电卡，电卡似乎还是Broadcom 5703的破卡。内存呢，标配了4×256MB DDR1，后来买了8×1GB DDR1内存，总共应该是9GB----插上确实是9GB，然后过两天一看，成8GB了，再过两天，4GB，然后呢，2GB，最后呢，1GB。

于是，拔内存。似乎确实解决了一阵子事情，能稳定在4GB了，然后呢……

<!-- more -->

我们就用它做3P下载站的服务器了。青岛大学内网用户访问http://dl.osqdu.org/可以访问3P下载站。似乎开始的时候一切正常。拆下了RAID卡。用Solaris嘛，当然做了RAIDZ。杯具就从此开始了。大家都知道zfs吃内存的，于是机器就三天两头重启。每次打开KVM，总显示着一大堆ZFS相关的错误。于是，不得已，转移数据，折腾了好几个移动硬盘和服务器，终于将大约2TB的数据折腾出来了。然后，配上了UFS，毕竟UFS这么多年了，肯定没问题的，接受了时间的考验。BTW，fat能用这么长时间纯属意外，当然微软最后偷了点VMS 70年代的技术，搞出来一个"New Technology File System" aka NTFS....

随着数据的不断增多，有时候我在另外的几台服务器上下载完文件，用sftp传进去的时候，会发现突然就stall了。可以ping，可以ssh，但死活无法看到终端。跑到KVM上，输入用户名，按回车以后，可以看到键盘回显，但是不管怎样都没法看到Password: 提示符，更不要说是登陆了。因为不懂怎么调试Solaris，只好……后来就启动了KMDB，直接F1-A（或者是Stop-A），::quit重启了。

接着呢，我接管了服务器。这一天注定是杯具开始的一天。几乎每天，我都要登陆到KVM里，输入F1-A，敲::quit回车y回车，等着启动起来。每次的错误似乎都不一样。下面是两张示例，告诉大家都是怎么死机的。

![Dead 1](http://difan.org.cn/BlogIMG/dead1.png)

![Dead 2](http://difan.org.cn/BlogIMG/dead2.png)

![Dead 3](http://difan.org.cn/BlogIMG/dead3.jpeg)

大概每次的死机都会不一样……

因此，我由衷的感叹……

X86是不稳定，但这么不稳定的还真没见过，特别这还是服务器！

所以呢……

有预算买服务器的，还是不要买IBM的了吧。

BTW, 青岛大学的别的IBM服务器也不怎么样，今天去机房给这个可怜的机器换内存的时候，发现老师又在折腾另外某台可怜的IBM。看到可以运行好多好多年的Sun服务器，Sun Fire V880，还有Enterprise 250……

都是UNIX厂商，差距咋就这么大呢？
