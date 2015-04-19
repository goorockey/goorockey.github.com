title: SSH端口转发
date: 2012-2-22
tags: [network, ssh]
categories: network
---

## ssh端口转发是什么
ssh端口转发也被叫ssh隧道，ssh代理。

所谓隧道，就是用X协议封装Y协议的数据包，靠X协议来进行Y协议通信。

总的来说ssh隧道提供了两个好处：

- 突破防火墙等，进行受限协议的通信。

- 使如telnet等不安全的协议传输经过ssh的加密通道，提高安全性。

<!-- more -->

## 三种ssh端口转发

ssh端口转发有三种：

- 本地转发

- 远程转发

- 动态转发

## 本地转发

命令是：

    $ ssh –L <local port>:<remote host>:<remote port> <ssh host>

考虑这样的场景：

![本地转发](http://www.goorockey.com/uploads/2012/02/image002_thumb.jpg)

一个运行在服务器116.1.1.1的程序提供端口389的数据通信，但防火墙只允许其他计算机对服务器做ssh的通信。

而客户端116.4.0.1为了完成通信，可以借助ssh的本地端口转发。

在客户端执行：

    $ ssh –L  7001:localhost:389     116.1.1.1

同时把客户端程序输出到本机的7001端口。注意命令中的localhost是相对于116.1.1.1来说的。

那么整个数据流会是：

- 客户端程序到数据输出到客户端的7001口

- 客户端的ssh一直检测7001口，但发现本机有数据包到达，则把数据包加密，并通过跟服务端116.1.1.1的ssh通路传输

- 服务端的sshd收到数据包后包解密，并转发到服务端的389口

- 服务端返回数据，并原路返回

另外，在ssh本地转发命令中的remote host可以使任意的机子，包括本机或其他计算机。

例如，考虑这样的场景，用本地转发来进行远程桌面：

![远程桌面](http://www.goorockey.com/uploads/2012/02/image_thumb.png)

现在要在机子A对机子C做远程桌面。但机子A和机子C都在不同的子网，不能直接通信，也都只能跟机子B用ssh通信。

然后已知windows远程桌面的服务端端口是3389，这我们可以在机子A执行：

    $ssh –L   13389:<C hostname>:3389     <B hostname>

命令中的13389是任意的，但要注意只有管理员才能用1~1024的端口。

然后在A机子执行yuan远程桌面：

    mstsc /v:13389

就能在A机子远程桌面控制C机子了。

## 远程转发

其实远程转发跟本地转发是基本相同的。

命令是：

    $ ssh –R  <local port>:<remote host>:<remote port>    <ssh host>

考虑这样的场景：

![远程转发](http://www.goorockey.com/uploads/2012/02/image003_thumb.jpg)

客户端A和服务端B的端口都还是7001和389。

跟本地转发时候不同的是，ssh连接的sshd在客户端A，ssh在服务端B。

所以，远程转发可以应用在客户端A只允许对其做ssh连接的时候。

如果客户端和服务端都允许ssh连接，那选择本地转发还是远程转发都可以。

## 动态转发
命令是：

    $ ssh –D <local port>  <ssh host>

![动态转发](http://www.goorockey.com/uploads/2012/02/image005_thumb.jpg)

跟其他两种端口转发不同的是，动态转发在数据包经过ssh通过到达服务端后，sshd会根据把封装数据包的协议，转发到对应的主机和端口。

这时候ssh隧道是充当了SOCKS代理的作用。这就可以用来翻X之类了。

## Ending

总的来说，ssh是个好东西~~~

## 相关资料：

- <https://www.ibm.com/developerworks/cn/linux/l-cn-sshforward/>

- <http://lesca.me/blog/2011/03/01/ssh-port-forwarding-priciple-and-praticle-application/>
