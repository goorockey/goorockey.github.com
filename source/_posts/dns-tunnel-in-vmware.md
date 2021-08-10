title: 用VMWare组网，实验DNS隧道
date: 2012-3-15
tags: [network, vmware, 隧道]
categories: network
---

继续用VMWare来组网，这次要测试我想试很久的DNS隧道，之前碍于没有找到有独立ip的方法（当然是要免费的~~），现在用VMWare就可以了。

DNS隧道是什么就不解释了。google一下DNS隧道能搜到风河、云风两个大牛相关的blog。这次我用iodine来实现DNS隧道。

<!--more-->

## 场景 
现在情况是，用户只能跟外界有DNS通路，想借此进行平常的http、ftp等通信。

据说平常的CMCC等开放热点，虽然http等要账号和密码，但DNS是通的，然后你懂的了。

简单的拓扑图如下：

![拓扑图](http://goorockey.github.io/uploads/2012/03/image_thumb2.png)

整个回路就是：

- 用户把要想跟外网进行通信的数据包用DNS协议封装

- 得到的DNS包发送给DNS服务器，要求做DNS解析

- DNS服务器根据域名，解析出DNS代理的ip，并把数据包发给它

- DNS代理把数据包解封，并转发给外网的目标地址

- 外网回复的数据包原路返回，这样就完成通讯了。
 
## VMWare环境模拟

这次我用了三台机子，系统还是CentOS 6.0：

 主机名 角色 网卡ip

    HostA   用户 192.168.149.128 (Host-only)
    HostB   DNS代理 192.168.126.130 (NAT)
    HostC   DNS服务器 192.168.149.130 (Host-only)、192.168.126.233 (NAT)

要模拟的初始状态就是：

	HostA（用户）可以跟HostC做DNS解析，但不能访问外网 。     （所以虚拟网卡用Host-only模式）
	HostB（DNS代理）可以跟外网通信。            （用NAT和Bridged都可以，这次我选用NAT）
	HostC（DNS服务器）可以跟HostA进行DNS解析，且能跟HostB通信。   （所以用两张网卡，为了分别跟HostA和HostB通信）
	HostA的iptables不允许HostA和HostB之间互访

## DNS服务器配置

刚开始看教程好像很繁琐，感觉conf文件好多啊，而且配置项也多~~

静下心来看，其实要实现最基本的的DNS解析很简单，主要就是修改两个文件。

1.安装

需要在HostC执行以下命令，安装DNS服务器所需的bind和caching-nameserver：

    $ yum install –y bind bind-utils bind-chroot caching-nameserver

2.修改named的conf文件（/etc/named.conf）

添加域名goorockey.go域名的配置：

    zone “goorockey.go” IN {
        type   master;
        file      “goorockey.go.zone”;
        allow-update {none; };
    }

大概解释：

- zone “ goorockey.go”： 指示要添加goorockey.go这个域名的正向解析。正向解析就是指域名到ip的解析，反向解析是指ip到域名的解析。例如想通过查询DNS服务器，知道192.168.0.1判定了多少域名，则在DNS服务器上配置zone “1.0.168.192.in-addr-arpa”的项。

- type master：对于goorockey.go这个域名，当前DNS服务器是它的主DNS服务器。type可以还可以使hint和slave。只有zone “.”可以配置type hint。type slave是指对于这个域名，当前DNS服务器是辅助DNS服务器，即它的DNS记录是从主服务器拷贝过来的，目的是为了达到DNS解析的分布式、负载均衡。

- file “goorockey.go.zone”：这个域名的DNS记录文件在goorockey.go.zone，文件所在目录在/etc/named.conf的options项中的directory来定义。默认是/var/named

- allow-update：定义时候允许更新

要注意的是，/etc/named.conf中的options项是所有域名的全局配置。默认时，有：

- `allow-query   {   localhost;   };` 意思是只允许本机做DNS查询，当然要把它注释掉。

- `listen-port    53    {  127.0.0.1;   };` 意思是服务端口为53，但监听的ip是127.0.0.1，这样就不能让别的机子访问DNS解析服务了。所以可以把这句话注释掉，或者把ip改为0.0.0.0或指定ip。

3.编辑goorockey.go的DNS记录文件

根据我们在/etc/named.conf的配置，文件是/var/named/goorockey.go.zone。

创建此文件，并编辑内容为：

    @  IN SOA localhost. root.localhost. (20120315 3600 1800 36000 3600)
       IN NS localhost.
    goorockey.go IN A 192.168.126.130

大概解释：

- 第一行是一条SOA记录。@指代当前域名，就是/etc/named中的goorockey.go。SOA记录是域名有效性的相关属性。localhost.是主服务器的地址。root.localhost.是邮箱。主要DNS记录文件的地址都用FQDN，每个地址最后的句号“.”表示结束。如果没有句号“.”，会自动追加域名，例如没有句号的localhost会解释成”localhost.goorockey.go“。后面就是具体属性项。

- 第二行开始是两个空格，第一个空格表示继续上一条的内容，这里指”@“，第二个空格就是分割@和IN的。这一行表示域名goorockey.go的域名服务器是本机。

- 第三行是一条A记录，A for address。意思就是域名goorockey.go会解析成ip 192.168.126.130。可以看出，搞这么久，就是为了找到这句话。所以说A记录是DNS服务器的核心，就是它标明DNS解析的。

DNS记录类型还会有：

- PTR用在反向解析

- MX用在邮件服务器

- TXT就是纯文本，对DNS服务器做标注

4.运行DNS服务

在HostC执行： `$ service  named   start` 或者 `$ /etc/init.d/named start`,这就可以运行DNS服务了。

对HostC的/etc/resolv.conf添加 `nameserver 127.0.0.1`

则在HostC用nslookup能成功返回DNS信息：

![DNS信息](http://goorockey.github.io/uploads/2012/03/20594453_thumb.png)

但还要配置iptables，使其他机子可以访问DNS服务的端口。

对于默认的53端口，在HostC运行：

    $ iptables –I INPUT -p tcp --dport 53 -j ACCEPT
    $ iptables –I INPUT -p udp --dport 53 -j ACCEPT

要解释一下的是，DNS包有可能以tcp或者udp方式传输。一般首选是udp方式。但因为udp包长度只能是512字节，也不能分包，所以如果当DNS包长度大于512时，就会选择tcp方式。所以这里要对tcp和udp都设置ACCEPT。

在HostA和HostB的/etc/resolv.conf添加HostA的ip后，就能正确解析goorockey.go了。

## iodine

iodine是外国人写的开源DNS隧道工具，有linux版、windows版和Mac版的。教程看它的ReadMe或者HowToSetup都比较清楚。

下载并安装对应自己版本的iodine后就能使用了。

在DNS隧道的服务器端(HostB)，先执行：

    $ iodined -f 10.0.0.1 goorockey.go

输入密码后，服务端就运行了。注意服务端运行的是iodined，有”d“。

在客户端（HostA)，执行：

    $ iodine -f -c 192.168.126.130 goorockey.go

其中192.168.126.130是服务端（HostB）的ip。

然后还要配置一下，HostA，HostB，HostC的iptables，使它们的DNS包可以通过就可以了。

这时候，HostA的虚拟网卡ip是10.0.0.2，HostB的虚拟网卡ip是10.0.0.1。两台机子已经建立了VPN。

本来两台不能互访的机子就可以访问了。

例如在HostA就可以ssh HostB了 ：

    $ ssh 10.0.0.1

 然后就可以用ssh隧道过去来做代理了~~

## 小结

那时候看到DNS隧道，真是非常的兴奋，感觉太爽、太妙了。其实协议都可以这样做隧道，只是那时候没有意识到而已。

之后还继续想实验一下ICMP隧道，看一下iodine的代码。O(∩_∩)O哈哈~

## 参考资料：

- 【风河的博文】<http://www.nsbeta.info/archives/96>

- 【云风的博文】<http://blog.codingnow.com/2011/06/dns_tunnel.html>

- 【iodine】<http://code.kryo.se/iodine/>
