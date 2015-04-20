title: VMWare组网实验(NAT)
date: 2012-3-13
tags: [network, vmware, nat]
categories: network
---

本着“干中学”的精神，看完资料，还是用VMWare来练习一下使用NAT，好加深认识。

实验涉及：NAT，iptables，

## 实验目标

这次我要用iptables实现NAT功能（SNAT和DNAT）。

先上拓扑图（可能有点不规范）：

![拓扑图](http://img.goorockey.com/2012/03/image_thumb.png)

如图分别有4台机子：A、B在内网，但在不同的网段中，C做网关，控制网段间的访问。D在外网。

要达到：

- A、B能通信（内网不同网段的互访）

- A、B能通过C与外网通信

- D能通过C访问到A、B的服务

## 环境

用VMWare虚拟出这4台机子，VMWare的版本为8.0

每台机子都跑CentOS 6.0

## VMWare环境配置

安装4个虚拟机，都装上CentOS，主机名分别定为hostA、hostB、hostC、hostD，对应A、B、C、D。

VMWare新建几张网卡（菜单栏【edit】-【Virtual Network Editor】），要求一张为Bridged（NAT应该也行），两张为Host-only。

![VMWare网卡设置](http://img.goorockey.com/2012/03/151306312_thumb.png)

设置A、B网卡分别为VMnet1和VMnet2，这是为了使它们原始都不能互访。

外网的D网卡设为VMnet0

C则有三张网卡VMnet0、VMnet1、VMnet2，这样C原始都能访问到A、B、D。

然后进入每个虚拟机，为了方便，我都设置为静态ip（网段跟上图对应）：

    A：192.168.149.128
    B：192.168.214.128
    C：192.168.4.233（eth0），192.168.149.130（eth1），192.168.214.130（eth2）
    D：192.168.4.234

CentOS里面配置网卡方法就是修改/etc/sysconfig/network-scripts/ifcfg-eth\*，没有则自己创建一个。

关键项就是ONBOOT，IPADDR，NETMASK，GATEWAY，DNS1，DNS2，PEERDNS

要注意的是，有PEERDNS项，当它值为yes，则会把DNS1和DNS2覆盖地写入/etc/resolv.conf。

这对于多网卡的C，如果ifcfg-eth0、ifcfg-eth1、ifcfg-eth2都设了PEERDNS，由于开机是按名字的顺序执行，则会把ifcfg-eth2的DNS写入/etc/resolv.conf，前两个文件的DNS会无效了的。所以我只在ifcfg-eth0配置PEERDNS=“yes“。

好，初步网络环境配置完成。

现在情况是：

	ABD都不能互访，因为在不同的网段
	C则都能跟它们三个互访

## 配置网关C的iptables，实现NAT

到关键也是好玩的地方了。

接下来配置网关C的iptables，实现不同网络间地址的转换（NAT）。

iptables内容比较多，详细可以参考：<http://www.frozentux.net/iptables-tutorial/cn/iptables-tutorial-cn-1.1.19.html>

1.A、B通过C实现通信

这个比较简单，没用到iptables，把A的网关设为C的对应网卡的ip（192.168.149.130），B的网关设为C对应网卡的ip（192.168.214.130）。

然后打开C的ip转发，在C中执行：

    $ echo 1 > /proc/sys/net/ipv4/ip_forward

这就把C作为了A、B的网关。A、B间通信的数据包会发到C，靠C的网卡间转发来完成通信。AB就可以相互ping通了。

2.A、B通过C与外网通信（SNAT）

现在A、B都不能跟D通信，因为现在A、B发到D的数据包源地址（192.168.149.128,192.168.214.128），D是无法知道的（D在C的同一个网络，网关设为相同的ip）。则包可以发到D，但D回复不了，因为它的网关不知道A、B。

现在就通过SNAT把A、B发送的包在经过C时，把源地址改为C的外网ip（192.168.4.233），这个D是知道的，也就可以顺利回复了。

具体在C中执行：

    $ iptables –t nat –A POSTROUTING –o eth0 –j SNAT --to-source 192.168.4.233

这样A、B就能ping通了。

SNAT可以看看我的博文。 嘻嘻……

3.D通过C访问A、B的服务（DNAT）

现在A、B可以跟D通信，但D不能主动访问A、B。还是因为D只知道C，不知道A、B。

假如现在A开了19991口的sshd：

在A的/etc/ssh/sshd\_config中添加：

    ListenAddress 0.0.0.0:19991

重启sshd

    $ service sshd restart

在A中让iptables允许对19991口的访问

    $ iptables –I INPUT -p tcp --dport 19991 -j ACCEPT 

现在D想ssh到A的19991，则可以在C中执行以下命令，实现DNAT：

    $ iptables -t nat -A PREROUTING -p  tcp --dport 19991 -j DNAT --to-destination 192.168.149.128

现在D可以通过ssh到C的19991口来ssh到A了。

## 小结

整个实验搞完，对iptables，NAT的原理还是深刻了不少。

然后，就是VMWare是个好东西。
