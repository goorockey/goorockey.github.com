title: 博客转移了（改用sourceforge+wordpress）
date: 2012-1-28
tags: [blog, wordpress, sourceforge]
categories: blog
---

这几天把博客从GAE+micolog转到了sourceforge+wordpress，主要是考虑到：

- 服务器：GAE现在对流量加大了限制，而sourceforge是没有流量或空间限制的

- 博客系统：wordpress的资源币micolog的要多很多

主要做了几件事：

- sourceforge+wordpress建站

- micolog数据导入到wordpress

- 优化wordpress：主题、插件、google analytics

<!--more-->

考虑到SEO的域名权重问题，就保留sourceforge的二级域名了。

教程的话网上资源很多，值得记录的东西：

- 安装wordpress，及其主题、插件等资源时，最好用ssh登录到sourceforge，从sourceforge那边wget下载，而不要从自己机子下载。因为资源多在国外的服务器，这样速度快多了。

- 注意关闭sourceforge项目管理中不必要的访问权限，以免博客里的文件被在sourceforge中能被访问。

感叹sourceforge真是太伟大了！ 感叹wordpress真的就两个字：折腾！！

---

2012.1.29 更新：

昨天还发现github+octopress这种免费建博客的形式。

但建的是静态站点，即服务器存放的就只是html+css+js网页,没有php、asp等。

优点：

- 最重要的优点是git的分布式管理，保证博文等数据不容易丢失；

- git的其他各种优点

- 静态站点的优点：响应速度快，对服务器端的负荷小

缺点:

- 我觉得缺点主要是静态站点的后期维护成本高！  因为到了后期，文章等数据多了，服务器就会存有大量网页，大大增加修改、备份等维护的成本！！

- 折腾octopress的成本。octopress的官网说其是"a blog framework for *HACKERS*"，本身是ruby应用，即折腾它要玩ruby。要折腾octopress似乎比较适合程序猿。（ruby折腾迷略过此条。。）


考虑到静态站点的后期维护问题，我觉得还是SF+WP比较适合现在的我啦~~嘻嘻

---

2012.2.5 更新：

根据SF的官方规定，架在上面的服务的outbound connection会被禁止。详细请参看： <http://sourceforge.net/apps/trac/sourceforge/wiki/Project%20web%20and%20developer%20web%20platform#Outboundconnectivity>

而因为很多WP的功能、插件等都会用到向外发送请求，所以在SF上架WP的时候，很多功能和插件都用不了的。

例如，WP内置的更新ping服务器，Akismet， google sitemap，微博同步等等。。

现在还没找到什么办法，唉。。
