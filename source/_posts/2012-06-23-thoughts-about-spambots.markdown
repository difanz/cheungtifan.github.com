---
comments: false
date: 2012-06-23 06:21:33
layout: post
published: true
slug: thoughts-about-spambots
title: 抓恶意爬虫以及防止恶意爬虫的一点想法
---

爬虫很讨厌。

为什么这么讨厌爬虫呢？是因为这玩意儿会让我的缓存全部失效。你想，平时大部分用户都是访问网站首页的几篇文章而已，突然来个爬虫，爬掉你的整个网站。例如爱范儿，几千篇文章，一篇一篇的爬下去，吃 CPU 不说，爬到后面，缓存干脆命中不到，只好去查询 MySQL，查询是要费时间的，也是要跑 TCP 的， overhead 很大的亲。如果爬的频率一高，机器说不定就会宕机了。

为了防止爬虫，我的工作也没少做。一是按时分析 HTTP access log，看什么不正常的东西，二是看 collectd，看图形有什么不正常的，三就是封不对的 User-agent。下面是 nginx 封掉的一些讨厌的 bot:

    if ($http_user_agent ~ "Rome") {
        return 403;
    }
    if ($http_user_agent = "Mozilla/4.0") {
        return 444;
    }
    if ($http_user_agent ~ "linkcrawler") {
        return 403;
    }
    if ($http_user_agent ~ "larbin") {
        return 403;
    }

另外，今天曝光网宿科技的一个 bot 121.9.213.36, user-agent: "Mozilla/8.0 (compatible; MSIE 8.0; Windows 7" (自己还少了一个括号)。

但是上面的都是事后诸葛亮，出事儿的时候根本没用。于是，想到了一点也许可以解决 spambot 的方法。

首先，我们应该分析 spambot 的行为。让我们思考，bot 的作者会怎么写一个 bot 呢？

Approach I: 输入一个页面，分析页面内所有链接，加入链接到一个 hashmap 中，爬过的写一个标志位，没爬过的分发到爬虫线程。

Approach II: 输入一个 URL pattern，直接生成待爬取页面列表，爬。

不管是哪种 approach，他们的目的都是获取网页内容。而这和普通用户有什么区别呢？

区别 I: Bot 不大可能请求 CSS 等资源，而正常用户会。

区别 II: Bot 不大可能去执行 JavaScript (当然现在也有 headless 的基于浏览器的 bot)

区别 III: Bot 的访问趋势是不同于人的，人的正常浏览习惯是在首页浏览，看到感兴趣的，打开一篇，看完继续翻页，而机器则不是这样子的

有这三个 major 区别，我们很容易在 Web 服务器上阻断 spambot。当然，如果使用 headless 的浏览器，区别 I 就会被干掉。区别 II 仍然有效，待会儿说。

对于简单的 bot，我们可以使用 nginx 外加模块，连接 memcached 计数的办法干掉。比如某个 ip 访问页面，计数器加 5，访问 CSS/JS 则减 1，半小时以后 invalidate。一般的用户很快就会被清到 0，因为一般页面上 CSS/JS 数量会远远超过 5 per page，何况我们还有图片。

对于简单的 bot，我们还可以使用 JavaScript 请求 AJAX，访问某个页面，这个页面将 memcached 中该 ip 的记录标志为可信。

对于 headless browser 类型的 bot，我们还是有办法解决的。

  1. 创建浏览器对象需要内存。他们会希望页面加载结束后，立即抓取内容，加载下一个页面，我们可以使用 JavaScript 延时 AJAX 的方法标志
  2. headless browser 也希望尽快的抓取内容，不妨对人的访问速度进行 profile，如果某用户的 pattern 不符合 profile，则标记为可疑

当然，bot 也是有好有坏的，比如我们希望 Google bot 等尽快抓取，因此可能还需要实现某个白名单（注意 User-agent 不代表一切，还有 source IP）。这样一来，大部分 spambot 就会没办法抓取。至于某些花了很大精力抓取内容的 bot，我只想说，最近的文章可以用 RSS 获取全文，您何苦呢？！

最后，对抗 bot 的战争是永远不会结束的，上面仅仅是对可能的反 bot 方法进行初步讨论。遇到疑似 bot 的访问时，不妨提供 recaptcha 让它填写。另外，一些人工智能算法也有可能可以 apply 进来。但是，我们要牢记，我们的目的只是挡住大部分 bot，不影响站点的正常工作而已。After all，如果 launch 了一大把分布的 bot，不断的抓取，这种行为应该视为 DDoS 的一种。对于 DDoS，目前没有什么太好的办法，因此我也就不继续讨论了。

本文的部分研究可以在我之前的一篇论文中找到。[IEEE Xplore](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=6127462)

[A Distributed Network-Sensor Based Intrusion Detection Framework in Enterprise Networks](http://difan.org.cn/BlogIMG/Botdetection.pdf), Difan Zhang, Wei Yu, and Rommie Hardy. Free Local Copy.
