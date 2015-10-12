title: 查看所有子域名
date: 2015-09-21 20:39:53
tags: [dns]
categories: others
---

有时候想查看一个域名的所有子域名，纯dig是办不到的。

一般的做法就是穷举，类似像dnspod等DNS域名管理服务，应该有个白名单，所以当我们添加域名的时候，能找到常用的子域名。

不想自己穷举，可以用在线的工具，如: <https://pentest-tools.com/information-gathering/find-subdomains-of-domain>

还找到个工具，看说明挺有效的样子，可以避免被认为恶意攻击: <https://github.com/TheRook/subbrute>
