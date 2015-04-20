title: 用nodejs实现Instapaper回顾邮件自动发送
date: 2014-05-12
tags: [Instapaper, nodejs, email]
categories: programming
---

[Instapaper]: http://www.instapaper.com "Instapaper"
[Appfog]: http://www.appfog.com "Appfog"
[SendGrid]: http://sendgrid.com "SendGrid"

最近在搞nodejs，刚好有个点子，想实现一个对自己这个月在[Instapaper]的已读文章做回顾，通过邮件形式发到自己的邮箱，起到复习的作用。然后上周末就用了两个通宵把这搞出来了，代码放在[Github](https://github.com/goorockey/instapaper-review)上面。

###总的情况

- 整个服务跑在[Appfog]上
- 用[SendGrid]服务发送邮件
- nodejs实现, 用到了cheerio、request、cron、sugar、ejs、sendgrid等几个模块

<!--more-->

###抓文章

Instapaper提供有API，可以获取到已读文章，但它需要填表、人工审核，感觉不容易通过，索性就自己模拟登录，抓下来算了。F12看了一下，发现Instapaper通信协议极其简单，好抓得很。

nodejs抓网页，可以用cheerio，说比传统的JSDOM要快、要方便，用着确实方便。

###Appfog

Appfog是我挺喜欢的一个PaaS，它是基于Cloudary提供服务的，支持语言多，部署简单。

这次因为要定时发邮件，在Appfog上面要定时执行，需要部署一个[standalone app](http://blog.appfog.com/task-scheduling-support-on-appfog-with-standalone-apps/)。

nodejs这边要实现定时，要node-cron很方便就搞定了。

###发邮件

Appfog上面提供SendGrid的插件。SendGrid在全球提供发邮件服务，为了防止垃圾邮件，其申请时的审核挺严格的。不过在Appfog上面想用SendGrid就简单得多，把插件启动了就好了，赞！

###nodejs

用nodejs的异步回调机制开发，确实要转一下思维。有时候循环还不得不写成递归，很函数式语言。

中间遇到一个问题，就是Instapaper上的时间都是用moment.js或者timeago.js之类转成了XXX days ago、XXX months ago的语义式时间。因为我只想回顾最近这个月的文章，要做判断，所以要做逆过程，恢复成日期的。sugarjs这个模块很强大地解决了这个问题~


