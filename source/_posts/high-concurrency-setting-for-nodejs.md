title: Node.js高并发配置
date: 2014-07-20
tags: [programming, nodejs, high concurrency]
categories: programming
---

node.js的异步模型让它很擅长实现IO密集型的系统，但是测试发现，当并发真的上到几W的时候，会有处理不过来的情况。除了从整个系统的设计上改进，还需要修改一些配置。这里总结一下为了让node.js应对高并发，需要做的配置。

<!--more-->

###linux系统配置

修改/etc/sysctl.conf，情况文件内默认的内容，写入以下项，保存后执行`sudo sysctl -p`使配置生效。注意里面的数值要根据具体情况修改。这些修改当然也适用于除node.js以为的应用。

    net.ipv4.ip_local_port_range = 10240 65535
    net.core.rmem_max=16777216
    net.core.wmem_max=16777216
    net.ipv4.tcp_rmem=4096 8738 16777216
    net.ipv4.tcp_wmem=4096 8738 16777216
    net.ipv4.tcp_fin_timeout = 40
    net.ipv4.tcp_tw_recycle = 1
    net.ipv4.tcp_tw_reuse = 1
    net.ipv4.tcp_timestamps = 0
    net.ipv4.tcp_window_scaling = 0
    net.ipv4.tcp_sack = 0
    net.core.netdev_max_backlog = 30000
    net.ipv4.tcp_no_metrics_save=1
    net.core.somaxconn = 65535
    net.ipv4.tcp_syncookies = 0
    net.ipv4.tcp_max_orphans = 262144
    net.ipv4.tcp_max_syn_backlog = 819200
    net.ipv4.tcp_synack_retries = 2
    net.ipv4.tcp_syn_retries = 2
    net.ipv4.tcp_max_tw_buckets = 65535

- `net.ipv4.ip_local_port_range` 可用端口范围，从第一个值到第二个值。默认的"1024 4999"很容易不够。

- `net.ipv4.tcp_tw_reuse` 是否让系统在安全情况下重用TIME_WAIT状态的连接。在高并发情况下，有大量的连接建立和关闭，TIME_WAIT的连接是快要关闭、但资源还没有回收的，像内存、端口都会占用着。

- `net.ipv4.tcp_max_tw_buckets` 维持TIME_WAIT状态最多连接数。当超过这个值时，连接就会立刻关闭，并报错，dmesg可以看到。

- `net.ipv4.tcp_fin_timeout` TIME_WAIT状态的连接回收时的等待时长。

- `net.ipv4.tcp_max_syn_backlog` 最多记录接受到多少SYN。

- `net.ipv4.tcp_rmem` tcp读缓存空间，三个值分别是最小、默认和最大。

- `net.ipv4.tcp_wmem` tcp写缓存空间，三个值分别是最小、默认和最大。

- `net.core.somaxconn` 最大连接数

修改/etc/security/limits.conf，提高文件句柄上限:

    soft nofile 65536
    hard nofile 65536

###socket池

nodejs的http模块内置socket池，默认[最多建立5个socket](http://nodejs.org/api/http.html#http_agent_maxsockets)

    require('http').globalAgent.maxSockets = 40000 # 也可以设成Infinity，无限制
    require('https').globalAgent.maxSockets = 40000

###垃圾回收

nodejs会周期性地向V8发出垃圾回收请求，在并发大的时候经常这样会过多地占用CPU。可以通过启动node时加入`--nouse-idle-notification`选项，关闭这个动作。如:

    node --nouse-idle-notification app.js

###多进程

nodejs虽然异步可以处理轻松地处理大量请求，但单进程单线程的模型在多核下还没有完全利用硬件资源。亲好nodejs原生的[cluster](http://nodejs.org/api/cluster.html)可以很简单地让程序编程多进程。

cluster是prefork模型的，即前面一个master负责总的接受请求，然后均匀地把请求分发给worker，每个worker是一个独立的进程。
例如对于express的应用，在程序入口添加以下代码即可：

    var express = require('express');
    ...

    if (cluster.isMaster) {
      // calculate number of proccesses to fork
      var num_cpus = require('os').cpus().length;
      var num_processes = Math.max(1, num_cpus - 1);

      debug('Master starts with %d processes.', num_processes);
      for (var i = 0; i < num_processes; i++) {
        cluster.fork();
      }

      // Listen for dying processes
      cluster.on('exit', function(worker, code, signal) {
        debug('A process(pid=%s) of master died (%s). Restarting...', worker.process.pid, signal || code);
        cluster.fork();
      });

      return;
    }

    // worker
    var app = express();
    ...

另外还有如[PM2](https://github.com/Unitech/pm2)等外部工具可以让原来单进程的程序变成多进程


###参考资料

- <http://www.oschina.net/translate/optimising-nginx-node-js-and-networking-for-heavy-workloads>

- <http://blog.caustik.com/2012/04/08/scaling-node-js-to-100k-concurrent-connections/>

- <http://engineering.linkedin.com/nodejs/blazing-fast-nodejs-10-performance-tips-linkedin-mobile>

- <https://rtcamp.com/tutorials/linux/sysctl-conf/>
