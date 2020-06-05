title: 什么是事件循环（Event Loop）
date: 2020-06-05 15:58:01
tags: [JS, 深入js系列]
categories: JS

## 什么是事件循环（`Event loop`）？

WIKI定义：

> In computer science, the **event loop, message dispatcher, message loop, message pump, or run loop** is a programming construct that waits for and dispatches events or messages in a program.

Event loop是一种程序结构，是实现异步的一种机制。Event loop可以简单理解为：

1. 所有任务都在主线程上执行，形成一个执行栈（execution context stack）。
2. 主线程之外，还存在一个"任务队列"（task queue）。系统把异步任务放到"任务队列"之中，然后主线程继续执行后续的任务。
3. 一旦"执行栈"中的所有任务执行完毕，系统就会读取"任务队列"。如果这个时候，异步任务已经结束了等待状态，就会从"任务队列"进入执行栈，恢复执行。
4. 主线程不断重复上面的第三步。

对JavaScript而言，Javascript引擎／虚拟机（如V8）之外，JavaScript的运行环境（runtime，如浏览器，node）维护了任务队列，每当JS执行异步操作时，运行环境把异步任务放入任务队列。当执行引擎的线程执行完毕（空闲）时，运行环境就会把任务队列里的（执行完的）任务（的数据和回调函数）交给引擎继续执行，这个过程是一个**不断循环**的过程，称为**事件循环**。

**注意：JavaScript（引擎）是单线程的，Event loop并不属于JavaScript本身，但JavaScript的运行环境是多线程／多进程的，运行环境实现了Event loop。**

