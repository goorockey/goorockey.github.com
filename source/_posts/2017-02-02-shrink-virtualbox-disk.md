title: 回收Virtualbox硬盘空间
date: 2017-02-02 15:07:54
tags:
- Virtualbox
- linux
categories: linux
---

Virtualbox虚拟机的虚拟硬盘文件（一般是vdi文件），支持根据使用情况动态增加，即创建硬盘时候设置大小为128G，并不会直接在宿主机创建一个128G的文件，而是根据虚拟机的使用慢慢增加。

但虚拟硬盘文件的大小不会因为虚拟机里面使用空间减少就自动缩减，需要使用工具完成。

<!--more-->

对于Linux虚拟机，Ubuntu为例：

- 安装zerofree

      $ apt-get install zerofree

- 重启虚拟机
- 在启动的grub菜单，选择进入recovery模式
- 对要回收的硬盘，以只读模式重新挂载

      $ mount -n -o remount,ro -t ext4 /dev/sda1 /

- 使用zerofree把硬盘没使用的空间置零

      $ zerofree /dev/sda1

- 关闭虚拟机
- 在宿主机使用VBoxManage工具压缩vdi文件，回收空间

      VBoxManage.exe modifyhd xxx.vdi compact

Done!
