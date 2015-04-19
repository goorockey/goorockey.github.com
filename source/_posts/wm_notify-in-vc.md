title: VC中对WM_NOTIFY的处理
date: 2011-8-19
tags: [windows, vc]
categories: windows
---



1.Reflect

对于WM_NOTIFY消息：

> 子控件没有Reflect，则由父窗口处理。
> 只要子控件（即消息对应最直接的控件）有了ON_NOTIFY_REFLECT就会被子控件处理,不会被父窗口所处理。

*Reflect的意思就是把消息反射给子控件处理。*

大概的就是这样：

子控件有ON\_NOTIFY\_REFLECT | 父窗口有ON\_NOTIFY       | 效果
--------------------------- | ------------------------ | ----------------------
WM\_NOTIFY未被处理          | WM\_NOTIFY只被父窗口处理 | WM\_NOTIFY只被子控件处理
Y                           | Y                        | WM\_NOTIFY只被子控件处理

<!--more-->

2.EX

ON_NOTIFY和ON_NOTIFY_EX 以及 ON_NOTIFY_REFLECT和ON_NOTIFY_REFLECT_EX

区别是：

> EX版本的处理函数有BOOL型返回值，表示处理的消息是否继续消息传递
> 返回TRUE，表示不继续；
> 返回FALSE，表示继续，即其他控件可以对它做处理。

*则EX的意思就是把消息重新放到消息循环，继续遍历别的CmdTargetObject。*

大概是这样：

子控件ON\_NOTIFY\_REFLECT\_EX返回值 | 父窗口有ON\_NOTIFY | 效果
---------------------------------   | ------------------ | ---
TRUE | N | WM\_NOTIFY只被子控件处理
TRUE | Y | WM\_NOTIFY只被子控件处理
FALSE | N | WM\_NOTIFY只被子控件处理
FALSE | Y | WM\_NOTIFY被子控件和父窗口处理
