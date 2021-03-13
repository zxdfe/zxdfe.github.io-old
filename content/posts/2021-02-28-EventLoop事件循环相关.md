---
title: EventLoop事件循环
date: 2021-02-28 23:11:23
# sidebar: auto
tags: 
---

## 为什么JavaScript是单线程的?

JavaScript是一门单线程非阻塞的脚本语言, 作为浏览器脚本语言,JavaScript的主要用途是与用户互动，以及操作DOM。

这决定了它只能是单线程，就是同一时间只能做一件事情，否则会带来很复杂的同步问题。比如，假定JavaScript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器就不知道以哪个为准了.


## 任务队列 (**Event Queue**)

单线程就意味着，所有任务需要排队，前一个任务结束，才会执行后一个任务。如果前一个任务耗时很长，后一个任务就不得不一直等着。但这样就太浪费资源了.


怎么解决呢? JS主线程挂起处于等待中的任务, 先运行后面的任务, 再在适当的时候执行这些挂起的任务.


于是，所有任务可以分成两种，一种是同步任务（synchronous），另一种是异步任务（asynchronous）。


`同步任务`: 立即执行的任务, 同步任务一般会直接进入到主线程中执行. 只有前一个任务执行完毕，才能执行后一个任务；
`异步任务`: 异步执行的任务,不进入主线程、而进入"任务队列"（task queue）的任务，如, Ajax网络请求,seTtimeout等定时函数, 异步任务会通过任务队列的机制(先进先出的机制)来进行协调.


