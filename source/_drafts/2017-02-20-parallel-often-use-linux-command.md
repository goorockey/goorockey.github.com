title: Linux常用命令并行化
date: 2017-02-20 16:13:03
tags: [linux, shell]
categories: linux
---

平常用的命令，如grep，很多都是单线程or单进程执行的，没有完全发挥现在多核机器的性能。总结了下常见命令的并行化。

### grep


### gzip

`pigz`和`unpigz`分别是`gzip`和`gunzip`的并行化版本，参数基本一致。

    $ pigz test.log
    $ unpigz test.log.gz

### parallel

对于很多没有并行化版本的命令，可以尝试用`parallel`达到并行




### 最后

在生产环境等情况，注意用`ionice`和`nice`限制资源占用，以免影响机器上其他程序的执行。

    $ ionice -c3 nice -n 19 unpigz test.log.gz
