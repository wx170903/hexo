---
title: 了解下Express
date: 2017-11-11 20:34:18
categories: express
tags: 
- node
- express
---
# <a id="route"></a> 路由
虽说第一项是路由。不过首先，还是应该安装一下express。
<!-- more -->
```sh
npm install express --save
```
## <a id="what-is-router"></a> 什么是路由
路由是定义应用请求的URLs以及响应客户端的请求。结构如下：`app.METHOD(path, [callback...], callback)`  其中app是express的一个实例, METHOD是一个HTTP请求方法, callback是当路径匹配时要执行的函数。

示例
```js
const express = require('express')
const app = express()

app.get('/', (req, res) => {
  res.send('hello world')
})

app.listen(3000, () => {
  console.log('服务开始在3000端口')
})
```

## <a id="method-router"></a> 路由方法
路由方法源自HTTP请求方法。

Express定义了以下与HTTP对应的方法：get, post, put, head, delete, options, trace, copy, lock, mkcol, move, purge, propfind, proppatch, unlock, report, mkactivity, checkout, merge, m-search, notify, subscribe, unsubscribe, patch, search和connect。
其中JS不合法的需要使用括号记法，比如：`app['m-search'].('/', callback...)`

`app.all` 也是method,它匹配的任何HTTP请求，句柄都会执行。
```js
app.all('/secret', (req, res, next) => {
  console.log('handle the secret!')
  next() // 处理写一个句柄，相当于promise中的promise.resolve()
})
```

## <a id="path-router"></a> 路由路径
路由路径和请求方法一起定义了请求的端点，它可以是字符串、字符串模式或者正则表达式。根据精确度排序匹配率排序 字符串>字符串模式>正则表达式

```js
// 字符串
app.get('/about', (req, res) => {
  res.send('about')
})

// 字符串模式
app.get('/ab*ut', (req. res) => {
  res.send('ab*ut')
})

// 字符?,+,*和()是正则表达式的子集、在基于字符串的路径按照字面值解释。

// 正则表达式
app.get(/ab/, (req, res) => {
  res.send('/ab/') // 匹配任何含有ab的路径
})
```

## <a id="next-router"></a> 路由句柄
可以为请求处理提供多个回调函数，行为类似中间件。唯一的区别，这些回调函数可能通过`next('route')`方法而略过其他路由回调函数。可以利用该机制为路由定义前提条件，如果在现有路径继续执行没有意义，则可将控制权交给剩下的路径。

路由句柄有多种形式，可以是一个函数、一个函数数组，或者是两者混合，如下所示.

使用多个回调函数处理路由（记得指定 next 对象）：
```js
app.get('/example/b', function (req, res, next) {
  console.log('response will be sent by the next function ...');
  next();
}, function (req, res) {
  res.send('Hello from B!');
});
```

使用回调函数数组处理路由：
```js
var cb0 = function (req, res, next) {
  console.log('CB0');
  next();
}

var cb1 = function (req, res, next) {
  console.log('CB1');
  next();
}

var cb2 = function (req, res) {
  res.send('Hello from C!');
}

app.get('/example/c', [cb0, cb1, cb2]);
```

## <a id="callback-router"></a> 响应方法
下表中响应对象（res）的方法向客户端返回响应，终结请求响应的循环。如果在路由句柄中一个方法也不调用，来自客户端的请求会一直挂起。

