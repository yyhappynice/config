---
title: 实现 Promise
date: 2017-10-23 20:43:26
tags:
---
![Promise](https://user-images.githubusercontent.com/12566627/31977916-8a3f7322-b971-11e7-8b87-1c5906bf64ae.png)

## 背景知识
JavaScript引擎是基于单线程（Single-threaded）事件循环的概念构建的，Promise则是异步编程的解决方案，在这之前传统的解决方式为`事件模型`、`回调模式`，ES6 将其写进了语言标准，统一了用法，原生提供了Promise对象。
<!--more-->
## Promise 基本使用
ES6 规定，Promise对象是一个构造函数，用来生成Promise实例。
```js
let promise = new Promise(function(resolve, reject) {
  /* ... some code */
  http.get(url, function(results) {
    if (/* 异步操作成功 */){
      resolve(value)
    } else {
      reject(error)
    }
  })
})

promise.then(function(value) {
  /* success */
}, function(error) {
  /* failure */
});
```
1、首先，Promise构造函数接受一个函数作为参数处理promise状态变化，该函数的两个参数分别是`resolve`和`reject`。它们是两个函数，由 JavaScript 引擎提供，不用自己部署。
2、`resolve`函数的作用是，将Promise对象的状态从__未完成__变为__成功__（即从 `pending` 变为 `resolved`），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；
3、`reject`函数的作用是，将Promise对象的状态从__未完成__变为__失败__（即从 `pending` 变为 `rejected`），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去；
4、`then`两个参数都是可选的，分别在完成、拒绝时做出响应，`catch`方法相当于`.then(null, rejection)`错误回调

## 实现Promise
首先,了解一下`Promise`生命周期
![promise](https://mdn.mozillademos.org/files/8633/promises.png)
我们知道Promise实际上是一个对象，我们运行下面的代码。
```js
typeof new Promise((resolve, reject) => {}) === 'object' <!--true-->
```
下面简单实现一下：
```js
class Promise {
  constructor(Fn) {
    if(Object.prototype.toString.call(Fn) !== '[object Function]') { /*传入必须为function*/
      throw new TypeError('Pass function object to create a Promise object')
    }

    this.status = "pending" /*初始化状态*/
    this.promiseResolvedArray = [] /*Promise resolve时的回调数组*/
    this.promiseRejectedArray = [] /*Promise reject时的回调数组*/

    this.onResolve = this.onResolve.bind(this)
    this.onReject = this.onReject.bind(this)
    Fn(this.onResolve, this.onReject)
  }

  then(onResolve, onReject) {
    this.promiseResolvedArray.push(onResolve)
    this.promiseRejectedArray.push(onReject || null)
    return this
  }

  catch(onReject) {
    return this.then(null, onReject)
  }

  onResolve(value) {
    if(this.status === 'pending') {
      this.status = 'resolved'
    } else {
      throw new TypeError('You have to use pending state to convert')
    }

    let storedValue = value /*增加结果传递*/

    try {
      this.promiseResolvedArray.forEach((nextFunction) => {
        storedValue = nextFunction(storedValue)
      })
    } catch (error) {
      throw new TypeError('error')
    }
  }

  onReject(error) {
    if(this.status === 'pending') {
      this.status = 'rejected'
    } else {
      throw new TypeError(error)
    }

    let storedValue = error /*增加结果传递*/

    try {
      this.promiseRejectedArray.forEach((nextFunction) => {
        storedValue = nextFunction(storedValue)
      })
    } catch (error) {
      throw new TypeError('error')
    }
  }
}
```
## 执行
<a class="jsbin-embed" href="//jsbin.com/buwewej/1/edit?js,console" target="_blank">JS Bin on jsbin.com</a>

## 参考
[1、Promises/A+规范](http://www.ituring.com.cn/article/66566)
[2、MDN Promises](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)
[3、Promise的简单实现](http://imweb.io/topic/59df35f3682534f127a79a5c)
[4、剖析 Promise 之基础篇](https://tech.meituan.com/promise-insight.html)
[5、剖析Promise内部结构](https://github.com/xieranmaya/blog/issues/3)
