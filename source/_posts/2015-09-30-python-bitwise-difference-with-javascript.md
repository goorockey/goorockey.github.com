title: Python按位运算结果跟Js不同
date: 2015-09-30 17:54:00
tags: [python, js]
categories: programming
---

由于python的整型是64位的，js的是32位，所以前后端通信的时候，会出现两边按位运算结果不同导致有问题。

解决是用python中ctypes的c_int32等类型，如:

    [javascript]
    > i = 1443603316765
    > ~i
    -494305310

    [python]
    >>> import ctypes
    >>> i = 1443603316765
    >>> ~i
    -1443603316766
    >>> (ctypes.c_int32(~i).value)
    -494305310
