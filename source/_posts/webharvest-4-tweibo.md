title: 用Web-Harvest抓腾讯微博
date: 2014-03-05 21:31
tags: [programming, Web-Harvest]
categories: programming
---

[Web-Harvest]: http://web-harvest.sourceforge.net "Web-Harvest"

好久没更新博客，水一文。

前阵子要给别人出试题，偶然发现[Web-Harvest](下称WH)这个抓网页的工具，它主要应用xpath和xquery抓网页，内置还定义了一套功能挺多的语法，就出了一道用WH抓微博的题目。

本来想抓新浪微博的，但发现它的微博内容都是js生成的，折腾了一下，还是可以用WH的函数提取出内容，但腾讯微博相对还是简单多了。

题目其中一个内容是用WH抓几页邓紫棋的腾讯微博，排除包含她演唱会广告的和没有图片的微博。

<!-- more -->

其中遇到的坑：

- 因为是包含中文，写入到文件时要用gbk编码

- 在xpath已经用li[@id]获取微博节点，但节点传到xquery里面时竟然还要再一次用li[@id]获取一级

配置文件如下

[[ gist goorockey:9368146 ]]
