title: CentOS下安装Apache+php+mysql+tomcat
date: 2011-11-12
tags: [linux, centos, apache, php, mysql, tomcat]
categories: linux
---

## 一、安装及配置Apache+php+mysql

1.安装Apache+php+mysql

- 安装Apache+php+Mysql，php连接mysql的组件

    yum -y install httpd php mysql mysql-server php-mysql

- 安装mysql扩展

    yum -y install mysql-connector-odbc mysql-devel libdbi-dbd-mysql

- 安装php的扩展

    yum -y install php-gd php-xml php-mbstring php-ldap php-pear php-xmlrpc

- 安装apache扩展

    yum -y install httpd-manual mod_ssl mod_perl mod_auth_mysql

<!--more-->

或者一次性粘贴安装:

    yum -y install httpd php mysql mysql-server php-mysql httpd-manual mod_ssl mod_perl mod_auth_mysql php-mcrypt php-gd php-xml php-mbstring php-ldap php-pear php-xmlrpc mysql-connector-odbc mysql-devel libdbi-dbd-mysql

2.配置Apache+php+mysql

- 设置apache为自启动 chkconfig httpd on

- mysql服务 chkconfig –-add mysqld

- mysqld服务 chkconfig mysqld on

- 自启动 httpd 服务 service httpd start

- 自启动mysqld服务 service mysqld start

## 二、安装和配置Tomcat：

1.安装JDK：

为了默认使用Sun的javac作为Java的编译器，首先删除CentOS系统默认的Java编译器--gcj。

- 查看:

        [root@localhost ~ ]#rpm –qa |grep gcj
        java-1.5.0-gcj-1.5.0.0-29.1.el6.i686
        libgcj-4.4.4-13.el6.i686

- 卸载

        [root@localhost ~ ]#rpm –e java-1.5.0-gcj-1.5.0.0-29.1.el6.i686 --nodeps
        [root@localhost ~ ]#rpm –e libgcj-4.4.4-13.el6.i686 --nodeps

- 检测

        [root@localhost ~]# java --version

会出现

        -bash: /usr/bin/java: No such file or directory

表示卸载成功

- 安装jdk

从Jdk官网下载安装包，如:`jdk-6u27-linux-i586-rpm.bin`

    P.S.
    由于我的CentOS没有图形界面，下载不方便，
    我是先在Windows上访问JDK官网下载安装包，
    然后再用Winscp传到CentOS的

比如安装包保存在/opt/tmp

跳到该目录添加可执行的权限，并执行

    chmod 777 jdk-6u27-linux-i586-rpm.bin
    ./jdk-6u27-linux-i586-rpm.bin

- 添加环境变量

vim /etc/profile

添加以下内容：

    export JAVA_HOME=/usr/java/jdk1.6.0_27
    export JAVA_BIN=/usr/java/jdk1.6.0_27/bin
    export PATH=$PATH:$JAVA_HOME/bin
    export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

保存后，执行java -version 如果有类似以下显示，则表示安装成功：

    java version "1.6.0_27"

2.安装Tomcat：

- 从Tomcat官网下载 安装包，如：apache-tomcat-7.0.22.tar.gz

- 把该压缩包拷贝到/usr/local

        cp apache-tomcat-7.0.22.tar.gz /usr/local

- 跳转到/usr/local，并解压压缩包

        cd /usr/local
        tar -zxvf apache-tomcat-7.0.22.tar.gz

- 把解压出来的目录改名为tomcat,并删除拷贝过来的压缩包

        rm apache-tomcat-7.0.22 tomcat

- 执行/usr/local/tomcat/bin/startup.sh ，自动添加环境变量，

- 测试

访问<http://localhost:8080>，出现tomcat默认页面，则表示tomcat安装成功

3.配置Tomcat为开机自启动：

- 添加开机daomon脚本

把/usr/local/tomcat/bin/catalina.sh拷贝到/etc/init.d，并命名为tomcat

    cp /usr/local/tomcat/bin/catalina.sh /etc/init.d/tomcat

- 在/etc/init.d/tomcat添加内容：

        #!/bin/sh
        # chkconfig: 2345 10 90
        # description:Tomcat service
        # Licensed to the Apache Software Foundation (ASF) under one or more

        ……

        # $Id: catalina.sh 1073891 2011-02-23 19:23:59Z markt $
        # ------------------------------------------------------------------------

        CATALINA_HOME=/opt/tomcat 
        JAVA_HOME=/opt/jdk1.6.0_23

        ……

- 添加tomcat服务

    chkconfig --add tomcat
    service tomcat stop
    service tomcat start
    chkconfig tomcat on

搞定！！
