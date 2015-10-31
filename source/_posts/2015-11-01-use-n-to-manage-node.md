title: 使用n管理nodejs
date: 2015-11-01 00:11:30
tags: [programming, nodejs, n]
categories: nodejs
---

n和nvm都是大家常用的nodejs版本管理工具，最先接触的是n，所以我一般用n。(BTW, n还是TJ大神的项目咧~)

## 安装

以前需要先安装npm才能安装n，现在n有了独立的安装工具[n-install](https://github.com/mklement0/n-install)，可以通过以下命令安装:

    curl -L http://git.io/n-install | bash

另外还可以用`n-update`更新n，`n-uninstall`卸载n(其实就是删除n的目录和环境变量设置)

<!--more-->

## 加速

用法就不说了，`n help`说明都比较清晰。记录一下使用淘宝镜像加速n对nodejs的包下载。

n支持`n project`命令，通过设置`PROJECT_NAME`和`PROJECT_URL`环境变量，指定下载nodejs包的源。国内从原始官方下载真的太慢了。通过以下命令使用淘宝镜像加速：

    PROJECT_NAME="node" PROJECT_URL="https://npm.taobao.org/mirrors/node/" n project stable
    PROJECT_NAME="io" PROJECT_URL="https://npm.taobao.org/mirrors/iojs/" n project stable

中间可能会遇到提示`Invalid version XXX`的问题，如这个[Issue](https://github.com/tj/n/issues/314)。原始是n脚本在下载包前，用curl或wget先测试链接时候有效:

[502行](https://github.com/tj/n/blob/1388dd0926abfa5c21228acbdb33795bee7c5d9d/bin/n#L502):

    is_ok $url || abort "invalid version $version"

[360行](https://github.com/tj/n/blob/1388dd0926abfa5c21228acbdb33795bee7c5d9d/bin/n#L360):

    is_ok() {
      if command -v curl > /dev/null; then
        $GET -Is $1 | head -n 1 | grep 200 > /dev/null
      else
        $GET -S --spider 2>&1 $1 | head -n 1 | grep 200 > /dev/null
      fi
    }

这里淘宝镜像貌似返回了302重定向，所以is_ok返回失败。保证版本有效的前提下，暂时注释掉502行可以解决问题。当然改grep也可以~
