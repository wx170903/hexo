---
title: socket.io简易用法
date: 2017-11-06 20:12:41
categories: socket.io
tags: 
- node
- socket.io
---
# <a id="toc-introduce"></a> 介绍Socket.io
Socket.IO 是一个面向实时 web 应用的 JavaScript 库。它使得服务器和客户端之间实时双向的通信成为可能。他有两个部分：在浏览器中运行的客户端库，和一个面向Node.js的服务端库。两者有着几乎一样的API。像Node.js一样，它也是事件驱动的.
<!-- more -->

Socket.IO主要使用WebSocket协议。但是如果需要的话，Socket.io可以回退到几种其它方法，例如Adobe Flash Sockets，JSONP拉取，或是传统的AJAX拉取，[2]并且在同时提供完全相同的接口。尽管它可以被用作WebSocket的包装库，它还是提供了许多其它功能，比如广播至多个套接字，存储与不同客户有关的数据，和异步IO操作。

可以使用npm（node 软件包）工具来安装。

### <a id="toc-advantage"></a> 优势
Socket.IO会自动选择合适双向通信协议，仅仅需要程序员对套接字的概念有所了解。

### <a id="toc-disadvantage"></a> 劣势
Socket.io并不是一个基本的、独立的、能够回退到其它实时协议的WebSocket库，它实际上是一个依赖于其它实时传输协议的自定义实时传输协议的实现。该协议的协商部分使得支持标准WebSocket的客户端不能直接连接到Socket.io服务器，并且支持Socket.io的客户端也不能与非Socket.io框架的WebSocket或Comet服务器通信。因而，Socket.io要求客户端与服务器端均须使用该框架。

# <a id="toc-use"></a> 使用Socket.io
### <a id="toc-install"></a> 安装
```sh
npm install socket.io
```

### <a id="toc-use-node"></a> 直接node服务中使用
Server(app.js)
```js
var app = require('http').createServer(handler)
var io = require('socket.io')(app);
var fs = require('fs');

app.listen(80);

function handler (req, res) {
  fs.readFile(__dirname + '/index.html',
  function (err, data) {
    if (err) {
      res.writeHead(500);
      return res.end('Error loading index.html');
    }

    res.writeHead(200);
    res.end(data);
  });
}

io.on('connection', function (socket) {
  socket.emit('news', { hello: 'world' }); // 派发事件“news”
  socket.on('my other event', function (data) {
    console.log(data);
  }); // 接受Client派发的事件
});
```

Client (index.html)
```js
<script src="/socket.io/socket.io.js"></script>
<script>
  var socket = io('http://localhost');
  socket.on('news', function (data) {
    console.log(data);
    socket.emit('my other event', { my: 'data' }); // 接受完“new” 事件之后 派发事件"my other event"
  });
</script>
```

### <a id="toc-use-express"></a> 使用Express 3/4 和 Express 2.x
Server (app.js) 使用Express 3/4
```js
var app = require('express')();
var server = require('http').Server(app);
var io = require('socket.io')(server);

server.listen(80);

app.get('/', function (req, res) {
  res.sendfile(__dirname + '/index.html');
});

io.on('connection', function (socket) {
  socket.emit('news', { hello: 'world' });
  socket.on('my other event', function (data) {
    console.log(data);
  });
});
```
Server (app.js) 使用Express 2.x
```js
var app = require('express').createServer();
var io = require('socket.io')(app);

app.listen(80);

app.get('/', function (req, res) {
  res.sendFile(__dirname + '/index.html');
});

io.on('connection', function (socket) {
  socket.emit('news', { hello: 'world' });
  socket.on('my other event', function (data) {
    console.log(data);
  });
});
```

Client 都是一样的
```js
<script src="/socket.io/socket.io.js"></script>
<script>
  var socket = io.connect('http://localhost');
  socket.on('news', function (data) {
    console.log(data);
    socket.emit('my other event', { my: 'data' });
  });
</script>
```

### <a id="toc-emit-on"></a> 发送和接受事件
Socket.io允许我们派发和接受自定义事件。除此之外，`connect`,`message`和`disconnect`， 你可一发送自定义事件

Server
```js
// 注意io(<port>) 会为你创建一个http服务
var io = require('socket.io')(80);

io.on('connection', function (socket) {
  io.emit('this', { will: 'be received by everyone'});

  socket.on('private message', function (from, msg) {
    console.log('I received a private message by ', from, ' saying ', msg);
  });

  socket.on('disconnect', function () {
    io.emit('user disconnected');
  });
});
```

### <a id="toc-namespace"></a> 限制你自己的命名空间
如果您需要控制特定应用程序所以发出的消息和事件，可以使用默认或者命名空间来工作。如果你想用第三方代码或者产生代码来分享给他人使用，socket.io提供了命名一个`socket`的方法。

单个连接使用`multiplexing`是有好处的。一个socket连接来代替使用两个`WebSocket`连接。

Server (app.js)
```js
var io = require('socket.io')(80);
var chat = io
  .of('/chat')
  .on('connection', function (socket) {
    socket.emit('a message', {
        that: 'only'
      , '/chat': 'will get'
    });
    chat.emit('a message', {
        everyone: 'in'
      , '/chat': 'will get'
    });
  }); // 空间１

var news = io
  .of('/news')
  .on('connection', function (socket) {
    socket.emit('item', { news: 'item' });
  }); // 空间２
```

Client (index.html)
```js
<script>
  var chat = io.connect('http://localhost/chat')
    , news = io.connect('http://localhost/news'); // 分别链接两个命名空间
  
  chat.on('connect', function () {
    chat.emit('hi!');
  });
  
  news.on('news', function () {
    news.emit('woot');
  });
</script>
```

### <a id="toc-volatile"></a> 发送不稳定的消息
有时，有些消息会被丢失。比如说，你有一个应用程序，显示实时连接的关键字`bieber`

如果某个客户端没有准备好接收消息（可能是网络缓慢或其他问题， 或者他们处于长轮询连接并且处于请求阶段）。如果他没有收到所有消息，你的程序如何不受影响呢？

在这种情况下，你也许需要发送一些不稳定信息
Server
```js
var io = require('socket.io')(80);

io.on('connection', function (socket) {
  var tweets = setInterval(function () {
    getBieberTweet(function (tweet) {
      socket.volatile.emit('bieber tweet', tweet);
    });
  }, 100);

  socket.on('disconnect', function () {
    clearInterval(tweets);
  });
});
// 就是一个定时器。。。加上volatila 客户端接收到了的话，肯定会有一个回调like: clearInterval(tweets)
```

### <a id="toc-callback"></a> 发送和获取数据（确认）
有时，你也许想要客户端接收到消息发送一个回调来确认小安溪接收到了。OK，可以做到这一点

只需在`.send`， `.emit`后面添加一个回调函数fn
Server (app.js)
```js
var io = require('socket.io')(80);

io.on('connection', function (socket) {
  socket.on('ferret', function (name, fn) {
    fn('woot');
  });
});
```

Client (index.html)
```js
<script>
  var socket = io(); // TIP: io() with no args does auto-discovery
  socket.on('connect', function () { // TIP: you can avoid listening on `connect` and listen on events directly too!
    socket.emit('ferret', 'tobi', function (data) {
      console.log(data); // data will be 'woot'
    });
  });
</script>
```

### <a id="toc-broadcast"></a> 广播消息
要广播，直接在`emit`和`send`前面添加`broadcast`就可以了。广播意味着除了消息发送者，所有人都将获得这个消息

Server
```js
var io = require('socket.io')(80);

io.on('connection', function (socket) {
  socket.broadcast.emit('user connected');
});
```

