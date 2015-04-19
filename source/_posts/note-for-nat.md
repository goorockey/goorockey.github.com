title: NAT学习总结
date: 2012-3-11
tags: [network, nat]
categories: network
---

最近要恶补一下计算机网络的基础知识，今天先总结一下NAT。

## NAT的背景

随着Internet的普及，网络中的ip资源是越来越紧张。而NAT就是为了解决这个问题的方案。

NAT是Network Address Translation,网络地址转换，会在网关中实现局域网内部ip和外网ip之间转换。

![NAT](http://www.goorockey.com/uploads/2012/03/nat.png)

如上图，局域网内部网段是192.168.1.X，这些ip只在这个局域网内有意义，外网无法根据这些ip定位计算机。

而NAT就是做内网和外网这样两个网络间的ip转换。

<!-- more -->

## NAT的类型

按照通信发起方的不同，NAT可以分为：

- SNAT，即Source NAT

- DNAT，即Destination NAT

1.SNAT

SNAT是对数据包源ip的转换，主要用于内网机子发起连接到外网的情况。

【考虑以下场景】：

内网ip为192.168.1.2的机子向外网的8.8.8.8发包。如果数据包的源ip直接就是192.168.1.2，数据包虽然可以成功到达8.8.8.8，但是它无法根据192.168.1.2的源ip回复数据包，因为在外网中没有192.168.1.2，则造成通信失败。

而SNAT就是当内网发起连接到外网时，具有NAT功能的机子，例如网关，在数据包要出外网之前，把包的源ip改为这个局域网的外网ip，如1.1.1.1，同时会有映射表记录转换。

由于1.1.1.1是外网中有意义的ip，1.1.1.1和8.8.8.8可以成功的完成数据包的发送和接受。这时8.8.8.8是把1.1.1.1作为目标ip回复数据包，网关收到数据包后，会查表把包的目标ip映射回内网机子ip 192.168.1.2。

可以看出来，整个过程对内网机子是透明的，即发送和接受数据包的ip都对应，仿佛没有做过转换。

2.DNAT

DNAT是对数据包目标ip的转换，主要用于外网向内网发起连接的情况。

【考虑一下场景】：

在内网中有很多机子，其中有一台ip为192.168.1.2的机子是对外网提供服务的web服务器，现在外网的8.8.8.8要访问它。但对于8.8.8.8来说，web服务器所在ip会是192.168.1.2所在内网的外网ip，如1.1.1.1。

可想而知，当8.8.8.8向1.1.1.1发送数据包，网关会做DNAT，把包的目标ip从1.1.1.1改为192.168.1.2，同时会把转换记录到一个表中。然后192.168.1.2回复数据包，包的源ip是192.168.1.2，目标ip会是8.8.8.8。网关接受到包后，则查表，把源ip修改回1.1.1.1。

## NAT的转换方式

NAT有四种转换方式：

- 静态NAT  (Static NAT)

- 动态NAT  (Dynamic NAT)

- 过载     (Overload NAT)

- 重叠     (Overlap NAT)

1.Static NAT

![Static NAT](http://www.goorockey.com/uploads/2012/03/nat-static.jpg)

局域网有多个外网ip，数量等于或多于内网ip数。

则做NAT转换时，每个内网ip对应一个外网ip。

网关的表中记录着这样一对一的关系。

2.Dynamic NAT

![Dynamic NAT](http://www.goorockey.com/uploads/2012/03/nat-dynamic.jpg)

局域网有多个外网ip，但数量少于内网ip数。

则做转换时，每个内网ip从当前未被映射的外网ip选取一个来做转换。

网关的表也会记录这种转换，且会根据情况不断更新。

3.Overload NAT

![Overload NAT](http://www.goorockey.com/uploads/2012/03/nat-overload.jpg)

如果局域网只有一个外网ip，每个内网ip都映射到这个外网ip，但端口口会不同。

网关的表中会记录这种端口的映射。

4.Overlap NAT

![Overlap NAT](http://www.goorockey.com/uploads/2012/03/nat-overlap.jpg)

当内网的ip在外网中已经注册且已被其他机子使用时，网关要在选择一个外网中已注册但未被使用的ip做转换。

网关的表中记录这种转化。

## 小结

其实所谓的内网和外网都是相对而言，只要是两个网络间的通信，都可以或需要用网关或路由做NAT。

【参考资料】：

- <http://article.yeeyan.org/view/185403/150856>

- <http://zh.wikipedia.org/wiki/网络地址转换>
