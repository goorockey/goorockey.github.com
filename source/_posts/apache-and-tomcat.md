title: Apache和Tomcat
date: 2011-11-22
tags: [linux, apache, tomcat]
categories: linux
---

调研了一下Apache和Tomcat：

1.apache 只是一个web服务器，负责响应客户端的请求。

2.apache对于页面请求：

<!--more-->

    如果是静态页面请求，会立刻返回相应的页面；
    如果是动态页面请求，apache会根据httpd.conf中AddType的配置，把请求提交给合适的动态脚本解析程序来处理，处理后生成的静态页面返回给apache，再返回给客户端。所以在配置php和jsp这样的环境的时候，都要在httd.conf中添加对应的AddTpye语句。

3.tomcat侧重于是一个Servlet/JSP的容器，但也能可以独立于apache运行，响应html请求

4.tomcat响应静态页面较apache要慢

5.整合apache和tomcat可以有三种方法:JK,http\_proxy,ajp\_proxy

具体介绍见：<http://www.ibm.com/developerworks/cn/opensource/os-lo-apache-tomcat>

JK较老，相对比较稳定，配置比较麻烦

两种proxy模式原理都是让apache做tomcat的代理，配置简单
