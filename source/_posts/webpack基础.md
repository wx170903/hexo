---
title: webpack基础
date: 2019-04-08 14:57:42
categories: JS
tags: 
- JS工具
- webpack
---
对于webpack,前端开发人员都是必备的技能，建议开始学习的时候先预览一遍文档，了解一下什么是前端自动化构建工具，为什么要用到webpack，webpack会给开发中带来什么。
<!-- more -->
# 先了解一下传统开发中的script标签的加载问题
- ## 在以前的传统开发过程中，如果我们有一段javaScript代码，我们有两种方法引用它
  1.直接在html书写这段script
  2.使用标签引入<script src="*.js"></script>
- ## 然后就会出现很多问题：
  1.无法被压缩
  2.过多的script标签
  在HTTP1.1的网络协议下，浏览器同时最大连接数为：
    |version    |Maximun connections|
    |--------|----:|
    |Internet Explorer® 7.0       |2|
    |Internet Explorer 8.0 and 9.0|6|
    |Internet Explorer 10.0       |8|
    |Internet Explorer 11.0      |13|
    |Firefox®                     |6|
    |Chrome™                      |6|
    |Safari®                      |6|
    |Opera®                       |6|
    |iOS®                         |6|
    |Android™                     |6|

    | Tables        | Are           | Cool  |
    | ------------- |:-------------:| -----:|
    | col 3 is      | right-aligned | $1600 |
    | col 2 is      | centered      |   $12 |
    | zebra stripes | are neat      |    $1 |

  3.脚步比较难维护，代码耦合性太高，出现变量污染的情况比较大，迭代的情况下，容易产生BUG。
- ## 解决办法
  这时 IIFE（Immediately Invoked Function Expression）立即执行函数出现在人们眼前
  ```js
  // makeCounter函数返回的是一个新的函数，该函数对makeCounter里的局部变量i享有使用权
  function makeCounter() {
    // i只是makeCounter函数内的局部变量
    var i = 0;
    return function() {
      console.log( ++i );
    };
  }
  // 注意counter和counter2是不同的实例，它们分别拥有自己范围里的i变量
  var counter = makeCounter();
  counter(); // 1
  counter(); // 2
  var counter2 = makeCounter();
  counter2(); // 1
  counter2(); // 2
  i; // 报错，i没有定义，它只是makeCounter内部的局部变量
  ```
  那么现在我们需要做的就是，把每一个文件都当作一个 IIFE（揭示模块）调用。

- ## 大量的使用IIFE也会带来问题：

  1.每一次文件改变都需要所有的IIFE全部执行一边。

  2.大量的使用IIFE，调用回变得非常慢，页面会出现空白和卡顿。

  3.只有同步加载，没有异步加载，无法实现lazy code。
   
#模块的历史
在谷歌发布了V8引擎后，Node.js顺势而起，伴随着Node.js诞生的CommonJS（Module 1.0）。采用静态
语言分析，使用NPM + Node + Module的生态。

- ## Node.js的CommonJS存在的问题：

  1.不支持浏览器（不支持浏览器的解决方法：使用 RequireJS (loader)）

  2.没有活动绑定，存在循环引用问题。

  3.模块同步加载，加载速度慢。

