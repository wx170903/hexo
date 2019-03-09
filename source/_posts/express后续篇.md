---
title: express后续篇
date: 2017-11-12 20:27:42
categories: express
tags: 
- node
- express
---

# 模板引擎
## 在 Express 中使用模板引擎
需要在应用中设置模板文件目录views，设置模板引擎view engine
<!-- more -->
* views, 放模板文件的目录，比如： app.set('views', './views')
* view engine, 模板引擎，比如： app.set('view engine', 'jade')

如上例，我们用了jade模板语言，那么我们对应的需要npm相关的软件包
```sh
npm install jade --save
```

> 和 Express 兼容的模板引擎，比如 Jade，通过 res.render() 调用其导出方法 __express(filePath, options, callback) 渲染模板。
>
>有一些模板引擎不遵循这种约定，Consolidate.js 能将 Node 中所有流行的模板引擎映射为这种约定，这样就可以和 Express 无缝衔接。

一旦 view engine 设置成功，就不需要显式指定引擎，或者在应用中加载模板引擎模块，Express 已经在内部加载，如下所示。
```js
app.set('view engine', 'jade');
```

现在我们在views目录下创建名字为index.jade的Jade模板文件，内容如下：
```jade
html
  head
    title!= title
  body
    h1!= message
```

然后创建一个路由来渲染index.jade文件。如果没有设置 view engine，您需要指明视图文件的后缀，否则就会遗漏它。
```js
app.get('/', function (req, res) {
  res.render('index', { title: 'Hey', message: 'Hello there!'});
});
```

此时向主页发送请求，“index.jade” 会被渲染为 HTML。

# 错误处理
错误处理的函数形式相当于其他的中间件的`callback(req, res, next)`，它需要四个参数`(err, req, res, next)`,例如：
```js
app.use(function(err, req, res, next) {
  console.error(error.stack)
  res.status(500).send('Something broke!')
})
```

它的位置则在`app.use()`和路由调用后，最后定义错误处理中间件，比如：
```js
var bodyParser = require('body-parser');
var methodOverride = require('method-override');

app.use(bodyParser());
app.use(methodOverride());
// ...路由
app.use(function(err, req, res, next) {
  // 业务逻辑
})
```
中间件返回的响应是随意的，可以响应一个HTML错误页面、一句简单的话、一个JSON字符串、或者其他你想要的任何东西，这也就加强了对于错误的自己定义化的做法。

为了便于组织，可能会想定义常规中间件一样定制多个错误处理中间件，加强项目的逻辑性。

比如你想为使用XHR的请求定义一个，还想为没有使用的定义一个，那么：
```js
var bodyParser = require('body-parser');
var methodOverride = require('method-override');

app.use(bodyParser());
app.use(methodOverride());
app.use(logErrors);
app.use(clientErrorHandler);
app.use(errorHndler);
```

logErrors会将请求和错误信息写入标准错误输出、日志或类似服务：
```js
function logErrors(err, req, res, next) {
  console.error(err.stack);
  next(err);
}
```
clientErrorHandler 的定义如下（注意这里将错误直接传给了 next）：
```js
function clientErrorHandler(err, req, res, next) {
  if (req.xhr) {
    res.status(500).send({ error: 'Something blew up!' });
  } else {
    next(err);
  }
}
```

errorHandler 能捕获所有错误，其定义如下：
```js
function errorHandler(err, req, res, next) {
  res.status(500);
  res.render('error', { error: err });
}
```

错误处理的next() 传入参数（除了 ‘route’ 字符串），express会认为当前请求有错误的输出，会跳到下一个错误处理的中间件。

如下节所示。

如果路由句柄有多个回调函数，可使用 ‘route’ 参数跳到下一个路由句柄。比如：

```js
app.get('/a_route_behind_paywall', 
  function checkIfPaidSubscriber(req, res, next) {
    if(!req.user.hasPaid) { 
    
      // 继续处理该请求
      next('route');
    }
  }, function getPaidContent(req, res, next) {
    PaidContent.find(function(err, doc) {
      if(err) return next(err);
      res.json(doc);
    });
  });
```

在这个例子中，句柄 getPaidContent 会被跳过，但 app 中为 /a_route_behind_paywall 定义的其他句柄则会继续执行。

