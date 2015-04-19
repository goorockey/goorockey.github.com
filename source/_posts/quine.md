title: 自产生程序
date: 2011-8-8
tags: programming
categories: programming
---



自产生程序：不通过任何输入，输出自己代码的程序。

很多语言都可以写出这样的程序。

C版本(只有一行)：   

    char s[] = "char s[] = %c%s%c;int main(){ printf(s, 34, s, 34);return 0;}";int main(){ printf(s, 34, s, 34);return 0;}

虽然是写出来了，但里面还是有很多值得研究的东西的。 

参考资料：<http://www.nyx.net/~gthompso/quine.htm>
