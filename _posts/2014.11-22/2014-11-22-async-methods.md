---
layout: post
title: nodejs Asyncx详解
categories : [NodeJS]
tags: [NodeJS]
---  
**1、Async介绍** 

Async是一个流程控制工具包，提供了直接而强大的异步功能。基于Javascript为Node.js设计，同时也可以直接在浏览器中使用。
Async提供了大约20个函数，包括常用的 map, reduce, filter, forEach 等，异步流程控制模式包括，串行(series)，并行(parallel)，瀑布(waterfall)等。

**2、Async函数介绍**  
基于async的0.2.9版本。

async主要实现了三个部分的流程控制功能：

集合: Collections
流程控制: Control Flow
工具类: Utils   

**1). 集合: Collections**  
each: 如果想对同一个集合中的所有元素都执行同一个异步操作。  
map: 对集合中的每一个元素，执行某个异步操作，得到结果。所有的结果将汇总到最终的callback里。与each的区别是，each只关心操作不管最后的值，而map关心的最后产生的值。  
filter: 使用异步操作对集合中的元素进行筛选,   需要注意的是，iterator的callback只有一个参数，只能接收true或false。  
reject: reject跟filter正好相反，当测试为true时则抛弃  
reduce: 可以让我们给定一个初始值，用它与集合中的每一个元素做运算，最后得到一个值。reduce从左向右来遍历元素，如果想从右向左，可使用reduceRight。  
detect: 用于取得集合中满足条件的第一个元素。  
sortBy: 对集合内的元素进行排序，依据每个元素进行某异步操作后产生的值，从小到大排序。  
some: 当集合中是否有至少一个元素满足条件时，最终callback得到的值为true，否则为false.  
every: 如果集合里每一个元素都满足条件，则传给最终回调的result为true，否则为false  
concat: 将多个异步操作的结果合并为一个数组。  

**2). 流程控制: Control Flow**    
series: 串行执行，一个函数数组中的每个函数，每一个函数执行完成之后才能执行下一个函数。  
parallel: 并行执行多个函数，每个函数都是立即执行，不需要等待其它函数先执行。传给最终callback的数组中的数据按照tasks中声明的顺序，而不是执行完成的顺序。  
whilst: 相当于while，但其中的异步调用将在完成后才会进行下一次循环。  
doWhilst: 相当于do…while, doWhilst交换了fn,test的参数位置，先执行一次循环，再做test判断。  
until: until与whilst正好相反，当test为false时循环，与true时跳出。其它特性一致。  
doUntil: doUntil与doWhilst正好相反，当test为false时循环，与true时跳出。其它特性一致。  
forever: 无论条件循环执行，如果不出错，callback永远不被执行。  
waterfall: 按顺序依次执行一组函数。每个函数产生的值，都将传给下一个。  
compose: 创建一个包括一组异步函数的函数集合，每个函数会消费上一次函数的返回值。把f(),g(),h()异步函数，组合成f(g(h()))的形式，通过callback得到返回值。  
applyEach: 实现给一数组中每个函数传相同参数，通过callback返回。如果只传第一个参数，将返回一个函数对象，我可以传参调用。  
queue: 是一个串行的消息队列，通过限制了worker数量，不再一次性全部执行。当worker数量不够用时，新加入的任务将会排队等候，直到有新的worker可用。  
cargo: 一个串行的消息队列，类似于queue，通过限制了worker数量，不再一次性全部执行。不同之处在于，cargo每次会加载满额的任务做为任务单元，只有任务单元中全部执行完成后，才会加载新的任务单元。  
auto: 用来处理有依赖关系的多个任务的执行。  
iterator: 将一组函数包装成为一个iterator，初次调用此iterator时，会执行定义中的第一个函数并返回第二个函数以供调用。  
apply: 可以让我们给一个函数预绑定多个参数并生成一个可直接调用的新函数，简化代码。  
nextTick: 与nodejs的nextTick一样，再最后调用函数。  
times: 异步运行,times可以指定调用几次，并把结果合并到数组中返回  
timesSeries: 与time类似，唯一不同的是同步执行  

**3). 工具类: Utils**    
memoize: 让某一个函数在内存中缓存它的计算结果。对于相同的参数，只计算一次，下次就直接拿到之前算好的结果。  
unmemoize: 让已经被缓存的函数，返回不缓存的函数引用。  
log: 执行某异步函数，并记录它的返回值，日志输出。  
dir: 与log类似，不同之处在于，会调用浏览器的console.dir()函数，显示为DOM视图。  
noConflict: 如果之前已经在全局域中定义了async变量，当导入本async.  js时，会先把之前的async变量保存起来，然后覆盖它。仅仅用于浏览器端，在nodejs中没用，这里无法演示。  


演示代码：[https://github.com/napster99/code-fragment/tree/master/nodejs-async/](https://github.com/napster99/code-fragment/tree/master/nodejs-async/)
