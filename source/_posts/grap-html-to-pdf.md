title: 抓网页保存为pdf
date: 2012-10-29
tags: [others, pdf]
categories: others
---

[pdfmyurl]: http://pdfmyrul "pdfmyurl"
[pdftk]: http://www.pdflabs.com/tools/pdftk-the-pdf-toolkit/ "pdftk"

最近在刷题，总想把题目保存下来，这样没网的时候也可以做题，放手机里也可以随时做了。所以就想着把题目抓下来保存为比较通用的pdf了。

一开始想的是先做类似整站下载，把文字和图片都抓下来，然后再做html转pdf。自己写工具抓下来的话，要抓图片、修改网页里面图片的链接，昨天调研了一下，没找到用python有什么方便的办法，就放弃这条路了。

今天突然找到，网上还是有很多直接网页保存为pdf的工具、网站的。最方便、强大的要数[pdfmyurl]了。

<!-- more -->

## pdfmyurl

可以通过http://pdfmyurl.com?url=\<siteurl\> ,来把指定链接的网页保存为pdf，而且是直接返回的，即用wget http://pdfmyurl.com?url=\<siteurl\>就可以直接得到所需的pdf，不用再按什么按钮之类的了。

在官网上可以发现，pdfmyurl可以算是个api服务，可以通过传很多get参数来得到需要的pdf结果。比较常用的有：

- \-\-filename    输出的pdf文件名
- \-\-page-size   页面大小，默认是A4
- \-\-proxy       通过制定的代理访问该页面，--username --password还指定用户和密码
- -b            使得到的pdf有目录、书签、页眉等书的样式，不过目录不咋地

pdfmyurl已经很强大了，但每次只能完成一个网面的pdf，所以还得想办法做pdf的合并。

## pdftk

[pdftk]也是一个pdf方面的神奇，可以完成pdf合并、合并多个pdf指定页、分割、加水印等，而且是跨windows、linux、mac多个平台的。不过我也只用来合并pdf:

    $ pdftk 1.pdf 2.pdf cat output 12.pdf

但弄出来的pdf是没有目录的。

所以更好的办法其实是用更强大的latex。不过latex还有待系统地研究，现在的再写个bash就已经满足我90%的要求了～

---

更新：

1. 今天还是发现了一个用python抓oj的，有空参考一下，自己实现一个。

2. 今天还发现，pdfmyurl似乎对单个ip一定时间内的请求做了限制，超过限制后，请求都会返回一个错误信息的pdf。我就借cjb的tor解决了这个问题（又用tor干了邪恶的事了）。
