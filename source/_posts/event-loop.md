---
title: 浏览器异步之macrotask与microtask
date: 2017-08-09 10:36:06
---

本文主要是基于浏览器标准(html5)介绍macrotask和microtask. 由于事件循环和执行环境相关所以在node中执行结果稍有不同。  
* 一个线程中，事件循环(event-loop)是唯一的，但是根据任务源(tasks source)做区分的任务队列(tasks queued)可以拥有多个。

* 任务队列又分为macrostask(宏任务)与microtask(微任务)，在最新标准中，它们被分别称为task与jobs。

* macrotask大概包括：script(整体代码), setTimeout, setInterval, setImmediate, I/O, UI rendering。

* 在每个macrotask(宏任务)也就是task执行之间浏览器也许会更新渲染

* 在执行macrotask(宏任务)的时候遇到一个新的macrotasks(宏任务)会将任务分发到对应队列中等待下一次事件循环执行

* microtasks大概包括: process.nextTick, Promise, Object.observe(已废弃), MutationObserver(html5新特性)


* mircrotask在两种情况会执行  
 1在回调函数中函数栈没有正在执行中的其他js
 2在每个task的后面

* 处理 microtask 队列期间，新添加的 microtask 添加到队列的末尾并且也被执行

* 来自不同任务源的任务会进入到不同的任务队列。其中setTimeout与setInterval是同源的。

* 在每一次事件循环(event-loop)中，macrotask 只会提取一个执行，而 microtask 会一直提取，直到 microtask 队列清空。

```
console.log('glob1');
setTimeout(function() {
      setTimeout(function(){
		console.log('test1');
		Promise.resolve().then(function(){
            console.log('test1_promise')
        });
	});
    console.log('timeout1');
    new Promise(function(resolve) {
        console.log('timeout1_promise');
        resolve();
    }).then(function() {
        console.log('timeout1_then')
    });
})

new Promise(function(resolve) {
    console.log('glob1_promise');
    resolve();
    console.log('glob2_promise')
}).then(function() {
    console.log('glob1_then')
}).then(function(){
	Promise.resolve().then(function(){
		console.log('glob1_then_test')
	});
});

// golb1
// glob1_promise
// glob2_promise
// glob1_then
// glob1_then_test
// timeout1
// timeout1_promise
// timeout1_then
// test1
// test1_promise
```
