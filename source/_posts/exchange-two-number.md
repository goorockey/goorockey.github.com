title: 小记：不用第三个变量交换两数
date: 2011-8-6
tags: programming
categories: programming
---



经典的解法1:    

    a += b; // a变成a+b
    b = a – b;  // b变成原来的a，因为a+b-b = a
    a –= b; // a变成原来的b，因为a+b-a = b
    
存在的问题: 

    a+b太大的话会溢出
    
经典的解法2:    

    a ^= b; // a变成a^b
    b ^= a; // b变成原来的a，因为b^(a^b) = a
    a ^= b; // a变成原来的b，因为a^(a^b) = b
    
存在的问题:

    a、b的值不能超过对方的位数所能表达的范围.

如下面这种情况就会出问题:

    int a = 0x10101010; // 设此时int为4个字节
    short b = 0x15;

这个问题已经老生常谈了，不过今天还是遇到一个问题：

当两个要交换的数其实是同一个变量（更准确的说是同一个内存）时两种解法都会出错，变量被置0。

原因自己推算即可得出。 

所以，我觉得更为保险的方法就是在交换之前先判断a、b是否相等，就既提高效率又防止出错。