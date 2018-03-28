---
title: JS运行机制
date: 2017-07-16 17:37:18
tags: [JavaScript, Event-Loop]
---
![event-loop](https://user-images.githubusercontent.com/12566627/30576012-58e0c3d6-9d37-11e7-9c4e-7621fa244a6d.png)
## JavaScript单线程

> 我们常说：js是一门单线程的语言，JS引擎中负责解释和执行JavaScript代码的线程只有一个，即在某个特定的时刻只有特定代码能执行，并阻塞其他代码。
<!--more-->

## Event Loop
Event Loop会有一个或多个任务列队`task queues`，同时任务列队又分为：`macrotask`队列和`microtask`队列。
##### task
`task`会被排定好，于是browser可以按顺序将它带给JavaScript/DOM。在task和task的之间，浏览器或许会渲染更新。比如鼠标点击的回调函数就可以排定一个task，再比如：setTimeOut它会在给定的时间之后，对回调函数排定一个Task，script end是第一个task，而setTimeout是独立的另一个task
##### mircrotask
`mircrotask`经常会被排定在当前主任务执行结束之后立即执行，一旦没有其他JS代码在执行中，microtask队列会立即执行，执行过程中如果有microtask插入，也同时会被执行。⚠️这里注意：microtask必须在当前JS代码运转完后才会被操作，而且会在下一个task之前执行。

我们看下这个例子🌰：
``` bash
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});

console.log('script end');
```
打印顺序为：script start，script end，promise1，promise2，setTimeout

<div class="tip">
*注释*：整个script中代码会在task中执行，执行到setTimeout也放入了task中。而promise.then放入了microtask queue中。所以执行顺序为：取一个task queue中的task，执行之，然后把所有microtask queue顺序执行完，再取task queue中的下一个任务之行，重复这个过程到所有队列都为空
</div>
[Promises / A +规范](http://www.ituring.com.cn/article/66566)中提出了macrotask这个概念，三者的关系为`(task queue = macrotask queue != microtask queue
)`

其中`macrotasks`和`microtasks`分别包括：
> `macrotasks:` script(整体代码)，setTimeout，setInterval，setImmediate，requestAnimationFrame，I/O，UI rendering

> `microtasks:` process.nextTick，Promises，Object.observe，MutationObserver

这张图说明了整个Event Loop过程：
![图片](https://camo.githubusercontent.com/a12555cbe873831500e20955075d50558735a5f8/68747470733a2f2f706963312e7a68696d672e636f6d2f76322d61643161323531636239316433373632353138356134666238373434393466635f622e706e67)

>1、 在任务队列中选择最`oldest`的任务执行

>2、如果任务为空（表示任务队列为空），则跳转到步骤6

>3、将“当前运行的任务”设置为“任务A”

>4、运行“任务A”

>5、将“当前运行的任务”设置为null，删除“任务A”

>6、执行微任务队列
  * a、微任务队列中选择最`oldest`的任务
  * b、如果任务为空跳转到g
  * c、将“当前运行的任务”设置为“任务B”
  * d、运行“任务B”
  * e、将“当前运行的任务”设置为null，删除“任务B”
  * f、在微任务队列中选择下一个最`oldest`的任务，跳转到步骤（b）
  * g、完成微任务队列

>7、跳到步骤1

## 总结
综上所述：
>1、任务按顺序执行，浏览器可能在它们之间`render`
>2、微任务队列中的任务将在当前轮次中运行，而宏任务队列中的任务必须等待下一轮事件循环。


## 参考
[1、Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)
[2、Promise的队列与setTimeout的队列有何关联？](https://www.zhihu.com/question/36972010)
[3、great talk at JSConf on the event loop ](https://www.youtube.com/watch?v=8aGhZQkoFbQ)
[4、Vue源码详解之nextTick](https://github.com/Ma63d/vue-analysis/issues/6)
[5、MDN并发模型与事件循环](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop)
[6、HTML5规范#event-loop](https://html.spec.whatwg.org/multipage/webappapis.html#event-loop)
[7、深入核心，详解事件循环机制](http://www.jianshu.com/p/12b9f73c5a4f)
[8、JavaScript Event Loop 机制详解与 Vue.js 中实践应用](https://zhuanlan.zhihu.com/p/29116364)