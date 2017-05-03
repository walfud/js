---
title: Macrotask 与 Microtask 核心概念
tag:
  - js
date: 2017/05/03
---

# 问题是什么?
来看一段理解 js task queue 的代码:
```js
setTimeout(function () {
    console.log(5);
}, 0);
setImmediate(function() {
    console.log(6);
});

new Promise(function (resolve) {
    console.log(1);
    resolve();
    console.log(2);
}).then(function () {
    console.log(4);
});

process.nextTick(function() {
    console.log(3);
});
```
执行结果:
```
1
2
3
4
5
6
```

问题在于, 如何解释上述执行顺序. 我们先来捋一捋一些 js 日常开发中不常见的概念.

# 核心概念
### js 引擎模型
从宏观角度讲, js 的执行是单线程的. 所有的异步结果都是通过 "任务队列(Task Queue)" 来调度被调度. 消息队列中存放的是一个个的任务(Task). 规范中规定, Task 分为两大类, 分别是 Macro Task 和 Micro Task, 并且每个 Macro Task 结束后, 都要清空所有的 Micro Task. 这是什么意思呢? 我们后面会用图说明. 现在先来看看规范怎么做的分类.

### Task 分类
* Macrotask 包括: 
  * `setImmediate`
  * `setTimeout`
  * `setInterval`
* Microtask 包括:
  * `process.nextTick`
  * `Promise`
  * `Object.observe`
  * `MutaionObserver`

我们用一段代码说明一下:
```js
setTimeout(function() {
    // Task 2
    ...
}, 0);

process.nextTick(function() {
    // Task 3
    ...
});
```
![](/images/macrotask-microtask/macro_micro.png)

宏观上讲, Macrotask 会进入 Macro Task Queue, Microtask 会进入 Micro Task Queue. 

但从微观实现的角度讲, 引擎都会有三个 Task Queue:

* Macro Task Queue
* Micro Task Queue
* Tick Task Queue

Macrotask 全部存放于 Macro Task Queue 中. 而 Micro Task 被分到了两个队列中. 'Micro Task Queue' 存放 `Promise` 等 microtask. 而 'Tick Task Queue' 专门用于存放 `process.nextTick` 的任务.

然而问题来了, 只有一个事件循环, 那么该调度哪个任务队列呢? 我们来用一段伪码展示:
```js
for (macroTask of macroTaskQueue) {

    // 1. Handle current MACRO-TASK
    handleMacroTask();

    // 2. Handle all NEXT-TICK
    for (nextTick of nextTickQueue) {
        handleNextTick(nextTick);
    }

    // 3. Handle all MICRO-TASK
    for (microTask of microTaskQueue) {
        handleMicroTask(microTask);
    }
}
```
正如 [核心概念](#核心概念) 一开始所说的. '每个 Macro Task 结束后, 都要清空所有的 Micro Task'. 引擎会遍历 Macro Task Queue, 对于每个 Macrotask 执行完毕后都要遍历执行 Tick Task Queue 的所有任务, 紧接着再遍历 Micro Task Queue 的所有任务. 

现在我们回头来看 [问题](#问题是什么?) 中的代码(简化版):
```js
setTimeout(function () {
    console.log(5);
}, 0);

process.nextTick(function() {
    console.log(3);
});
```
结果:
```
3
5
```
都是异步操作, 为什么 `setTimeout` 先执行但是其 task(`console.log(5)`) 却后被调度. 来看看图:
![](/images/macrotask-microtask/schedule.png)


# ref
[Promise的队列与setTimeout的队列有何关联？](https://www.zhihu.com/question/36972010)