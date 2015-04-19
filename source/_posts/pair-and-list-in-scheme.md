title: Scheme里面的pair和list
date: 2012-09-25
tags: [lisp, scheme]
categories: lisp
---

最近学scheme，总结一下pair和list的区别，主要是两点：


1.list一定是pair，但只有以null（空list）结尾的pair才是list

对于(define list1 (list a b c)),list1表现为(a b c),其实也可以写成(a . (b . ()))。

可以看到list其实就是pair,而且是以null结尾的pair。

对于像(a.(b.(c.d)))这样的连续pair，因为没有以空list结尾，所以不是list

所以有：

    > (define x '(1 2))
    > x
    (1 2)
    > (list? x)
    #t
    > (cons? x)
    #t
    > (cddr x) ; 以null结尾
    ()

    > (define y (cons 1 2))
    > (list? y)
    #f
    > (cons? y)
    #f
    > (set-cdr! y '()) ; 把y的cdr设为null，使y变成list
    > y
    (1)
    > (list? y) ; 变成了list
    #t
    > (cons? y)
    #t

2.pair的显示规则

引用[这里](http://download.plt-scheme.org/doc/html/guide/Pairs__Lists__and_Scheme_Syntax.html)的解释：

> In general, the rule for printing a pair is as follows: use the dot notation always, but if the dot is immediately followed by an open parenthesis, then remove the dot, the open parenthesis, and the	matching close parenthesis. Thus, (0 . (1 . 2)) becomes (0 1 . 2), and (1 . (2 . (3 . ()))) becomes (1 2 3).

大意就是，如果pair的“点”紧接着小括号，则这个点和小括号都可以去掉。

所以(a.(b.c))等价于(a b.c), (a.(b.(c.())))等价于(a b c)。
