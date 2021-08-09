title: 面试趣题
date: 2011-8-24
tags: [programming, 面试]
categories: programming
---



### 题一、有一个整数数组，请求出两两之差绝对值最小的值

方法：先排序，再找差值最小的点对。

效率：时间复杂度O(nlogn)

### 题二、平面上N个点，每两个点都确定一条直线，求出斜率最大的那条直线所通过的两个点（斜率不存在的情况不考虑）

方法：根据x排序，用图解枚举所有情况，能证明斜率最大的两点肯定是相邻的两点。

效率：时间复杂度O(nlogn)

### 题三、一棵排序二叉树，令 f=(最大值+最小值)/2，设计一个算法，找出距离f值最近、大于f值的结点

方法：把f插入，中序排序

效率：时间复杂度O(logn)

<!--more-->

### 题四、找出两个字符串中最大公共子字符串

如"abccade","dgcadde"的最大子串为"cad"

*方法一：*

从字符串一中遍历子串，并在字符串二中匹配。时间复杂度为O(n^3)。

*方法二：*

矩阵法：用矩阵表示两字符串，横竖字符相同的格置1，则在45度方向连续1最多的就是所求，时间复杂度O(n^2) 。

### 题五、检查单向链表中是否有环

这算是经典的面试题了，记录一下好的解法和推广。

*方法一：*

操作：把链表反向，当游标指针回到首节点时表示有环，否则无环。

解释：如果有环，把链表反向后，游标指针会从环内回到环外，最后回到首节点。

效率：时间复杂度O(n)，空间复杂度O(1)。

不足：破坏原链表的结构，需要再遍历一次链表来恢复链表结构。

*方法二:*

操作：

两个游标指针,一个慢指针每次移动一个节点，一个快指针每次移动两个节点。如果在快指针遍历到链表结尾前遇到慢指针，则链表有环，否则无环。

解释：

如果有环，当慢指针刚进入环时，设快指针与慢指针的距离为n（距离指慢指针不动是，快指针要经过几次节点达到慢指针），由于快指针每次都追上慢指针一个节点，则两者经过n次后总会相遇。

效率：时间复杂度O(n)， 空间复杂度O(1)。

{% gist 3712175 %}

### 推广一：有环单向链表中环的节点数

操作：

还是用快慢指针，当快慢指针在环内相遇后，两指针继续移动，并对慢指针移动的节点计数。当两指针再次相遇时，计数的结果就是环的节点数。


解释：

还是题一中的思想。设环的节点数为n，当两指针第一次相遇时，可看做两指针的距离为n，则再慢指针再经过n个节点后，两指针会再次相遇，所以慢指针移动的节点数就是环的节点数。

{% gist 3712217 %}

### 推广二、找到有环单向链表中环首节点

如以下有环单向链表：

    1=>2=>3=>4=>5=>6=>7=>9=>4

即第9节点的next指针指向第4节点，则环的首节点为第4节点

操作：
 
    先计算环的节点数n。
    两个前后指针，前指针先移动n个节点，然后两指针一齐移动，每次都只移动一个的节点。
    当两指针相遇时，两指针指向的节点就是所求的环首节点。

解释：因为开始两指针相距n个节点，当后指针刚进入环时，肯定会与前指针在环的首节点相遇。

### 推广三、破坏有环单向链表的环

操作：

在上面的基础上，当找到了环首节点和环内节点数n后，只要从环首节点移动n到达环的尾节点，修改环尾节点的next指针即可。

{% gist 3712226 %}

### 题五、找到有序链表的中位值

常规方法：遍历一次链表得到链表长度length，再遍历一次链表到length/2的位置即为中位值

更好的方法：用两个指针p1、p2，p1每次走两步，p2每次走一步，等到p1到链表尾时，p2所指即为中位值

{% gist 3824662 %}