|方法|描述|
|----|---|
|[res.download()](http://www.expressjs.com.cn/4x/api.html#res.download)|提示下载文件。|
|[res.end()](http://www.expressjs.com.cn/4x/api.html#res.end)|终结响应处理流程。|
|[res.json()](http://www.expressjs.com.cn/4x/api.html#res.json)|发送一个 JSON 格式的响应。|
|[res.jsonp()](http://www.expressjs.com.cn/4x/api.html#res.jsonp)|发送一个支持 JSONP 的 JSON 格式的响应。|
|[res.redirect()](http://www.expressjs.com.cn/4x/api.html#res.redirect)|重定向请求。|
|[res.render()](http://www.expressjs.com.cn/4x/api.html#res.render)|渲染视图模板。|
|[res.send()](http://www.expressjs.com.cn/4x/api.html#res.send)|发送各种类型的响应。|
|[res.sendFile](http://www.expressjs.com.cn/4x/api.html#res.sendFile)|以八位字节流的形式发送文件。|
|[res.sendStatus()](http://www.expressjs.com.cn/4x/api.html#res.sendStatus)|设置响应状态代码，并将其以字符串形式作为响应体的一部分发送。
|

## <a id="appRoute"></a> app.route()
可使用 app.route() 创建路由路径的链式路由句柄。由于路径在一个地方指定，这样做有助于创建模块化的路由，而且减少了代码冗余和拼写错误。
对于同一个路径不同method的链式路由句柄
```js
app.route('/book')
  .get(function(req, res) {
    res.send('Get a random book');
  })
  .post(function(req, res) {
    res.send('Add a book');
  })
  .put(function(req, res) {
    res.send('Update the book');
  });
```

## <a id="expressRouter"></a> express.Router
可使用 express.Router 类创建模块化、可挂载的路由句柄。Router 实例是一个完整的中间件和路由系统，因此常称其为一个 “mini-app”。

在 app 目录下创建名为 birds.js 的文件，内容如下：

```js
var express = require('express');
var router = express.Router();

// 该路由使用的中间件
router.use(function timeLog(req, res, next) {
  console.log('Time: ', Date.now());
  next();
});
// 定义网站主页的路由
router.get('/', function(req, res) {
  res.send('Birds home page');
});
// 定义 about 页面的路由
router.get('/about', function(req, res) {
  res.send('About birds');
});

module.exports = router;
```

然后在应用中加载路由模块：
```js
var birds = require('./birds');
...
app.use('/birds', birds);
```
应用即可处理发自 /birds 和 /birds/about 的请求，并且调用为该路由指定的 timeLog 中间件。

# <a id="middleware"></a> 中间件
Express框架实现功能就相当于在调用相关的中间件（Middleware），它可访问request和response
它的功能包括：
* 执行任何代码。
* 修改请求和响应对象。
* 终结请求-响应循环。
* 调用堆栈中的下一个中间件。

如果当前中间件没有终结请求-响应循环，则必须调用 next() 方法将控制权交给下一个中间件，否则请求就会挂起。

Express 应用可使用如下几种中间件：
* 应用级中间件
* 路由级中间件
* 错误处理中间件
* 内置中间件
* 第三方中间件

## <a id="app-middleware"></a> 应用级中间件
应用级中间件绑定到 app 对象 使用 app.use() 和 app.METHOD()
```js
// 路由和句柄函数(中间件系统)，处理指向 /user/:id 的 GET 请求
app.get('/user/:id', function (req, res, next) {
  res.send('USER');
}); // :id 由req.params.id 来获取
```

调用`next('route')`时，会跳过剩余中间件

如果需要在中间件栈中跳过剩余中间件，调用 next('route') 方法将控制权交给下一个路由。 **注意：** next('route') 只对使用 app.VERB() 或 router.VERB() 加载的中间件有效。
```js
// 一个中间件栈，处理指向 /user/:id 的 GET 请求
app.get('/user/:id', function (req, res, next) {
  // 如果 user id 为 0, 跳到下一个路由
  if (req.params.id == 0) next('route');
  // 否则将控制权交给栈中下一个中间件
  else next(); //
}, function (req, res, next) {
  // 渲染常规页面
  res.render('regular');
});

// 处理 /user/:id， 渲染一个特殊页面
app.get('/user/:id', function (req, res, next) {
  res.render('special');
});
```

## <a id="router-middleware"></a> 路由级中间件
路由级中间件和应用级中间件一样，只是它绑定的对象为 express.Router()。

## <a id="error-handle-middleware"></a> 错误处理中间件
> 错误处理中间件有 4 个参数，定义错误处理中间件时必须使用这 4 个参数。即使不需要 next 对象，也必须在签名中声明它，否则中间件会被识别为一个常规中间件，不能处理错误。

错误处理中间件和其他中间件定义类似，只是要使用 4 个参数，而不是 3 个，其签名如下： (err, req, res, next)。
```js
app.use(function(err, req, res, next) {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});
```

## <a id="in-middleware"></a> 内置中间件
内置中间件只有一个`express.static` 。负责Express应用中托管静态资源。用法： `express.static(root, [options])`

可选options：
|属性|描述|类型|缺省值|
|----|---|---|-----|
|dotfiles|是否对外输出文件名以点（.）开头的文件。可选值为 “allow”、“deny” 和 “ignore”|String|“ignore”|
|etag|是否启用 etag 生成|Boolean|true|
|extensions|设置文件扩展名备份选项|Array|[]|
|index|发送目录索引文件，设置为 false 禁用目录索引。|Mixed|“index.html”|
|lastModified|设置 Last-Modified 头为文件在操作系统上的最后修改日期。可能值为 true 或 false。|Boolean|true|
|maxAge|以毫秒或者其字符串格式设置 Cache-Control 头的 max-age 属性。|Number|0|
|redirect|当路径为目录时，重定向至 “/”。|Boolean|true|
|setHeaders|设置 HTTP 头以提供文件的函数。|Function||

下面的例子使用了 express.static 中间件，其中的 options 对象经过了精心的设计。
```js
var options = {
  dotfiles: 'ignore',
  etag: false,
  extensions: ['htm', 'html'],
  index: false,
  maxAge: '1d',
  redirect: false,
  setHeaders: function (res, path, stat) {
    res.set('x-timestamp', Date.now());
  }
}

app.use(express.static('public', options));
```

每个应用可有多个静态目录。当然也可以分别设置options。

```js
app.use(express.static('public'));
app.use(express.static('uploads'));
app.use(express.static('files'));
```
更多关于 serve-static 和其参数的信息，请参考 [serve-static](https://github.com/expressjs/serve-static) 文档。
## <a id="out-middleware"></a> 第三方中间件
通过使用第三方中间件从而为 Express 应用增加更多功能。

安装所需功能的 node 模块，并在应用中加载，可以在应用级加载，也可以在路由级加载。

下面的例子安装并加载了一个解析 cookie 的中间件： cookie-parser
```sh
$ npm install cookie-parser
```

```js
var express = require('express');
var app = express();
var cookieParser = require('cookie-parser');

// 加载用于解析 cookie 的中间件
app.use(cookieParser());
```
请参考 [第三方中间件](http://www.expressjs.com.cn/resources/middleware.html) 获取 Express 中经常用到的第三方中间件列表。



