title: 在dotcloud上搭建wordpress
date: 2012-2-7
tags: [blog, dotcloud, wordpress]
categories: blog
---

之前在sourceforge搭建了wordpress，但SF有两点不好：

- SF在防火墙禁止了对外连接，使得WP好多功能、插件都无法使用（如Akismet）

- 访问SF速度很慢很慢

之后在寻找更好的方案时，偶遇dotcloud上搭建wordpress的文章，还看到了借dotcloud的ssh来翻x哦。

<!-- more -->

## dotcloud介绍

dotcloud是PaaS(Platform as a Service)的云计算平台，类似的还有GAE、Heroku、国内的SAE。

dotcloud支持几乎所有主流服务，php、java、python、ruby、mysql、postsql、node.js、Hadoop等等。详细参见：<http://docs.dotcloud.com/firststeps/platform-overview/>

dotcloud对搭建的应用没有空间和流量限制，但现在有两个限制：

- 每个用户只能有两个应用（每个用户对应一个邮箱，有多个邮箱就可以多申请几个了嘛，呵呵）

- 网上流传数据库容量限制在10M（官网上没找到这个说法）

dotcloud的访问速度还是很快的。

## dotcloud使用

搭建wordpress的话，网上资源很多，我主要是参考：

- <http://blog.yangtse.me/2011/10/wordpress-dotcloud/>

- <http://olddocs.dotcloud.com/tutorials/wordpress/>

反正算蛮简单的。

## 建议：

不要用postinstall脚本连接的方法。由于dotcloud的push，是把原有的全部删除，重新建立一份，这个脚本是把wp-content移出current目录，防止push后把wp-content覆盖了。

但由于wordpress的bug（其他应用不确定），一些插件会出现路径的错误。例子可见：<http://blog.yangtse.me/2011/10/wordpress-dotcloud-habari-error/>

dotcloud的push我除了第一次上传代码用过，之后都是直接用ssh控制的。所以我就干脆不用postinstall脚本了。