另外，视频[What the heck is the event loop anyway](http://2014.jsconf.eu/speakers/philip-roberts-what-the-heck-is-the-event-loop-anyway.html) 站在前端的角度，用动画的形式描述了上述过程，可以便于理解。

## 解释Node.js的Event loop

当Node.js启动时，它会初始化event loop，处理提供的代码（代码里可能会有异步API调用，timer，以及`process.nextTick()`），然后开始处理event loop。

下面是node启动的部分相关代码：

```
// node.cc
  {
    SealHandleScope seal(isolate);
    bool more;
    do {
      v8_platform.PumpMessageLoop(isolate);
      more = uv_run(env.event_loop(), UV_RUN_ONCE);

      if (more == false) {
        v8_platform.PumpMessageLoop(isolate);
        EmitBeforeExit(&env);

        // Emit `beforeExit` if the loop became alive either after emitting
        // event, or after running some callbacks.
        more = uv_loop_alive(env.event_loop());
        if (uv_run(env.event_loop(), UV_RUN_NOWAIT) != 0)
          more = true;
      }
    } while (more == true);
  }
```

### Event Loop的执行顺序

下面的示意图展示了一个简化的event loop的操作顺序：

```
   ┌───────────────────────┐
┌─>│        timers         │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     I/O callbacks     │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     idle, prepare     │
│  └──────────┬────────────┘      ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │
│  │         poll          │<─────┤  connections, │
│  └──────────┬────────────┘      │   data, etc.  │
│  ┌──────────┴────────────┐      └───────────────┘
│  │        check          │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└──┤    close callbacks    │
   └───────────────────────┘
```

（图来自[Node.js API](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)）

*图中每个“盒子”都是event loop执行的一个阶段（phase）。*

每个阶段都有一个**FIFO**的回调队列（queue）要执行。而每个阶段有自己的特殊之处，简单说，就是当event loop进入某个阶段后，会执行该阶段特定的（任意）操作，然后才会执行这个阶段的队列里的回调。当队列被执行完，或者执行的回调数量达到上限后，event loop会进入下个阶段。

### Phases Overview 阶段总览

- **timers:** 这个阶段执行`setTimeout()`和`setInterval()`设定的回调。
- **I/O callbacks:** 执行被推迟到下一个iteration的 I/O 回调。
- **idle, prepare:** 仅内部使用。
- **poll:** 获取新的I/O事件；node会在适当条件下阻塞在这里。这个阶段执行几乎所有的回调，除了`close`回调，timer的回调，和`setImmediate()`的回调。
- **check:** 执行`setImmediate()`设定的回调。
- **close callbacks:** 执行比如`socket.on('close', ...)`的回调。

### Phases in Detail 阶段详情

#### timers

一个timer指定一个下限时间而不是准确时间，在达到这个下限时间后执行回调。在指定时间过后，timers会尽可能早地执行回调，但系统调度或者其它回调的执行可能会延迟它们。

注意：技术上来说，**poll** 阶段控制 timers 什么时候执行。

注意：这个下限时间有个范围：`[1, 2147483647]`，如果设定的时间不在这个范围，将被设置为1。

#### I/O callbacks

这个阶段执行一些系统操作的回调。比如TCP错误，如一个TCP socket在想要连接时收到`ECONNREFUSED`,
类unix系统会等待以报告错误，这就会放到 **I/O callbacks** 阶段的队列执行。

#### poll

**poll** 阶段有两个主要功能：

1. 执行下限时间已经达到的timers的回调，然后
2. 处理 **poll** 队列里的事件。

当event loop进入 **poll** 阶段，并且 *没有设定的timers（there are no timers scheduled）*，会发生下面两件事之一：

1. 如果 **poll** 队列不空，event loop会遍历队列并同步执行回调，直到队列清空或执行的回调数到达系统上限；
2. 如果 **poll** 队列为空，则发生以下两件事之一：
   1. 如果代码已经被`setImmediate()`设定了回调, event loop将结束 **poll** 阶段进入 **check** 阶段来执行 **check** 队列（里的回调）。
   2. 如果代码没有被`setImmediate()`设定回调，event loop将阻塞在该阶段等待回调被加入 **poll** 队列，并立即执行。

但是，当event loop进入 **poll** 阶段，并且 *有设定的timers*，一旦 **poll** 队列为空（**poll** 阶段空闲状态）：

```
1. event loop将检查timers,如果有1个或多个timers的下限时间已经到达，event loop将绕回 **timers** 阶段，并执行 **timer** 队列。
```

#### check

这个阶段允许在 **poll** 阶段结束后立即执行回调。如果 **poll** 阶段空闲，并且有被`setImmediate()`设定的回调，event loop会转到 **check** 阶段而不是继续等待。

`setImmediate()`实际上是一个特殊的timer，跑在event loop中一个独立的阶段。它使用`libuv`的API
来设定在 **poll** 阶段结束后立即执行回调。

通常上来讲，随着代码执行，event loop终将进入 **poll** 阶段，在这个阶段等待 incoming connection, request 等等。但是，只要有被`setImmediate()`设定了回调，一旦 **poll** 阶段空闲，那么程序将结束 **poll** 阶段并进入 **check** 阶段，而不是继续等待 **poll** 事件们 （**poll** events）。

#### close callbacks

如果一个 socket 或 handle 被突然关掉（比如 `socket.destroy()`），close事件将在这个阶段被触发，否则将通过`process.nextTick()`触发。

### event loop的一个例子讲述

```
var fs = require('fs');

function someAsyncOperation (callback) {
  // 假设这个任务要消耗 95ms
  fs.readFile('/path/to/file', callback);
}

var timeoutScheduled = Date.now();

setTimeout(function () {

  var delay = Date.now() - timeoutScheduled;

  console.log(delay + "ms have passed since I was scheduled");
}, 100);


// someAsyncOperation要消耗 95 ms 才能完成
someAsyncOperation(function () {

  var startCallback = Date.now();

  // 消耗 10ms...
  while (Date.now() - startCallback < 10) {
    ; // do nothing
  }

});
```

当event loop进入 **poll** 阶段，它有个空队列（`fs.readFile()`尚未结束）。所以它会等待剩下的毫秒，
直到最近的timer的下限时间到了。当它等了95ms，`fs.readFile()`首先结束了，然后它的回调被加到 **poll**
的队列并执行——这个回调耗时10ms。之后由于没有其它回调在队列里，所以event loop会查看最近达到的timer的
下限时间，然后回到 **timers** 阶段，执行timer的回调。

所以在示例里，回调被设定 和 回调执行间的间隔是105ms。

## `setImmediate()` vs `setTimeout()`

`setImmediate()` 和 `setTimeout()`是相似的，区别在于什么时候执行回调：

1. `setImmediate()`被设计在 **poll** 阶段结束后立即执行回调；
2. `setTimeout()`被设计在指定下限时间到达后执行回调。

下面看一个例子：

```
// timeout_vs_immediate.js
setTimeout(function timeout () {
  console.log('timeout');
},0);

setImmediate(function immediate () {
  console.log('immediate');
});
```

代码的输出结果是：

```
$ node timeout_vs_immediate.js
timeout
immediate

$ node timeout_vs_immediate.js
immediate
timeout
```

是的，你没有看错，输出结果是 **不确定** 的！

从直觉上来说，`setImmediate()`的回调应该先执行，但为什么结果随机呢？

再看一个例子：

```
// timeout_vs_immediate.js
var fs = require('fs')

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log('timeout')
  }, 0)
  setImmediate(() => {
    console.log('immediate')
  })
})
```

结果是：

```
$ node timeout_vs_immediate.js
immediate
timeout

$ node timeout_vs_immediate.js
immediate
timeout
```

很好，`setImmediate`在这里永远先执行！

所以，结论是：

1. 如果两者都在主模块（main module）调用，那么执行先后取决于进程性能，即随机。
2. 如果两者都不在主模块调用（即在一个 IO circle 中调用），那么`setImmediate`的回调永远先执行。

那么又是为什么呢？

看`int uv_run(uv_loop_t* loop, uv_run_mode mode)`源码（deps/uv/src/unix/core.c#332）：

```
int uv_run(uv_loop_t* loop, uv_run_mode mode) {
  int timeout;
  int r;
  int ran_pending;

  r = uv__loop_alive(loop);
  if (!r)
    uv__update_time(loop);

  while (r != 0 && loop->stop_flag == 0) {
    uv__update_time(loop);
    //// 1. timer 阶段
    uv__run_timers(loop);
    //// 2. I/O callbacks 阶段
    ran_pending = uv__run_pending(loop);
    //// 3. idle/prepare 阶段
    uv__run_idle(loop);
    uv__run_prepare(loop);

    // 重新更新timeout，使得 uv__io_poll 有机会跳出
    timeout = 0;
    if ((mode == UV_RUN_ONCE && !ran_pending) || mode == UV_RUN_DEFAULT)
      timeout = uv_backend_timeout(loop);

    //// 4. poll 阶段
    uv__io_poll(loop, timeout);
    //// 5. check 阶段
    uv__run_check(loop);
    //// 6. close 阶段
    uv__run_closing_handles(loop);

    if (mode == UV_RUN_ONCE) {
      uv__update_time(loop);
      // 7. UV_RUN_ONCE 模式下会再次检查timer
      uv__run_timers(loop);
    }

    r = uv__loop_alive(loop);
    if (mode == UV_RUN_ONCE || mode == UV_RUN_NOWAIT)
      break;
  }

  if (loop->stop_flag != 0)
    loop->stop_flag = 0;

  return r;
}
```

上面的代码看起来很清晰，一一对应了我们的几个阶段。

+ 1. 首先进入timer阶段，如果我们的机器性能一般，那么进入timer阶段时，1毫秒可能已经过去了（`setTimeout(fn, 0)` 等价于`setTimeout(fn, 1)`），那么`setTimeout`的回调会首先执行。

+ 2. 如果没到一毫秒，那么我们可以知道，在check阶段，`setImmediate`的回调会先执行。

+ 3. 为什么`fs.readFile`回调里设置的，`setImmediate`始终先执行？因为`fs.readFile`的回调执行是在 **poll** 阶段，所以，接下来的 **check** 阶段会先执行 `setImmediate` 的回调。

+ 4. 我们可以注意到，`UV_RUN_ONCE`模式下，event loop会在开始和结束都去执行timer。

## 理解 `process.nextTick()`

直到现在，我们才开始解释`process.nextTick()`。因为从技术上来说，它并不是event loop的一部分。相反的，`process.nextTick()`会把回调塞入`nextTickQueue`，`nextTickQueue`将在当前操作完成后处理，不管目前处于event loop的哪个阶段。

看看我们最初给的示意图，`process.nextTick()`不管在任何时候调用，都会在所处的这个阶段最后，在event loop进入下个阶段前，处理完所有`nextTickQueue`里的回调。

### `process.nextTick()` vs `setImmediate()`

两者看起来也类似，区别如下：

+ 1. `process.nextTick()`立即在本阶段执行回调；

+ 2. `setImmediate()`只能在 **check** 阶段执行回调。

####  文章转载于[here](https://github.com/creeperyang/blog/issues/26) 