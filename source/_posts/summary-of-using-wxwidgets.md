title: 用wxWidgets做移植的总结
date: 2014-04-21
tags: [programming, wxwidgets, cpp]
categories: programming
---

[wxWidgets]: http://www.wxwidgets.org "wxWidgets"

这几个月做的项目是把一个承载在MFC的软件从Windows移植到Mac，现在进入最后验收阶段了。当时网上调研了一下，决定用[wxWidgets]这个跨平台的开源库来帮助移植。总的来说，只要把原软件对Windows API的调用都改为对wxWidgets的调用，基本就完成了移植。但基于wxWidgets和MFC一些设计上的差异和原软件特殊的功能，还不是简单的全局替换就能了事的，甚至还得改wxWidgets的源码。

wxWidgets库在总的结构上跟MFC相似，比如消息响应、相关类的命名。它现在已经出到了3.0，总体还是比较成熟了，但还是好些不完善的地方，这个在看它源码的时候就会发现挺多TODO comment。不过它的官方论坛和stackoverflow上相关问题还是挺活跃的，在上面提问很快就能得到一些资深程序员的答复。有一次我误以为发现了它的一个bug（其实是我理解错了），在上面提问，回复的人不仅有文字的讲解，还附上了自己写的测试用例，让我真心赞叹对方好负责任啊。

现在总结一些项目移植过程中遇到的问题吧。

<!-- more -->

## wxRect和CRect

Windows的CRect要替换成wxWidgets的wxRect真不能简单的替换，因为两者内部设计不一样。

第一，两者对应接口的入参有不同，最经典的是两者的构造函数，CRect是传left,top,right,bottom，wxWidgets是传left,top,width,height，等于说所有构造CRect的地方都要把第三、第四个参数做减法。

第二，wxRect定义自己所表示的矩形，范围是[left, right - 1]和[top, bottom - 1]，意思就是右边界和下边界是不属于矩形一部分的，这个从其源码能看到。官方说法是不承认width或height为1的矩形，那应该视为线段。这在移植一些矩形操作时，会跟CRect的有些差异。

第三，也是我觉得最坑的，CRect内部直接保存left,top,right,bottom来定义矩形，而wxRect内部保存left,top,width,height来定义矩形，即wxWidgets要GetRight的时候，是返回left + width，SetRight的时候set的是width，其他的同理。这看着没什么问题。但如果我们先SetRight，再SetLeft，wxRect的行为就跟CRect的不一样了。

例如对一个(left,top,right,bottom)=(1,1,5,5)的CRect，对应的wxRect是(left,top,width,height)=(1,1,4,4)，我们对其依次执行SetRight(10)和SetLeft(3)，得到CRect是(left,top,right,bottom)=(3,1,10,5)，wxRect是(left,top,width,height)=(3,1,9,4)，换算过来wxRect是(left,top,right,bottom)=(3,1,12,5)，跟得到的CRect不同！原因就是SetLeft的时候，wxRect只改left，没改width，但等于right还是改了，其实是使整个矩形做了偏移；CRect的行为则是修改左边界，右边界不会动，其实是使整个矩形做压缩。真是坑啊！注意到这一点之后，原软件每个连续SetLeft、SetRight，SetTop、SetBottom的地方，都要注意执行的顺序。

鉴于wxRect和CRect以上的差异，其实移植的时候最好的做法是自己写一个封装了wxRect的MyRect类，把wxRect和CRect的差异在MyRect里面做转换。这其实是应用了Adaptor Pattern。

## Mac下wxBitmap在剪切板wxClipboard取回时，Alpha信息丢失

wxWidgets在从剪切板wxClipboard取回图片数据wxBitmapDataObject时，会丢失了透明信息，即Alpha channel。具体来看就是原来用RGBA表示的有透明信息的图片，通过剪切板传递之后，RGBA的A都变成255了。

我通过查看wxWidgets的源码，发现确实是个bug，wxWidgets在Mac这边调用Cocoa接口从剪切板获取像素数据保存为图片时，没有关注Alpha channel。这个我通过修改它源码解决了。详见我提交到官方的这个[ticket](http://trac.wxwidgets.org/ticket/16198)。

## wxDC在Unix下不是线程安全的

wxDC是wxWidgets的绘制上下文，对应于Windows下的CDC。官方资料说了，wxDC在Unix平台下是非线程安全的。这里指的Unix平台具体是GTK(Linux)和OSX(Mac)。所以绘制的时候最好是下wxPaintEvent的响应函数里面做。在子线程做绘制不保证正确，即使是用wxWidgets的wxGuiEnter/wxGuiLeave加锁也是不行。

## 对话框资源的移植

在MFC中，对话框资源都保存在rc文件中。而对应到wxWidgets，每个对话框以xml格式保存成各自的xrc文件，跟rc有一定区别。MFC大部分控件在wxWidgets都能找到。对于对话框资源的移植，我们是用脚本批量完成的，中间一个坑是转换时候对话框和控件的大小在MFC的rc和wxWidgets的xrc不是1:1的，要乘一个比例，1.5左右。

## ALL IN ALL

以上是暂时记得的问题。总的来说，wxWidgets是个强大的跨平台库，用着真心方便，常用的操作也都覆盖了，代码也整洁漂亮，社区也活跃，还是可以放心选用的。而且作为一个开源库，有什么问题都能自己定位自己修改解决，开源万岁～
