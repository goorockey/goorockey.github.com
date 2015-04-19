title: STL中用copy和ostream_iterator输出一个数组
date: 2011-8-2
tags: [cpp, stl]
categories: cpp
---



    #include <iostream>
    #include <iterator>
    #include <algorithm>

    int a[] = { 335, 33, 98, 39, 54, 24, 3 };
    int nSize = sizeof(a) / sizeof(a[0]);    

    // 输出数组a到标准输出，同时每个元素都以空格为结束（最后一个元素后面也会有空格）
    std::copy(a, a + nSize, std::ostream_iterator<int>(std::cout, " ")); 
    
结果：335 33 98 39 54 24 3

一个字：妙！！(^0^)/ 
