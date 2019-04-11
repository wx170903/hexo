---
title: 使用RequireJs的好处
date: 2017-10-31 16:19:26
categories: JS工具
description: 初步认识模块化开发的理由。
tags: 
- JS
- 项目优化
---

一、使用RequireJs的好处

最早的时候，所有Javascript代码都写在一个文件里面，只要加载这一个文件就够了。后来，代码越来越多，一个文件不够了，必须分成多个文件，依次加载。下面的网页代码，相信很多人都见过。

	<script type="text/javascript" src="./js/a.js"></script>
	<script type="text/javascript" src="./js/b.js"></script>
	<script type="text/javascript" src="./js/c.js"></script>
	<script type="text/javascript" src="./js/d.js"></script>
	<script type="text/javascript" src="./js/e.js"></script>

这些js文件都是按照顺序从上到下依次同步加载的。

这样的写法有很大的缺点。首先，加载的时候，浏览器会停止网页渲染，加载文件越多，网页失去响应的时间就会越长；其次，由于js文件之间存在依赖关系，因此必须严格保证加载顺序（比如上例的a.js要在b.js的前面），依赖性最大的模块一定要放到最后加载，当依赖关系很复杂的时候，代码的编写和维护都会变得困难。

RequireJs很好的解决了两个问题及其优势：

1、实现js文件的异步加载，避免网页失去响应；

2、管理模块之间的依赖性，便于代码的编写和维护。

3、基于AMD模块化机制，让前端代码也能实现模块化。查看《CommonJS和AMD/CMD区别详解》。

开始今天的学习之前，首先需要准备4个文件，分别是【jQuery库】、【require库】、【bootstrap库】、【require加载css插件】，大家可以点击下面的链接进行下载：

1、jquery.js

2、require.js

3、bootstrap.min.js

4、require.css.js

二、如何使用RequireJs？

1、在html页面中引入require库，如下：

	<script type="text/javascript" src="./js/require.js"></script>
2、加载require.js以后，下一步就要加载我们自己的代码了。假定我们自己的代码文件是main.js，也放在js目录下面。那么，只需要写成下面这样就行了：

	<script src="js/require.js" data-main="./js/main"></script>
data-main属性的作用是，指定网页程序的主模块。在上例中，就是js目录下面的main.js，这个文件会第一个被require.js加载。由于require.js默认的文件后缀名是js，所以可以把main.js简写成main（data-main引入的main.js文件是异步加载的，所以在加载这个文件的同时也会加载其他js文件，其他文件要依赖这个main.js，所以可以将以上代码分开来写）。

	<script type="text/javascript" src="./js/require.js"></script>
	<script type="text/javascript" src="./js/main.js"></script>
3、主模块main.js的写法：

主模块main.js主要是加载requireJs的配置项，所以需要先了解一下这些配置项的意思：

（1）baseUrl：模块默认加载路径；

（2）paths：自定义模块加载路径；

（3）shim：定义模块之间的依赖关系。

下面具体来看下，这是我的目录结构：

![](https://i.imgur.com/p7Cnmal.jpg)

	require.config({
	    'baseUrl' : './app/',
	    'paths' : {
	        'jquery' : '../js/jquery',//只写文件名，不用带后缀
	    },
	});

html页面调用：

	require(['jquery'], function(a){
	    console.log(a);//functionn()
	});