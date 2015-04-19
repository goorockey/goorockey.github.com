title: Code Swarm
date: 2012-2-3
tags: 趣味
categories: others
---

今天拿自己项目组svn的日志来小玩了一下code swarm。

## 什么是code swarm?

code swarm是可以把svn、cvs、git等代码管理系统的日志，以可视化的形式展现的项目。

swarm是蜂群的意思，code swarm会以蜂群的形式表示每个人上传的文件。

<!--more-->

很多大的项目，如Apache、Python、豆瓣等，都做了自己的code swarm。

- Apache、Python等：<http://www.michaelogawa.com/code_swarm>

- 豆瓣：<http://v.youku.com/v_show/id_XMzQzNDc4MDk2.html>

我个人感觉，看别人的code swarm没什么特别的感受，只有看自己项目的才有感觉，呵呵。

## 使用 code swarm

可以从它google code 的主页中下载代码：<http://code.google.com/p/codeswarm/downloads/list>

我参照别人博客，使用了code swarm别的fork：<https://github.com/rictic/code_swarm>，它可以显示每个人的头像。

根据wiki或下载包内的README，使用code swarm，要先安装java和ant。

code swarm有可以通过run.bat或者runrepositoryfetch.bat来启动。run.bat需要我们手工把svn等软件的日志转为code swarm所需的xml，而runrepositoryfetch.bat可以输入reposition url，让code swarm自动下载日志并转换。我选择简单的runrepositoryfetch.bat方式。

在命令行提示中选择配置文件后，code swarm就能呈现了，但还可以修改配置来达到自己的效果。

## 配置code swarm

我觉得关键的配置：

- **InputFile** code swarm所需的xml文件

- **TakeSnapshots** 是否保存每一帧图片。code swarm不能直接输出视频，只能输出每一帧图片。所以我们要导出视频的话，需要自行把每一帧图片转换为视频。

- **SnapshotLocation** 保存输出图片的目录

还有一些控制帧速度，显示项等等的配置。

以下是fork中才有的配置：

- **AvatarFetcher** 每个人使用头像的来源。可以是NoAvatar（没有头像），GravatarFetcher（程序自己生成），LocalAvatar（提供本地目录，使用跟commiter id对应的头像）。

- **LocalAvatarDirectory** LocalAvatar方式时，存放头像的目录，目录里如果有文件名与commiter id相同的图片，则使用该图片否则使用默认头像。如果没有默认头像，则程序会中断。

- **LocalAvatarDefaultPic** 默认头像
 
- **AvatarSize** 选择LocalAvatar方式时，每张头像的高或宽。这里要求每张头像图片的尺寸相同，且一定是正方形。
 
- **CircularAvatars** 用圆形截取头像图片，这会用到程序代码src下的mask.png图片，这里也要注意修改AvatarSize后，mask.png的尺寸也要改变，否则程序中断。

## 把code swarm的图片合成为视频

方法很多，抱着学习的心态，我试着按wiki的方法用mencoder。

我要加背景音乐，所以加了参数-audiofile：

    mencoder mf://*.png -mf fps=33:type=png -ovc lavc -oac copy –audiofile bg.mp3 -o my.avi

这里可以通过修改fps的值来控制生成视频的帧速度。

还可以用mencoder添加字幕，这个我就没做了。

我的视频：<http://v.youku.com/v_show/id_XMzQ4NjA5ODYw.html>


P.S.相关资料

- 【code swarm wiki】 ：<http://code.google.com/p/codeswarm/wiki/GeneratingAVideo>

- 【fork of code swarm】：<https://github.com/rictic/code_swarm>

- 【制作code swarm】：<http://blog.xupeng.me/2012/01/12/code-swarm/>

- 【用mencoder把多张图片合成为视频】：<http://www.mplayerhq.hu/DOCS/HTML/en/menc-feat-enc-images.html>

- 【使用mencoder】：<http://hi.baidu.com/creatives/blog/item/41f6c32ad06cdb2bd42af128.html>

- 【windows下安装mencoder】：<http://hi.baidu.com/%D7%AF%D7%D3%C8%E7%CA%C7%CB%B5/blog/item/611a28b11abebd5f0823021b.html>
