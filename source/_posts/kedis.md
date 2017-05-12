---
title: 'kedis: 一个更优雅的redis cluster proxy'
date: 2017-05-12 17:57:08
tags: [redis, cluster, proxy, c]
---

# redis cluster简介

Redis的开源代码包含三种运行模式

- Standalone：用户的请求直接访问内存数据并返回。
- Sentinel：作为redis的监控身份运行，监控多个redis实现raft协议完成故障恢复。这个模式下采用了hiredis访问redis。
- Cluster：redis开启了两个端口，一个用于用户访问，一个用于交换gossip协议。Redis之间通过二进制流交换信息。

以下是redis cluster的网络拓扑模型，client直连redis，并实现cluster协议(处理moved ask)。遗憾的是大部分redis客户端没有实现cluster协议。
![redis cluster](/img/kedis/cluster.jpeg)

kedis是一个优雅的redis cluster proxy解决方案，它在代码层级上和db/sentinel/cluster并列，并在事件循环中挂钩。kedis接收client的请求并维护request队列，写入到后端redis的backend并维护callback队列，redis的返回会触发相应的client调用。

# kedis代码结构图

![kedis](/img/kedis/kedis.jpeg)

上图中的proxy和backend两部分是kedis proxy的核心功能，其他为redis自带模块，只做了很少的修改。

# kedis代码演进

第一版kedis我用hiredis作为redis异步客户端。然而深入到hiredis发现这个库为了实现通用性和接口的清晰放弃了性能的最优。举例来说，在read()调用时hiredis会首先把数据读到栈空间，然后feed到read buffer。再比如，每一次调用回调函数时会要求数据被拷贝走，hiredis默认释放掉reply的内存。这个版本没做性能测试。


## memcoy()优化：

先观察一下使用hiredis的kedis内存拷贝次数

- request：tcp协议栈 –> read buffer -> redis object -> hiredis的format函数调用栈转为redis协议字符串 –> write buffer -> tcp协议栈
- reply： tcp协议栈 -> 函数调用栈buffer –> feed到read buffer -> reply object –> 回调函数需拷贝reply内容 -> client write buffer -> tcp协议栈

这种模式下转发一次请求需要拷贝数据5次，转发一次响应需要拷贝数据7次。

优化之后的kedis将redis协议栈代码拿了出来。对其内存管理大做文章，其内存转移模型变为：

- request：tcp协议栈-> read buffer-> redis object->write buffer->tcp协议栈
- reply: tcp协议栈-> read buffer-> redis object/raw string->write buffer->tcp协议栈

优化之后请求和转发都只需要拷贝4次数据。这一次进行性能测试kedis的单核性能已经达到了12w qps，和redis cluster单实例的性能基本一致。

## malloc()调用次数优化

对于mget这样的多个请求key的命令，proxy要做拆分并转发，最后拼接返回给客户端。但是对于hgetll，set这样只有一个key的命令，proxy完全可以做到不感知返回内容，直接转发给客户端。

做完这个优化，在lrange_300的测试环境下，redis一次返回300个元素，kedis吞吐量提升了5倍。

## client回写优化

这部分完全复用了redis的代码。核心思想为以下三点：

- 减少write()调用，每次事件循环将数据写入writer buffer，在调用epoll前调用write()
- 不使用calloc()，writer buffer采用链式结构。
- 少量优先，每次事件循环每个客户端最多写16k数据，避免饿死请求量少的client。

一个新产品面世当然要和老产品作比较，鉴于codis没有对hgetall, lrange这样的多元素返回的命令做优化，我们就拿codis最擅长的get，set命令对比。在下面这个测试报告的基础上可以得出结论：

**kedis对CPU做到了更有效的利用，其单核性能比codis 20核qps高与此同时延时更低。**

 

## 测试报告

kedis: CPU占用100%, qps: 125078

补充一下：kedis是redis的延续，是单线程。

redis-benchmark -h 100.69.89.31 -p 36379 -n 1000000 -d 20 -r 200000 -e -l -c 200 -t get,set

====== SET ======
1000000 requests completed in 7.99 seconds
200 parallel clients
20 bytes payload
keep alive: 1
42.32% <= 1 milliseconds
99.64% <= 2 milliseconds
99.99% <= 3 milliseconds
100.00% <= 3 milliseconds
125078.17 requests per second
 
====== GET ======
1000000 requests completed in 8.05 seconds
200 parallel clients
20 bytes payload
keep alive: 1
48.97% <= 1 milliseconds
99.75% <= 2 milliseconds
99.98% <= 3 milliseconds
100.00% <= 3 milliseconds
124285.37 requests per second
 
codis：分配20个核，占用1500%, qps: 114692

redis-benchmark -h 100.69.89.31 -p 3000 -n 1000000 -d 20 -r 200000 -e -l -c 200 -t get,set

====== SET ======
1000000 requests completed in 8.72 seconds
200 parallel clients
20 bytes payload
keep alive: 1
72.64% <= 1 milliseconds
98.96% <= 2 milliseconds
99.20% <= 3 milliseconds
99.26% <= 4 milliseconds
99.29% <= 5 milliseconds
99.32% <= 6 milliseconds
99.34% <= 7 milliseconds
99.36% <= 8 milliseconds
99.38% <= 9 milliseconds
99.44% <= 10 milliseconds
99.52% <= 11 milliseconds
99.57% <= 12 milliseconds
99.60% <= 13 milliseconds
99.67% <= 14 milliseconds
99.74% <= 15 milliseconds
99.80% <= 16 milliseconds
99.83% <= 17 milliseconds
99.85% <= 18 milliseconds
99.86% <= 19 milliseconds
99.88% <= 20 milliseconds
99.89% <= 21 milliseconds
99.91% <= 22 milliseconds
99.94% <= 23 milliseconds
99.96% <= 24 milliseconds
99.98% <= 25 milliseconds
100.00% <= 26 milliseconds
100.00% <= 27 milliseconds
100.00% <= 27 milliseconds
114692.05 requests per second
 
====== GET ======
1000000 requests completed in 8.71 seconds
200 parallel clients
20 bytes payload
keep alive: 1
71.84% <= 1 milliseconds
99.01% <= 2 milliseconds
99.23% <= 3 milliseconds
99.29% <= 4 milliseconds
99.32% <= 5 milliseconds
99.35% <= 6 milliseconds
99.37% <= 7 milliseconds
99.39% <= 8 milliseconds
99.42% <= 9 milliseconds
99.48% <= 10 milliseconds
99.56% <= 11 milliseconds
99.62% <= 12 milliseconds
99.66% <= 13 milliseconds
99.72% <= 14 milliseconds
99.80% <= 15 milliseconds
99.86% <= 16 milliseconds
99.90% <= 17 milliseconds
99.92% <= 18 milliseconds
99.93% <= 19 milliseconds
99.94% <= 20 milliseconds
99.95% <= 21 milliseconds
99.96% <= 22 milliseconds
99.97% <= 23 milliseconds
99.98% <= 24 milliseconds
99.98% <= 25 milliseconds
99.99% <= 26 milliseconds
100.00% <= 27 milliseconds
100.00% <= 28 milliseconds
100.00% <= 28 milliseconds
114771.03 requests per second
 