---
comments: false
date: 2012-06-08 12:09:32
layout: post
published: true
slug: disappointing-aliyun
title: 令人失望的阿里云
---

阿里云果然烂透了，从用的第一天起，就充满了杯具。

爱范儿迁移到阿里云的第一天早晨9点，阿里云就开始挂了，无奈之下，切换回了日本Linode，看了看，似乎是我自己的配置问题，后来因为人太多，10M小水管不够用就没人可以访问了，于是我们上了又拍云cdn，感觉效果不错，访问速度快了很多，也启用了 cdn.ifanr.cn 的 cdn 域名。

于是，修 PHP，改 Wordpress，又搬了回去，似乎问题没有了，团队成员也都很开心，速度变快了----除了我，成天被 GFW RESET…

某天，发现 PHP 确实有问题，于是让 PHP 吐个核看看，写到磁盘里。我们用了 APC 做 Object Cache，512M 的 shm，当然这些也会被 dump 出来，8个 PHP-FPM 进程一共是 4GB。写 core 不要紧，一看才知道 IO 居然这么差，写 core 的时候 IOWAIT 居然到了 100%，愣是挂了半个小时，这半个小时 ssh 都登陆不进去，后来从 collectd 里看到，居然 loadavg 到了三十几，所有的 CPU 全都在等 IO，所有的 read() 全都被 block 住了--你妹！

我有个坏毛病，没事儿爱输入 sync 玩，某次输入一看，怎么这么久！
    
    [L:root@ifanr-cn] ~# time sync
    sync  0.00s user 0.02s system 0% cpu 4.927 total

于是，跑个 hdparm 看了看，一看不要紧----
    
  [L:root@ifanr-cn] ~# hdparm -t /dev/xvdb1
  /dev/xvdb1:
  Timing buffered disk reads:  74 MB in  3.11 seconds =  23.76 MB/se`

那么，我们来 iostat -x 看看吧

    avg-cpu:  %user   %nice %system %iowait  %steal   %idle
               1.84    0.00    0.63   35.53    0.00   62.00
        Device:         rrqm/s   wrqm/s     r/s     w/s   rsec/s   wsec/s avgrq-sz avgqu-sz   await  svctm  %util
        xvda              0.00   810.00    0.00  202.50     0.00 16136.00    79.68   143.56  719.75   4.94 100.00
        xvdb              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00   0.00   0.00
        scd0              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00   0.00   0.00
        
        avg-cpu:  %user   %nice %system %iowait  %steal   %idle
                   1.36    0.00    0.49   28.39    0.06   69.70
        
        Device:         rrqm/s   wrqm/s     r/s     w/s   rsec/s   wsec/s avgrq-sz avgqu-sz   await  svctm  %util
        xvda              0.00   748.00    0.00  198.00     0.00 15776.00    79.68   143.96  746.70   5.05 100.00
        xvdb              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00   0.00   0.00
        scd0              0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00   0.00   0.00

上面的数据是正常数据哦亲！

无奈之下，[发了条微博](http://www.weibo.com/1039929634/ynqS5zk99)，一看回复，原来大家都有这毛病！

本来觉得阿里还算是个正儿八经的公司，现在一看，连个盘柜都不舍得买么？！就连我校的好多年前买来的日立光纤存储都能到 400MB/s 好不好（虽然有 RAID）！

吐槽完毕，反正阿里云和IBM一样，以忽悠为主。