![image.png](https://cdn.nlark.com/yuque/0/2021/png/302528/1614587877626-3f9d770f-8ee8-4811-ab0e-ffc029b40220.png#align=left&display=inline&height=457&margin=%5Bobject%20Object%5D&name=image.png&originHeight=914&originWidth=776&size=544464&status=done&style=none&width=388)


## 事件和回调函数

"任务队列"是一个事件的队列（也可以理解成消息的队列），IO设备完成一项任务，就在"任务队列"中添加一个事件，表示相关的异步任务可以进入"执行栈"了。主线程读取"任务队列"，就是读取里面有哪些事件。


"任务队列"中的事件，除了IO设备的事件以外，还包括一些用户产生的事件（比如鼠标点击、页面滚动等等）。只要指定过回调函数，这些事件发生时就会进入"任务队列"，等待主线程读取。


所谓"回调函数"（callback），就是那些会被主线程挂起来的代码。异步任务必须指定回调函数，当主线程开始执行异步任务，就是执行对应的回调函数。


"任务队列"是一个先进先出的数据结构，排在前面的事件，优先被主线程读取。主线程的读取过程基本上是自动的，只要执行栈一清空，"任务队列"上第一位的事件就自动进入主线程。但是，由于存在后文提到的"定时器"功能，主线程首先要检查一下执行时间，某些事件只有到了规定的时间，才能返回主线程。


## 事件循环 Event Loop

主线程从"任务队列"中读取事件，这个过程是循环不断的，所以整个的这种运行机制又称为Event Loop（事件循环）


同步和异步任务分别进入不同的执行环境，同步的进入主线程，即主执行栈，异步的进入任务队列。主线程内的任务执行完毕为空后，会去任务队列读取对应的任务，推入主线程执行。 上述过程的不断重复就是我们说的 Event Loop (事件循环)。

在事件循环中，每进行一次循环操作称为tick


[http://latentflip.com/loupe/](http://latentflip.com/loupe/)
栈是一种结构化的内存，遵循LIFO（先进后出）的原则
队列也是一种结构化的内存，遵循FIFO（先进先出）的原则


**宏任务:**

- script( 整体代码)、为什么??
- setTimeout、
- setInterval、
- I/O、
- UI Render
- setImmediate(Node.js 环境)



**微任务:**

- promise.then() || catch()
- Async/Await(实际就是promise)
- MutaionObserver()
- process.nextTick(Node.js 环境)



执行一个宏任务-->清空微任务-->UI Render  ==> 执行一个宏任务-->清空微任务-->UI Render
(UI Render不一定执行, 是否有UI变动)


宏任务和微任务在不同的任务队列中

ES6 规范中，MicroTask 称为 jobs，MacroTask 称为 task
宏任务是由宿主发起的，而微任务由JavaScript自身发起。

## Node.js的Event Loop

Node.js也是单线程的Event Loop，但是它的运行机制不同于浏览器环境


![image.png](https://cdn.nlark.com/yuque/0/2021/png/302528/1614593172374-ee8ff6eb-ca0b-48df-8b2e-6fb4ea44a094.png#align=left&display=inline&height=316&margin=%5Bobject%20Object%5D&name=image.png&originHeight=316&originWidth=800&size=55988&status=done&style=none&width=800)

:::tips
（1）V8引擎解析JavaScript脚本。
（2）解析后的代码，调用Node API。
（3）[libuv库](https://github.com/joyent/libuv)负责Node API的执行。它将不同的任务分配给不同的线程，形成一个Event Loop（事件循环），以异步的方式将任务的执行结果返回给V8引擎。
（4）V8引擎再将结果返回给用户。
:::


#### JS

```js
console.log(0);

setTimeout(function () {
    console.log(1);
});

new Promise(function(resolve,reject){
    console.log(2)
    resolve(3) 
}).then(function(val){
    console.log(val);
})

console.log(4);

// 0--> 2 --> 4 --> 3 --> 1   // resolve(3) --> then() val==3, 如果不resolve,then不执行
```

```javascript
async function async1(){
	console.log('async1 start')
  await async2()           // await async2()中, `async`可以看作放在promise里面, await async2后面的代码,可以看作放在then()里面
  console.log('async1 end');
}
async function async2(){  //微任务
  console.log('async2');
}

console.log('script start')

setTimeout(function(){ console.log('setTimeout')},0);
async1()

new Promise(resolve => {
  console.log('Promise'); 
  resolve(); // undefined
}).then(() => {
  console.log('promise1') //微任务
}).then(() => {
  console.log('promise2') //微任务
})
console.log('script end')

// script start ->async1 start-> async2->promise->script end->async1 end -> promise1 -> promise2 -> setTimeout
```

```javascript
async function async1(){
  await async2()
  console.log('async1 end');
}
async function async2(){  //微任务
  console.log('async2 end');
}

async1(); //宏任务

setTimeout(function(){ console.log('setTimeout')},0); //宏任务

new Promise(resolve => {
  console.log('Promise'); 
  resolve();
}).then(() => {
  console.log('promise1') //微任务
}).then(() => {
  console.log('promise2') //微任务
})

// 我的：Promise -> async1 end -> async2 end -> promise1 -> promise2 -> setTimeout
// 正确：async2 end -> Promise -> async1 end ->  promise1 -> promise2 -> setTimeout
```

---

## 参考

-  [深入浅出Javascript事件循环机制](https://zhuanlan.zhihu.com/p/26229293)
-  [JavaScript 运行机制详解：再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)
-  [JS事件循环](https://juejin.cn/post/6844903577052250119)
-  [深入理解 JavaScript 事件循环 event loop](https://www.cnblogs.com/dong-xu/p/7000163.html)
- [译]  [深入理解 JavaScript 事件循环 task and microtask](https://www.cnblogs.com/dong-xu/p/7000139.html)
-  [深入理解JavaScript事件循环机制](https://www.cnblogs.com/yugege/p/9598265.html)
-  [【JS】深入理解事件循环,这一篇就够了](https://zhuanlan.zhihu.com/p/87684858)
-   [在 JavaScript 中通过 queueMicrotask() 使用微任务](https://developer.mozilla.org/zh-CN/docs/Web/API/HTML_DOM_API/Microtask_guide#%E4%BB%BB%E5%8A%A1_vs_%E5%BE%AE%E4%BB%BB%E5%8A%A1)
-  [浏览器事件循环原理](https://juejin.cn/post/6844903983287549965)
-  [JavaScript 异步与事件循环](https://juejin.cn/post/6844903711106400264)
-  [事件循环机制Event loop](https://juejin.cn/post/6884987711074091016)
-  [JavaScript 事件循环机制](https://juejin.cn/post/6844903887099412488)
-  [微任务、宏任务与Event-Loop](https://juejin.cn/post/6844903657264136200#heading-9)

---

- [Call Stack（调用栈）是什么？](https://zhuanlan.zhihu.com/p/71168084)
- [Call stack（调用栈）](https://developer.mozilla.org/zh-CN/docs/Glossary/Call_stack)
- [https://www.html.cn/archives/10216](https://www.html.cn/archives/10216)
- [深入理解Javascript之Callstack&EventLoop](https://www.jianshu.com/p/735ee3d12a43)

---

-  [javascript中script整体代码属于宏任务怎么理解](https://segmentfault.com/q/1010000023206213?utm_source=tag-newest)
- [https://www.jianshu.com/p/bfc3e319a96b](https://www.jianshu.com/p/bfc3e319a96b)                                           
<!-- - async await [https://segmentfault.com/a/1190000007535316]( -->
