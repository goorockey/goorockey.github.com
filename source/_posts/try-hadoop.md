title: 玩一下hadoop
date: 2012-10-21 23:47
tags: [hadoop, programming]
categories: hadoop
---

[HOP]: http://code.google.com/p/hop/ "HOP"
[Hadoop]: http://hadoop.apache.org/ "Hadoop"
[HDFS]: http://hadoop.apache.org/docs/stable/hdfs_design.html "HDFS"
[WordCount]:http://hadoop.apache.org/docs/stable/mapred_tutorial.html "WordCount"
[MapReduce]:http://wiki.apache.org/hadoop/HadoopMapReduce "MapReduce"

由于某种原因，今天玩了一下[Hadoop]。正确来说，我是玩[HOP]，一个Hadoop的修改版本。

> The Hadoop Online Prototype (HOP) is a modified version of Hadoop MapReduce that allows data to be pipelined between tasks and between jobs. This can enable better cluster utilization and increased parallelism, and allows new functionality: online aggregation (approximate answers as a job runs), and stream processing (MapReduce jobs that run continuously, processing new data as it arrives). 

就是多了pipeline（流水线）的Hadoop。分布式流水线可以有效加快各jobs在各节点的同步运算。

<!-- more -->

## 准备

我是在linux上弄的，windows下用cygwin也行。

下载HOP压缩包后，看里面的docs就够了，同时src/example还有一些例子。

确保ssh,sshd,rsync,jdk都有了。同时要保证ssh localhost不要输入密码的认证步骤。具体docs/quickstart也有说，可以这样：

    $ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
    $ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

然后是设置jdk的目录，修改conf/hadoop-env.sh中JAVA\_HOME。一般为/usr/lib/jvm/下的某个java目录，我就直接写成/usr/lib/jvm/default-java了。

这时候执行bin/hadoop就会出现帮助信息了。

## 跑例程

Hadoop的文件系统叫[HDFS]（Hadoop distribution filesystem)，是一个分布式文件系统。每份数据都会在多个节点有备份，以容错、修复。所有数据都要先放进HDFS才能Hadoop处理。

Hadoop的分布式体系中，有一个NameNode，是master的角色，负责主控各节点，有多个DataNode，是slave，负责真正存储数据。这些可以在conf/master和conf/slave设置。
同时还有一个JobTracker，负责调度jobs，默认就是NameNode这个主机一起充当NameNode，这个在conf/hadoop-site.xml设置。另外所有DataNode都是TaskTracker，负责执行jobs。具体更多对conf/hadoop-site.xml的配置参看docs/cluster\_setup.html

执行bin/hadoop namenode -format，会创造一个namenode。文件都已某种格式放在/tmp/hadoop-"hostname"那里。

执行bin/start-all.sh会启动hadoop，默认通过http://localhost:50070/可以访问NameNode，http://localhost:50030/可以访问JobTracker。

现在执行一个例子:

    $ mkdir input
    $ cp conf/*.xml input/

    $ bin/hadoop fs -put intput input   # 把当前文件系统input目录复制为HDFS的input
    $ bin/hadoop jar hadoop-*-examples.jar grep input output 'dfs[a-z.]+'  # 执行所有example.jar，后面的是参数

    # 一段时间后，执行完毕 #
    $ bin/hadoop fs -get output output # 把HDFS中的output目录复制为当前文件系统的ouput
    $ cat output/* # 打印结果

    # 或者直接对HDFS操作 #
    $ bin/hadoop fs -ls output
    $ bin/hadoop fs -cat output/*

## WordCount例子

[WordCount]是hadoop中的另一个例子

Hadoop是通过[MapReduce]机制来处理大数据的。Map阶段分割输入的数据，并整合成\<key,value\>的对应关系。每对\<key,value\>对送到Combiner做每个key的整合，当整合出一定数量的\<key,value\>后，\<key,value\>会送到Reducer做处理输出最终的\<key,value\>。

    (input) <k1, v1> -> map -> <k2, v2> -> combine -> <k2, v2> -> reduce -> <k3, v3> (output) 

按照[WordCount]中的代码编辑WordCount.java，然后编译打包生成wordcount.jar:

    $ mkdir wordcount_classes
    $ javac -classpath hadoop-hop-0.2-core.jar -d wordcount_classes WordCount.java
    $ jar -cvf wordcount.jar -C wordcount_classes/ . 


然后自行构造一些要统计的文件，放在input目录下。这时候注意，在执行了上一次例子后，如果想把输入文件还是放在HDFS的input下，要先清空原来的文件:

    $ bin/hadoop fs -rmr input/
    $ bin/hadoop fs -rmr output/

    $ bin/hadoop fs -put input input # 把输入文件目录input重新放到HDFS中
    $ bin/hadoop jar wordcount.jar org.myorg.WordCount input output  # 执行wordcount.jar

    # 执行一段时间后完毕 #

    $ bin/hadoop fs -cat output/*  # 打印结果

## 结语

尝试了一下Hadoop，还有更多有待研究
