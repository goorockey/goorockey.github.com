title: 红米2刷第三方rom小记
tags:
  - Android
  - redmi2
categories: Android
date: 2016-07-27 21:39:29
---


由于洁癖，相对于MIUI，更喜欢原生的Android系统，更干净更安全。

今天手贱更新opengapps的时候，把手机刷无限重启了，recovery也进不了，只能重新线刷。借此机会记录一下。

## rom选择

到国外最流行的Android发烧友论坛[xdadevelopers](http://forum.xda-developers.com)找到对应版块，如红米2的 <http://forum.xda-developers.com/redmi-2/development>

浏览一下rom的更新时间、遗留问题、最后几页的讨论，选择自己觉得比较稳妥的rom，毕竟换不同类型的rom是要清SD卡，重新安装配置软件，很麻烦，谁也不想经常这样。

<!--more-->

## recovery

找到适合自己机型的recovery，对于卡刷需要签名的recovery，fastboot刷则不需要。

需要安装非MIUI的系统，需要第三方的recovery，最流行的TWRP或CWM

刷recovery时，如果能进原有的recovery，可以卡刷，即把要刷的recovery包放到手机里，然后进recovery选择包来安装。

还可以在fastboot模式，刷recovery。一般手机进fastboot是同时按`音量减键`和`开机键`开机进入。

进入fastboot模式后，手机连PC，用android工具集（即adb所在的）的fastboot执行：

```
fastboot devices  # 查看fastboot模式的设备
fastboot flash recovery xxx.img  # 刷recovery
```

## 第三方rom

进recovery，把rom复制进手机并选择安装，然后清cache和dalvik，重启即可。

对于在已有rom基础上，重新刷另外类型的rom，刷rom之前最好还请data、system。重新刷同样的rom（只是版本升级），可以不用。

## opengapps

现在很多rom里面都会去掉google服务，如果需要，则可以在刷完rom后，刷opengapps

到<http://opengapps.org/>找到对应机型和rom android版本的opengapps。

一般选择micro足以，包含最基本的服务。谨慎刷nano和pico，刷后不一定能保证机子能正常启动。

跟刷rom一样，进recovery选择opengapps包安装，清cache和dalvik即可。

今天刷pico的opengapps后启动会一直报`setup wizard has stopped working`，网上找到[解决方法](https://www.reddit.com/r/cyanogenmod/comments/3wv9t4/setup_wizard_has_stopped_loop/):

- adb执行：`adb shell pm grant "com.google.android.setupwizard" android.permission.READ_PHONE_STATE`
- 到`setting - apps`，给google service的相关app授权


=================== 分割线 =======================

暂时记录到这，随笔记录可能有误。刷机有风险，请谨慎。
