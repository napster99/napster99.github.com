---
layout: post
title:  NodeJS的代码调试和性能调优
categories : [NodeJS]
tags: [NodeJS]
---
# NodeJS的代码调试和性能调优  (2016-06-21)

###nodejs debug <- keywords

[原文地址](https://nodejs.org/api/debugger.html#debugger_commands_reference/)

###命令行调试
我们平时的调试方式：

* 在某个需要输入的地方输入 console.log()，打印调试结果
* 引入 asserts 模块，对调试区域进行 debug

这两种调试方式，都需要我们显式将各种debug信息嵌入到我们的业务逻辑代码中，而熟悉了命令行调试之后，我们可以更好地开启自己的调试之旅。NodeJS给我们提供了Debugger模块，内建客户端，通过TCP将命令行的输入传送到内建模块以达到调试的目的。

在启动文件时，添加第二个参数 debug：  
```
➜  $ node debug proxy2.js
< Debugger listening on port 5858
debug> . ok
break in proxy2.js:1
> 1 var http = require('http');
  2 var net = require('net');
  3 var url = require('url');
debug>
```

>调试代码的时候，存在两种状态，一个是操作调试的位置，比如下一步，进入函数，跳出函数等，此时为debug模式；
>另一个是查看变量的值，比如进入循环中，想查看循环计数器i的值，此时为repl（read-eval-per-line）状态，在debug模式下输入repl即可进入状态；  

```
debug> repl
Press Ctrl + C to leave debug repl
>http
print something about http
>
```
按下 Ctrl+C 可以从 repl 状态回到 debug 状态下，我们也不需要记忆 debug 状态下有多少调试命令，执行 help 即可：
```
debug> help
Commands: run (r), cont (c), next (n), step (s), out (o), backtrace (bt), setBreakpoint (sb), clearBreakpoint (cb),
watch, unwatch, watchers, repl, restart, kill, list, scripts, breakOnException, breakpoints, version
```
相关的命令不算很多：

>命令  解释
>cont, c 进入下一个断点
>next, n 下一步
>step, s 进入函数
>out, o  跳出函数
>setBreakpoint(), sb() 在当前行设置断点
>setBreakpoint(line), sb(line) 在 line 行设置断点

上面几个是常用的，更多命令可以戳[这里](https://nodejs.org/api/debugger.html#debugger_commands_reference)。

###NodeJS的调试原理
我们平时开发都使用 IDE 工具，实际上很多 IDE 工具已经集成了 NodeJS 的调试工具，比如 Eclipse、webStorm 等等，他们的原理依然是利用 Nodejs 的 Debugger 内建模块，在这个基础上进行了封装。
细心的同学会发现，当我们使用 debug 参数打开一个 node 文件时，会输出这样一行文案：
```
Debugger listening on port 5858
```
可以访问下 http://localhost:5858，会看到：
>Type: connect  
>V8-Version:3.28.71.19   
>Protocol-Version:1  
>Embedding-Host:node v0.12.7  
>Content-Length:0  

它告诉我们 nodejs 在打开文件的时候启动了内建调试功能，并且监听端口 5858 过来的调试命令。除了在命令行中直接调试之外，我们还可以通过另外两种方式去调试这个代码：

* node debug <URL>,通过URI连接调试，如node debug localhost:5858
* node debug -p <pid>, 通过PID链接调试

如果我们使用--debug 参数打开文件：
```->$ node --debug proxy2.js```
此时nodejs不会进入到命令行模式，而是直接执行代码，但是依然会开启内建调试功能，这就意味着我们具备了远程调试nodejs代码的能力，使用--debug参数打开服务器的nodejs文件，然后通过：
```->$ node debug <服务器IP>:<调试端口，默认5858>```
可以在本地远程调试nodejs代码。不过这里需要区分下，--debug和--debug-brk,前者会执行所有的代码，一般是在监听事件的时候使用，而后者，不会执行代码，需要等到外部调试接入后，进入代码区
默认端口号是5858，如果这个端口被占用了，程序会递增端口号，我们也可以指定端口号：
```
-> node node-debug-brk=8787 proxy2.js
Debugger listening on port 8787
```

###更多的调试方式
####node-inspector
nodejs提供的内建调试十分强大，它告诉v8,在执行代码的时候中断程序，等待开发者操作代码的执行程度，我们熟知的node-inspector也是用的这个原理。
```->$ node-inspector --web-port 8080 --debug-port 5858```
这里的--web-port是chrome devtools的调试页面地址端口，--debug-port 为nodejs启动的内建debug端口，我们可以在http://localhost:8080/debug?port=5858打开页面，调试使用--debug(-brk)参数打开的程序。

还有一个种是IDE调试，我这里就不介绍了。


####发现程序的问题

上面介绍了nodejs调试需要掌握的几个基本技能，掌握起来很快入手，但还是自己需要自己动手去实践下。

Nodejs相比Java、PHP这些老牌语言来说，周边设施还是有所欠缺的，如性能分析工具和监控工具等，加上它的单线程运行特性，在大型系统中，很容易让cpu或内存达到瓶颈，从而导致程序崩溃。一旦发现程序cpu负载过高，内存飙高时，我们该如何深入排查nodejs代码存在的问题呢？

首先分析下，内存飙高的几个可能存在的因素：

* 缓存，很多人把缓存当内存用，比如使用一个对象存储用户的session信息
* 闭包，作用域没有被释放掉
* 生产者和消费者存在速度差，比如数据库忙不过来，Query队列堆积


CPU过高的几个可能存在的因素：

* 垃圾回收频率过高、量过大，这一般是因为内存或者缓存暴涨导致的
* 密集型的长轮询计算，比如大量遍历文件夹、大量计算等

这些问题是最让人头疼的，一个项目几十上百个文件，收到这些警报如果没有经验，根本无从下手排查。
最直接的手段就是分析 GC 日志，因为程序的一举一动都会反馈到 GC 上，而上述问题也会一一指向 GC，如：我要说话

* 内存暴涨，尤其是 Old Space 内存的暴涨，会直接导致 GC 的次数和时间增长
* 缓存增加，导致 GC 的时间增加，无用遍历过多
* 密集型计算，导致 GC Now Space次数增加

这里需要稍微插一段，NodeJS 的内存管理和垃圾回收机制。我要说话

V8 的内存分为 New Space 和 Old Space，New Space 的大小默认为 8M，Old Space 的大小默认为 0.7G，64位系统这两个数值翻倍。我要说话

对象的生命周期是：首先进入 New Space，在这里，New Space 被平均分为两份，每次 GC 都会将一份中的活着的对象复制到另一份，所以它的空间使用率是 50%，这个算法叫做 Cheney 算法，这个操作叫做 Scavenge。过一段时间，如果 New Space 中的对象还活着，会被挪到 Old Space 中去，GC 会每隔一段时间遍历 Old Space 中死掉的对象，然后整理碎片（这里有两种模式 mark-sweep 和 mark-compact，不祥述）。上面提到的”死掉“，指的是对象已经没有被引用了，活着说被引用的次数为零了。我要说话

知道这些之后，我们就好分析问题了，如果缓存增加（比如使用对象缓存了很多用户信息），GC 是不知道这些缓存死了还是活着的，他们会不停地查看这个对象，以及这个对象中的子对象是否还存活，如果这个对象数特别大，那么 GC 遍历的时间也会特别长。当我们进行密集型计算的时候，会产生很多中间变量，这些变量往往在 New Space 中就死掉了，那么 GC 也会在这里多次地进行 New Space 区域的垃圾回收。






