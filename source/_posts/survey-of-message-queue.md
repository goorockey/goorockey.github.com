title: 消息队列方案粗略调研
date: 2014-06-09
tags: [nodejs, programming, redis, queue]
categories: programming
---

最近在做的系统准备加个消息队列，重构成“master-queue-workers”的结构.

感觉现在好多系统都是这个结构。这样master就专心接受用户的请求，把任务放进队列，让workers去处理。master就可以立刻回复用户，而不用等待处理完整个业务才回复。

主流的消息队列方案可以看[这里](http://queues.io/)。

我主要考虑的方案有rabbitMQ、redis、celery、mongodb。

<!--more-->

- `rabbitMQ`： 之前在学erlang的时候就知道rabbitMQ，它在消息或任务队列里面是主流的方案，成熟稳定。同类的还有ZeroMQ等。

    **优点**：成熟放心。

    **缺点**：需要开发和运维都充分掌握，学习和维护成本较redis的高。

- `redis`：内部的列表支持对队列的操作，其中List的blpop是阻塞式的，满足消息队列的要求。

    **优点**：使用和维护简单。自身的内存缓存特点也保证了速度。

    **缺点**：因为不是原生设计成对消息队列的应用，有的队列操作还需要补充实现。这个有现成的库可以帮助解决，但可靠性、成熟度需要确认。

- `celery`：把rabbiMQ、redis、mongodb等作为后端，进一步的封装。最初是python的，现在也有nodejs版

    **优点**：进一步的封装，可以方便把后端切换成不同的方案。python版相对比较成熟。

    **缺点**：nodejs版还不够成熟，有点功能还不支持，比如底层现在指支持rabbitMQ，不支持redis

- `mongodb`: 其实也有人用mongodb的capped collection来做队列，鉴于它在node.js的应用中广泛被使用，它也是一种选择。

    **优点**：如果本来应用就是用mongodb，那可以一并来用

    **缺点**：当并发量大的时候，速度不理想

我最后选择了用redis。因为系统是用node.js写的，所有用到了node的[kue](https://github.com/LearnBoost/kue)库。
