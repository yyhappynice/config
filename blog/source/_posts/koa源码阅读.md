---
title: koa源码阅读
date: 2017-08-23 19:12:37
tags: node
---
![koa](https://user-images.githubusercontent.com/12566627/30463808-91c60fb0-9a00-11e7-949c-5a9260cc6bb2.png)

## koa 源码结构
首先，查看代码结构，在 lib 文件夹内，包括 4 个文件
* application.js 包含 Application 构造函数以及use、listen、callback等原型方法
* context.js 定义 context 对象, 传入中间件的上下文对象
* request.js 定义 req 的请求对象，包含请求相关的一些属性的get、setter
* response.js 定义 res 的响应对象，包含响应相关的一些属性的get、setter
<!--more-->

## Application的模块
可以在package.json中看到main: lib/application.js配置，也就是application.js是暴露出来npm包的总入口，application.js中export出一个构造函数，用来创建一个koa应用。
``` js
module.exports = class Application extends Emitter {
  constructor() {
    super();

    /**当 this.proxy 为 true 时，支持 X-Forwarded-Host*/
    this.proxy = false;
    /**创建一个空数组存放放middleware，保存所有注册过的中间件*/
    this.middleware = [];
    /**决定忽略的子域名数量，默认为2*/
    this.subdomainOffset = 2;
    /**处理环境变量 默认为开发模式*/
    this.env = process.env.NODE_ENV || 'development';
    /**实例上挂载封装好的context,request,response对象*/
    this.context = Object.create(context);
    this.request = Object.create(request);
    this.response = Object.create(response);
  }
}
```
根据Application的定义，构造函数主要作用：
* 继承node的 Events 类， 可以监听、触发事件
* 定义this.middleware 属性，默认为空数组，保存注册过的中间件
* 创建 this.context, this.request, this.response 对象

##### Application原型上的方法
> **use：** use方法用来注册一个中间件，其实最重要的作用就是**push**中间件到`middleware`数组中

``` js
use(fn) {
  if (typeof fn !== 'function') throw new TypeError('middleware must be a function!');
  if (isGeneratorFunction(fn)) {
    deprecate('Support for generators will be removed in v3. ' +
              'See the documentation for examples of how to convert old middleware ' +
              'https://github.com/koajs/koa/blob/master/docs/migration.md');
    /**将generator转为promise*/
    fn = convert(fn);
  }
  debug('use %s', fn._name || fn.name || '-');
  this.middleware.push(fn);
  /**返回this支持链式调用*/
  return this;
}
```

> **listen: ** http.createServer的语法糖，同时使用es6展开符代替apply操作，这里注意通过this.callback函数,调用一系列中间件，对请求解析处理。

``` js
listen(...args) {
  debug('listen');
  const server = http.createServer(this.callback());
  return server.listen(...args);
}
```

> **callback: ** 在`createServer`中调用了this.callback，callback中首先将middleware生成一个**列队**，同时将node默认的request和response对象封装成`context`可以通过this访问[（代码）](https://github.com/koajs/koa/blob/2.3.0/lib/application.js#L148-L167)，接着调用`onFinished`模块给当前请求注册一个callback，最后调用fn处理请求，成功则调用respond处理context，错误执行onerror方法

``` js
callback() {
  const fn = compose(this.middleware);

  /**判断是否注册error事件，没有则使用默认onerror方法*/
  if (!this.listeners('error').length) this.on('error', this.onerror);

  const handleRequest = (req, res) => {
    res.statusCode = 404;
    /**把一些常用方法挂载至ctx这个对象中，使this同时可以调用response和request含有的方法*/
    const ctx = this.createContext(req, res);
    const onerror = err => ctx.onerror(err);
    /**respond是一个辅助函数，用来辅助设置一些response的信息*/
    const handleResponse = () => respond(ctx);
    /**添加监听器，来监听请求done调用相应callback*/
    onFinished(res, onerror);
    /**执行fn，成功执行respond，失败执行this.error*/
    return fn(ctx).then(handleResponse).catch(onerror);
  };

  return handleRequest;
}
```
可以看到callback中有`compose`处理this.middleware数组，compose将数组中第一项resolve后并且**递归**调用，下一个中间件，当前aynxc函数中的await next()后面的代码将**等待执行**，下一个中间件同理，直到数组中最后一个中间件resolve之后，将`依次向上`触发每一个中间件await next()之后的代码执行，这也就是经典洋葱结构核心思想。

``` js
return function (context, next) {
  /** last called middleware #*/
  let index = -1
  /**从第一项开始执行中间件*/
  return dispatch(0)
  function dispatch (i) {
    if (i <= index) return Promise.reject(new Error('next() called multiple times'))
    index = i
    let fn = middleware[i]
    /**这里注意callback函数中调用fn其实只传了ctx，参数next则为underfined，目的是当数组最后一个中间件执行完，调用Promise.resolve()停止递归调用*/
    if (i === middleware.length) fn = next
    if (!fn) return Promise.resolve()
    try {
      /**更改promise状态，并且递归调用下一个中间件*/
      return Promise.resolve(fn(context, function next () {
        return dispatch(i + 1)
      }))
    } catch (err) {
      return Promise.reject(err)
    }
  }
}
```

> **toJSON** 获取当前上下文的`subdomainOffset`、`proxy`、`env`三个参数

``` js
toJSON() {
  return only(this, [
    'subdomainOffset',
    'proxy',
    'env'
  ]);
}
```

> **onerror** 错误处理

``` js
onerror(err) {
  assert(err instanceof Error, `non-error thrown: ${err}`);

  if (404 == err.status || err.expose) return;
  if (this.silent) return;

  const msg = err.stack || err.toString();
  console.error();
  console.error(msg.replace(/^/gm, '  '));
  console.error();
}
```

## 参考
[1、koa框架源码阅读笔记](https://github.com/loveky/Blog/issues/3)