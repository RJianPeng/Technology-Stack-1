# Node 事件循环

<div align="center">
  <img src="https://github.com/TanYJie/Technology-Stack/blob/master/Node/image/Node事件循环.png"/>
</div>

* 阶段概述
  * timer：**定时器**，本阶段执行已经安排的 `setTimeout()` 和 `setInterval()` 的回调函数。
  * pending callbacks：**待定回调**，执行延迟到下一个循环迭代的 I/O 回调。
  * idle, prepare：仅系统内部使用。`process.nextTick()`属于 idle 观察者
  * poll：**轮询**，检索新的 I/O 事件;执行与 I/O 相关的回调（几乎所有情况下，除了关闭的回调函数，它们由计时器和 `setImmediate()` 排定的之外），其余情况 node 将在此处阻塞。
  * check：**检测**，`setImmediate()` 回调函数在这里执行。
  * close callbacks：**关闭的回调函数**，一些准备关闭的回调函数，如：`socket.on('close', ...)`。

## 阶段的详细概述

### 定时器阶段 timer  

　　计时器指定可执行所提供回调的 **阈值**，而不是用户希望其执行的确切时间。计时器回调将尽可能早地运行，因为它们可以在指定的时间间隔后进行调度。但是，操作系统调度或其它回调的运行可能会延迟它们。
  
　　**注意**：**轮询**阶段控制何时定时器执行。
  
### 挂起的回调函数 pending callbacks
　　此阶段对某些系统操作（如 TCP 错误类型）执行回调。例如，如果 TCP 套接字在尝试连接时接收到 ECONNREFUSED，则某些 \*nix 的系统希望等待报告错误。这将被排队以在 挂起的回调 阶段执行。
  
### idle, prepare 观察者
　　`process.nextTick()`属于 idle 观察者，在此处执行。

### 轮询阶段 poll
　　轮询 阶段有两个重要的功能：

  1. 计算应该阻塞和轮询 I/O 的时间。
  2. 然后，处理轮询队列里的事件。

　　当事件循环进入轮询阶段且 **没有计划计时器时**，将发生以下两种情况之一：

  * 如果轮询队列 **不是空的**，事件循环将循环访问其回调队列并同步执行它们，直到队列已用尽，或者达到了与系统相关的硬限制。

  * 如果轮询队列 **是空的**，还有两件事发生：
    * 如果脚本已按 `setImmediate()` 排定，则事件循环将结束 **轮询阶段**，并继续 **检查阶段** 以执行这些计划脚本。
    * 如果脚本尚未按 `setImmediate()` 排定，则事件循环将等待回调添加到队列中，然后立即执行。

　　一旦轮询队列为空，事件循环将检查 **已达到时间阈值的计时器**。如果一个或多个计时器已准备就绪，则事件循环将 **绕回计时器阶段** 以执行这些计时器的回调。

### 检查阶段 check
　　check 阶段执行 setImmediate。
  
### 关闭的回调函数 close callbacks
　　close callbacks 阶段执行 close 事件
  
**注：上面介绍的都是 macrotask 的执行情况，microtask 会在以上每个阶段完成后立即执行。**
