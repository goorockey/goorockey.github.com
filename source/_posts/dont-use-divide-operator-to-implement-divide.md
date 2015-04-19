title: 不用除法运算符实现除法及其推广
date: 2011-8-22
tags: programming
categories: programming
---



两个正整数x、y，x是y的倍数，不用除法运算符实现x / y。

1、最简单的方法

循环用x减y，知道x等于0。

    int Div( int x, int y )
    {
        int result = 0;

        while ( x > y )
        {
            x -= y;
            result++;
        }

        return result;
    }

时间复杂度`O(n)`

<!--more-->

2、用移位实现

与很多优化算法相似，用2次幂实现加速。

考虑到x是y的倍数，设x = y * k

因为我们可以用二进制表示任意整数，所以任意整数都可表示成2次幂的和，即：

    k = 2^t1 + 2^t2 + …. + 2^tn;

所以有x = y * (2^t1 + 2^t2 + … + 2^tn)，即我们要的结果就是2^t1 + 2^t2 + … + 2^tn

由此，我们可以先找到一个刚好不大于x的s1 = y\*(2^t1)，即有`y*2^t1 <= x < y*2^(t1+1)`,

然后令x2 = x - s1 = y * (2^t2 + … + 2^tn)，从而继续递归直到xn – sn = 0。

    int Div( int x, int y )
    {
        int i = 1;          // 2次幂计数器
        int product = y;    // 中间乘积，等于y*2^t，即product = y * i

        // 找到刚好不大于x的product = y*i满足y*i <= x < y*(i+1)
        while ( product << 1 <= x )
        {
            i  <<= 1;
            product <<= 1;
        }

        // 递归得到结果
        int result = 0;
        for ( ; x > 0; i >>= 1, product >>= 1 )
        {
            // product自除2来寻找新的product，满足刚好不大于x
            if ( x >= product )
            {
                result += i;     // 累加结果result = 2^t1 + 2^t2 … + 2^t(k-1)
                x -= product;    // 相减得到新的x = y*(2^tk + … + 2^tn)
            }
        }

        return result;
    }

时间复杂度`O(logn)`

3、推广 - 不用开方运算符求幂数：

两个正整数x、y，不用开方运算符求x是y的几次幂。

思想与方法二类似。

    #include "math.h"

    int Extract( int x, int y )
    {
        int i = 1, power = y;
        while ( power * power <= x )
        {
            i <<= 1;
            power *= power;
        }

        int result = 0;
        for ( ; x > 1; i >>= 1, power /= pow( y, i ))
        {
            if ( x >= power )
            {
                 x /= power;
                 result += i;
            }
        }

        return result;
    }

时间复杂度`O(logn)`
