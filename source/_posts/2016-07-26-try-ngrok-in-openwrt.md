title: openwrt中使用ngrok
tags:
  - openwrt
  - ngrok
  - linux
categories: linux
date: 2016-07-26 00:53:00
---


由于所在网络没有公网IP，不能弄DDNS，想要在外面控制家里的openwrt路由，想到用内网穿透神器[ngrok](https://ngrok.com/)。

ngrok跟Teamviewer等的原理类似，内网的机子主动跟外网的机子保持连接，从而使内网机子“暴露”出外网。

## 准备

需要以下资源：

- 域名，可修改其DNS记录
- openwrt路由
- vps，有公网IP，如：10.0.10.1

<!--more-->

## DNS设置

使用ngrok需要通过域名定位到对应的服务。假设我们使用ngrokd.goorockey.com，则需要在goorockey.com中增加两个DNS记录

    ngrokd.goorockey.com A 10.0.10.1
    *.ngrokd.goorockey.com A 10.0.10.1

其中`10.0.10.1`为所拥有的vps的ip。注意需要添加第二条的泛解析，因为ngrok的http/https，需要子域名定位服务。

## 编译ngrok for openwrt

获取ngrok在openwrt的源码

    git clone https://github.com/dosgo/ngrok-c

在openwrt官网下载<https://downloads.openwrt.org/>对应路由的SDK，如我的小米路由mini为：

    wget -O openwrt-sdk.tar.bz2 https://downloads.openwrt.org/chaos_calmer/15.05.1/ramips/mt7620/OpenWrt-SDK-15.05.1-ramips-mt7620_gcc-4.8-linaro_uClibc-0.9.33.2.Linux-x86_64.tar.bz2
    tar xvf openwrt-sdk.tar.bz2

复制ngrok-c里面的openssl目录到openwrt-sdk指定目录下

    cp -r ngrok-c/include/openssl openwrt-sdk/staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/include

修改ngrok-c里面的`openwrtbuildv2.sh`，添加`openwrt-sdk/staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/bin`到PATH，修改`STAGING_DIR`为对应目录，查看openwrt的bin目录，修改对应的`CC`，如`mipsel-openwrt-linux-g++`

执行`openwrtbuildv2.sh`，在`build-mips`目录得到`ngrokc`，即为openwrt的ngrok客户端

把ngrokc复制到路由内，可能还需要路由安装`libstdcpp`

## ngrok server

ubuntu下可以直接`apt-get install ngrok-server`，其他的可以到ngrok官网下载可执行版本，或者源码编译。源码编译可以修改ngrok服务端的ssl证书，安全性更好。

## 使用ngrok

服务端执行:

```
  ngrokd -domain="ngrokd.goorockey.com" \
         -httpAddr=":10001" -httpsAddr=":10002" \
         -tunnelAddr=":10003" -log=/var/log/ngrok.log
```

客户端执行：

```
  ngrokc -SER[Shost:ngrokd.goorockey.com,Sport:10003] \
         -AddTun[Type:http,Lhost:127.0.0.1,Lport:80,Sdname:test] \
         -AddTun[Type:https,Lhost:127.0.0.1,Lport:443,Sdname:test2] \
         -AddTun[Type:tcp,Lhost:127.0.0.1,Lport:22,Rport:10004]
```

其中`10003`是服务端指定的ngrok服务监听端口，`10001`是服务端指定访问http服务时的端口，`10002`是服务端指定访问https服务时的端口
`10004`是客户端指定tcp服务绑定到服务端的端口

以上命令就可以达到：

- 访问`http://test.ngrokd.goorockey.com:10001`，能访问到路由在80端口的http服务
- 访问`https://test2.ngrokd.goorockey.com:10002`，能访问到路由在443端口的https服务
- 访问`tcp://ngrokd.goorockey`，端口10004，能访问到路由在22端口的tcp服务，如路由22端口是ssh，则可以ssh ngrokd.goorockey.com -p 10004

其他更多选项可参考[ngrok官网](https://ngrok.com/)和[ngrok-c的仓库](https://github.com/dosgo/ngrok-c)


## 参考资源

- <https://ngrok.com/>
- <http://www.jianshu.com/p/8428949d946c>
- <https://github.com/dosgo/ngrok-c/blob/master/README.md>


