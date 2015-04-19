title: 再次用linux做宿主系统
date: 2012-2-29
tags: linux
categories: linux
---

## 背景

之前就试过几次想把linux作为宿主来玩，但都因为舍弃不了一些windows下的软件而放弃了，例如wiz，qq等都是我常用的软件。试过wine，但总是有点错误，不完美。

最近也在微博上收集意见，发现用linux做宿主系统的人还是蛮多的。其实仔细想想，归根结底还是自己linux的操作还不熟练。

还好最近一段时间自己多了在linux的工作，这几天又下定决心一次装了linux做宿主来玩了。然后就想写个blog记录一下。

<!-- more -->

## 系统

这次选的linux的linux mint(64bit)，一个基于ubuntu的linux发行版。

官方说其目标是成为有windows那样市场占有率的linux发行版。

我不大喜欢ubuntu现在的natty，所以就在虚拟机试用了一下linux mint，感觉比ubuntu方便。

## 工具

- **浏览器**：firefox。一直用firefox，插件强大。

- **知识管理**：evernote。本地用nevernote，它是evernote的linux版；网页摘取用firefox的evernote clip。

- **X windows**: awesome。一种平铺窗口管理器。本来想用musca，但我没编译成功。唉~~

- **BT下载**：utorrent。utorrent有linux版，但是web gui版。

- **虚拟机**：virtualbox。在virtualbox装了xp，感觉比vmware快多了。。

## 日常应用

1.网络管理

ubuntu(包括linux mint)现在都是默认用NetworkManager来管理网络。我用了几次都不适应。这次立刻就把它卸载了，直接用脚本来管理网络。卸载命令：

    $ sudo apt-get --purge remove network-manager
    $ sudo apt-get --purge remove network-manager-gnome

然后就直接对/etc/network/interfaces和/etc/resolv.conf做修改，来配置网络了。

2.ADSL连接

寝室是用电信上网，如果不用路由拨号，就要自己电脑直接连到网口，自己拨号。

配置命令：

    $ sudo pppoeconf

在弹出的窗口中输入帐号和密码，注意之后有个提示选择是否开机时就自动拨号，如果不时总是直接连网口的，就不选吧。

配置完回到命令行，输入拨号命令就可以上网了：

    $ sudo pon dsl-provider

断开链接的命令则是：

    $ sudo pppoe-stop

3.连wifi

<http://www.jiangmiao.org/blog/1781.html>

4.做AP，共享wifi

<http://blog.csdn.net/feifei454498130/article/details/6642140>

5.截图

    $ scrot -bst [file]

然后用鼠标框主目标即可。如果没有制定输出文件路径file，默认输出到用户主目录，并以时间命名。

---

**2012.7.12 更新**

找到一篇介绍自己只使用命令行经验的博文，好正点！

<http://blog.chavezgu.com/2012/03/07/the-command-line-challenge/>

赞同里面循序渐进脱离GUI的方法；

- 坚持一天只使用命令行！

- 坚持一周！！

- 坚持一个月！！！

- 坚持半年！！！！

呵呵～～
