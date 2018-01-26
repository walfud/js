---
title: 为何 node 底层同样使用多线程却依然比传统多线程快?
tag:
  - js
date: 2018/01/26
---

[为何 node 底层同样使用多线程却依然比传统多线程快?](https://stackoverflow.com/questions/3629784/how-is-node-js-inherently-faster-when-it-still-relies-on-threads-internally)

Ryan 说: *Threads are expensive and should only be left to the experts of concurrent programming to be utilized*.

node 底层确实使用多线程并发的执行我们代码中的异步函数, 但是它在底层会使用 epoll 等类似技术让多个 io 请求在一个线程上并发执行, 从而减少需要的线程数量. 线程少了, 内核调度开销也少了, 所以速度就快了, 也节省资源.