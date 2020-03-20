---
title: JavaScript事件循环
date: 2020-03-19 12:24:34
header_img : https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/u=263690368,305441804&fm=26&gp=0.jpg
categories : JavaScript基础
discription : 'Javascript基础之时间循环'
---
# JavaSript中的事件循环机制
JavaScript中的代码运行都是单线程的，需要保持线程中的事件执行就需要有一个事件循环的机制，本文将讲述JavaScript维持事件执行的事件循环机制。由于Node环境与浏览器环境的事件循环有差距，所以本文会分开来讲述。
### 事件循环概念
JavaScript中的事件循环由两个任务队列来组成，分别是宏任务队列(macro-task)与微任务队列(micro-task)，两个任务队列间的执行顺序如下图所示。
![image](https://user-gold-cdn.xitu.io/2018/9/16/165e1427075ec517?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
具体描述：先执行宏任务，然后执行宏任务中产生的微任务，如果微任务中产生宏任务，则将宏任务放入下一轮循环的宏任务队列中，如果微任务中产生微任务，则继续执行新产生的微任务直到所有微任务结束，然后在执行下一轮的宏任务。
## 浏览器中的事件循环
##### 浏览器中的宏任务包括：
- script(整体代码)
- setTimeout
- setInterval
- setImmediate
- I/O
- UI render
##### 浏览器中的微任务包括：
- process.nextTick
- Promise的then与catch
- Async/Await(实际就是promise)
- MutationObserver(html5新特性)
##### 代码分析：

```
console.log('script start')

async function async1() {
    await async2()
    console.log('async1 end')
}
async function async2() {
    console.log('async2 end')
}
async1()

setTimeout(function() {
    console.log('setTimeout')
}, 0)

new Promise(resolve => {
    console.log('Promise')
    resolve()
})
.then(function() {
    console.log('promise1')
    setTimeout(function() {
        console.log('inside-timeout')
        new Promise(resolve => {
            console.log('inside-Promise')
            resolve()
        }).then(function(){
            console.log('inside-promise1')
        })
}, 0)
})
.then(function() {
    console.log('promise2')
})

console.log('script end')
```
输出的结果如下：
1. script start
1. async2 end
1. Promise
1. script end
1. async1 end
1. promise1
1. promise2
1. setTimeout
1. inside-timeout
1. inside-Promise
1. inside-promise1

执行顺序果分析：
 1. 执行整体代码（作为当前宏任务），输出**script start**
 2. 整体代码按顺序执行遇到**async1()**，进入**async1**遇到**await async2()**,**ascyn2()** 即执行函数 **async2**，输出**async2 end**，由于**async2()**没有返回值，所以相当于 **await undefined**,即会将后面的代码放入微任务队列
 宏任务 | 微任务
---|---
无 |  console.log('async1 end')

 3. 遇到**setTimeout()**,将setTimeout的回调压入宏任务队列，目前事件循环宏任务队列与微任务队列如下：

宏任务 | 微任务
---|---
setTimeout |  console.log('async1 end')
4.执行**new Promise()**,输出**Promise**，然后由于执行resolve的缘故，所以会将当前promise的then的回调函数压入微任务。

宏任务 | 微任务
---|---
setTimeout | console.log('async1 end')
无 | Promise.then(第一个then)
5.当前宏任务执行之最后一行，**console.log('script end')**，输出**script end**当前宏任务结束，开始执行当前宏任务所产生的微任务，目前队列如下所示
宏任务 | 微任务
---|---
setTimeout | console.log('async1 end')
无 | Promise.then(第一个then)
6. 执行微任务队列第一个任务**console.log('async1 end')**,输出**async1 end**
7. 继续执行微任务队列中的**Promise.then()**，代码如下

```
.then(function() {
    console.log('promise1')
    setTimeout(function() {
        console.log('inside-timeout')
        new Promise(resolve => {
            console.log('inside-Promise')
            resolve()
        }).then(function(){
            console.log('inside-promise1')
        })
}, 0)
})
```
首先输出**promise1**，然后将settimeout1放入宏队列
宏任务 | 微任务
---|---
setTimeout | Promise.then(第一个then)
setTimeout1 | 无
8.**Promise.then**(第一个then)执行完后遇到第二个**Promise.then**，目前在微任务执行中产生新的微任务则继续执行产生的微任务，输出**promise2**
9.目前微任务队列已清空，则执行宏任务队列中的任务，首先执行**setTimeout**，输出**setTimeout**
10.继续执行**settimeout1**，settimeout1会产生新的微任务，执行完settimeout1后开始执行所产生的微任务。

## Node中的事件循环
Node中的事件循环是指Node.js执行非阻塞I/O操作的机制，与浏览器中的事件循环有所不同。
下面是Node中具体的事件循环过程

```
   ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘
```
- incoming阶段：处理已提供的输入脚本（或丢入 REPL，本文不涉及到），它可能会调用一些异步的 API、调度定时器，或者调用 process.nextTick()，然后开始处理事件循环。
-  timers 阶段: 这个阶段执行 setTimeout(callback) 和 setInterval(callback) 预定的 callback;
- I/O callbacks 阶段: 此阶段执行某些系统操作的回调，例如TCP错误的类型。 例如，如果TCP套接字在尝试连接时收到 ECONNREFUSED，则某些* nix系统希望等待报告错误。 这将操作将等待在==I/O回调阶段==执行;
- idle, prepare 阶段: 仅node内部使用;
- poll 阶段: 获取新的I/O事件, 例如操作读取文件等等，适当的条件下node将阻塞在这里;
- check 阶段: 执行 setImmediate() 设定的callbacks;
- close callbacks 阶段: 比如 socket.on(‘close’, callback) 的callback会在这个阶段执行;

#### Poll阶段（轮询阶段）
##### 轮询 阶段有两个重要的功能：

- 计算应该阻塞和轮询 I/O 的时间。
-  然后，处理 轮询 队列里的事件。
当事件循环进入 轮询 阶段且 没有被调度的计时器时 ，将发生以下两种情况之一：

如果轮询队列 不是空的 ，事件循环将循环访问回调队列并同步执行它们，直到队列已用尽，或者达到了与系统相关的硬性限制。
如果轮询队列是空的 ，还有两件事发生：

如果脚本被 setImmediate() 调度，则事件循环将结束 轮询 阶段，并继续 检查 阶段以执行那些被调度的脚本。

如果脚本 未被 setImmediate()调度，则事件循环将等待回调被添加到队列中，然后立即执行。

一旦 轮询 队列为空，事件循环将检查 已达到时间阈值的计时器。如果一个或多个计时器已准备就绪，则事件循环将绕回（timer）计时器阶段以执行这些计时器的回调。

#### check阶段（检查阶段）
此阶段允许人员在轮询阶段完成后立即执行回调。如果轮询阶段变为空闲状态，并且脚本使用 setImmediate() 后被排列在队列中，则事件循环可能继续到 检查 阶段而不是等待。

setImmediate() 实际上是一个在事件循环的单独阶段运行的特殊计时器。它使用一个 libuv API 来安排回调在 轮询 阶段完成后执行。

通常，在执行代码时，事件循环最终会命中轮询阶段，在那等待传入连接、请求等。但是，如果回调已使用 setImmediate()调度过，并且轮询阶段变为空闲状态，则它将结束此阶段，并继续到检查阶段而不是继续等待轮询事件。