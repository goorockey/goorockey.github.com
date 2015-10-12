title: electron使用总结
tags:
  - electron
  - programming
  - nodejs
categories: nodejs
date: 2015-10-12 22:38:57
---


帮人做个简单的文件整理程序，尝试用[electron](http://electron.atom.io/)来实现，总结一下。


## 主进程和渲染进程

electron的程序运行时，分为主进程和渲染进程。主进程即为`var app = require('app');`所在一侧，也是程序的入口。通过`BrowserWindow`实例`loadUrl`访问网页时，会创建出渲染进程。

某些包是只有主进程才能包含的，如常用的`dialog`。想在渲染进程的逻辑中调用这些包，有两个方法，一个是使用`remote`包，如：

    var remote = require('remote');
    var dialog = remote.require('dialog');

另一个方法是使用`ipc`，即进程通信，发消息给主进程，由主进程调用后，把结果再通过`ipc`返回渲染进程。

<!--more-->

## 打包

用[electron-packager](https://github.com/maxogden/electron-packager)打包生成各平台的程序，还是很方便的，但是有些坑。

### 速度慢

对某个平台第一次打包的时候，packager需要下载对应的electron包，那速度真是慢啊！

幸好淘宝有electron[镜像](http://npm.taobao.org/mirrors)。通过设置`ELECTRON_MIRROR`环境变量，可以大大加快速度。

    ELECTRON_MIRROR=http://npm.taobao.org/mirrors/electron/ electron-packager ...

### 体积大

electron打包出来的程序，一般至少100M，对于一个小程序来说有点太大了，体积问题感觉是很多跨平台工具的通病。

为了减少体积，记得使用packager的ignore参数，排除掉例如electron等程序运行不必要的包，如果指定了packager的输出路径在程序的目录，记得也排除掉，不然会越打包越大。

最后我使用的打包命令如下：

    electron-packager . <程序名字> --platform=win32,darwin --arch=all --version=0.33.7 --out=dist/ --overwrite --ignore=node_modules/electron-* --ignore=node_modules/.bin --ignore=.git --ignore=dist --prune

把命令写在`package.json`的`scripts`里，比如`package`命令，则打包时运行：

    ELECTRON_MIRROR=http://npm.taobao.org/mirrors/electron/ npm run package


## electron一些资源

- [官方文档](http://electron.atom.io/docs/latest/)
- [electron内部结构](https://github.com/ilyavorobiev/atom-docs/blob/master/atom-shell/Architecture.md)
- [awesome-electron](https://github.com/sindresorhus/awesome-electron)
- [electron-sample-apps](https://github.com/hokein/electron-sample-apps)

用得还不够深入，之后遇到更多问题再补充。