> next() 和 next(err) 类似于 Promise.resolve() 和 Promise.reject()。它们让您可以向 Express 发信号，告诉它当前句柄执行结束并且处于什么状态。next(err) 会跳过后续句柄，除了那些用来处理错误的句柄。

## 缺省错误处理句柄
Express 内置了一个错误处理句柄，它可以捕获应用中可能出现的任意错误。这个缺省的错误处理中间件将被添加到中间件堆栈的底部。

如果你向 next() 传递了一个 error ，而你并没有在错误处理句柄中处理这个 error，Express 内置的缺省错误处理句柄就是最后兜底的。最后错误将被连同堆栈追踪信息一同反馈到客户端。堆栈追踪信息并不会在生产环境中反馈到客户端。

> 设置环境变量 NODE_ENV 为 “production” 就可以让应用运行在生产环境模式下。

如果你已经开始向 response 输出数据了，这时才调用 next() 并传递了一个 error，比如你在将向客户端输出数据流时遇到一个错误，Express 内置的缺省错误处理句柄将帮你关闭连接并告知 request 请求失败。

因此，当你添加了一个自定义的错误处理句柄后，如果已经向客户端发送包头信息了，你还可以将错误处理交给 Express 内置的错误处理机制。

```js
function errorHandler(err, req, res, next) {
  if (res.headersSent) {
    return next(err);
  }
  res.status(500);
  res.render('error', { error: err });
}
```

# 调试 Express
Express 内部使用 debug 模块记录路由匹配、使用到的中间件、应用模式以及请求-响应循环。

> debug 有点像改装过的 console.log，不同的是，您不需要在生产代码中注释掉 debug。它会默认关闭，而且使用一个名为 DEBUG 的环境变量还可以打开。

在启动应用时，设置 DEBUG 环境变量为 express:*，可以查看 Express 中用到的所有内部日志。
```sh
DEBUG=express:* node index.js
```

在由 express 应用生成器 生成的默认应用中执行，会打印出如下信息：
```sh
> test-express@1.0.0 test /home/hdd/练习/express
> node app.js

  express:application set "x-powered-by" to true +0ms
  express:application set "etag" to 'weak' +4ms
  express:application set "etag fn" to [Function:generateETag] +1ms
  express:application set "env" to 'development' +1ms
  express:application set "query parser" to 'extended' +0ms
  express:application set "query parser fn" to [Function: parseExtendedQueryString] +0ms
  express:application set "subdomain offset" to 2+0ms
  express:application set "trust proxy" to false +0ms
  express:application set "trust proxy fn" to [Function: trustNone] +1ms
  express:application booting in development mode+0ms
  express:application set "view" to [Function: View] +0ms
  express:application set "views" to '/home/hdd/练习/express/views' +0ms
  express:application set "jsonp callback name" to 'callback' +0ms
  express:application set "view engine" to 'jade'+0ms
  express:router use '/' query +1ms
  express:router:layer new '/' +0ms
  express:router use '/' expressInit +1ms
  express:router:layer new '/' +0ms
  express:router:route new '/' +0ms
  express:router:layer new '/' +0ms
  express:router:route get '/' +1ms
  express:router:layer new '/' +0ms
```

当应用收到请求时，能看到Express代码中打印出的日志。
```sh
express:router dispatching GET / +5s
  express:router query  : / +1ms
  express:router expressInit  : / +1ms
  express:view require "jade" +2ms
  express:view lookup "index.jade" +564ms
  express:view stat "/home/hdd/练习/express/views/index.jade" +1ms
  express:view render "/home/hdd/练习/express/views/index.jade" +0ms
  express:router dispatching GET /favicon.ico +245ms
  express:router query  : /favicon.ico +0ms
  express:router expressInit  : /favicon.ico +1ms
```
设置 DEBUG 的值为 express:router，只查看路由部分的日志；设置 DEBUG 的值为 express:application，只查看应用部分的日志，依此类推。

## 通过 express 生成应用
将命名空间限制其中
需要添加`DEBUG=ITEM-NAME`
详见 [调试指南](https://github.com/visionmedia/debug)

# 数据库集成
为 Express 应用添加连接数据库的能力，只需要加载相应数据库的 Node.js 驱动即可。

介绍下常见数据库驱动
* Cassandra
* CouchDB
* LevelDB
* MySQL
* MongoDB
* Neo4j
* PostgreSQL
* Redis
* SQLite
* ElasticSearch