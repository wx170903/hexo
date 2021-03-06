---
title: ES6之Promise（一）
date: 2018-01-18 10:50:44
categories: ES6
description: Promise是ES6更新一个强大的异步对象，是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。
tags:
- JS
---
# Promise含义

javaScript异步最常用的方式就是通过成功的回调函数进行对返回过来的参数进行逻辑处理。一般的回调函数我们都会使用嵌套回调或者链式回调。这样的写法会影响后续的开发。

## 会产生以下的问题：
1.当采用嵌套回调的时候，会导致层级他多，不利于维护；

2.所以我们又采用了链式回调，对嵌套回调进行拆分，拆分后的函数间的耦合度很高，容易产生BUG；

3.如果需要传递参数，函数之间的关联性会更高，参数需要校验，或者多参的时候，要进行多次判断提高代码的合理性；

4.如果回调函数的参数要提交给第三方库或者插件（支付，地图，第三方API，接口），就要考虑一些不可控因素；
- 调用的时间太早
- 调用的时间太晚
- 多次调用，或者调用少

在这个背景下，ES6诞生之时社区最早提出了更合理更强大的解决办法，ES6将其纳入写进了语言标准，统一了用法，原生提供了Promise对象。

# 什么是Promise
````
设想一下这个场景，我去一家螺丝粉店，交给收银员老板娘10元，下单买一个螺蛳粉，下单付款。到这里，我已经发出了一个请求（购买了一份螺丝粉），启动了一次交易。

但是厨房（系统）需要制作的时间（响应的时间），我不能马上得到螺蛳粉吃，收银员老板娘会给一张小票替代螺丝粉，这张小票就是一个承诺（Promise），保证我最后能得到这份螺蛳粉。

所以我需要好好的保留的这张小票，对我来说，小票就是螺蛳粉，虽然这张小票不能吃，我需要等待螺蛳粉做好，等待收银员老板娘叫号通知我。

在等待的过程中，我可以做别的事情，如：刷微博，撩老板娘……（loading等操作），pending（进行中）；

收银员老板娘终于叫到我的号了，我就用这张小票去换螺丝粉；

当然还有一种情况，当我去柜台取螺蛳粉的时候，收银员告诉我螺蛳粉卖光了，做螺蛳粉的师傅受伤了等等原因，导致了我无法得到这个螺蛳粉；

虽然我有小票（承诺）Promise，但是可能得到螺蛳粉 fulfilled（已成功），可能得不到螺蛳粉 rejected（已失败）；
我由等待螺蛳粉变成了等到或者等不到，这个过程不可逆。
````
上面很形象的介绍了promise，上面的等待螺蛳粉和得到螺蛳粉，螺蛳粉卖光了，得不到螺蛳粉，分别对应promise的三种状态
三种状态：pending（进行中）、fulfilled（已成功）和rejected（已失败）(一旦状态改变，就不会再变)。

# 回调函数调用过早

调用过早就是将异步函数作为同步处理了，

我们之前说过，javascript以单线程同步的方式执行主线程，遇到异步会将异步函数放入到任务队列中，

当主线程执行完毕，会循环执行任务队列中的函数，也就是事件循环，直到任务队列为空。

## 事件循环和任务队列

事件循环就像是一个游乐场，玩过一个游戏后，你需要重新排到队尾才能再玩一次

任务队列就是，在你玩过一个游戏后，可以插队接着玩

果断举一个栗子🌰：
```js
   const promise = new Promise((resolve, reject) => {
      resolve("成功啦")
    });
    promise.then(res => {
      console.log(res);
      console.log("我是异步执行的");
    })
    console.log('我在主线程');
```
输出顺序
```js
//我在主线程
//成功啦
//我是异步执行的
```
直接手动是promise的状态切为成功状态，console.log("我是异步执行的");这段代码也是异步执行的
提供给then()的回调永远都是异步执行的，所以promise中不会出现回调函数过早执行的情况。

# 回调函数调用过晚或不被调用

回调函数调用过晚的处理原理和调用过早很类似，

在promise的then()中存放着异步函数，所有的异步都存在于js的任务队列中，当js的主线程执行完毕后，会依次执行任务队列中的内容，不会出现执行过晚的情况

果断举一个栗子🌰：
```js
  const promise = new Promise((resolve, reject) => resolve('成功啦'))
  promise.then(s => console.log(s));
  console.log('我在主线程');
```
成功状态的输出
````
//我在主线程
//成功啦
````
成功状态下回调被调用
继续看一下失败的回调
```js
  const promise = new Promise((resolve, reject) => reject('失败啦'))
  promise.then(null, s => console.log(s));
  console.log('我在主线程');
```
失败状态的输出
````
//我在主线程
//失败啦
````

# 回调函数调用次数过多或者过少
调用次数过多
我们之前说了promise有三种状态
pending（进行中）、fulfilled（已成功）和rejected（已失败）状态一旦状态改变，就不会再变
一个栗子
```js
    const promise = new Promise((resolve, reject) => {
      reject('失败啦')
      resolve('成功啦')
    });
    promise.then(res => {
      console.log(`我是异步执行的成功:${res}`);
    },err =>{
      console.log(`我是异步执行的失败:${err}`);
    }).catch(err => {
      console.log(err);
    })
    console.log('我在主线程');
```

输出

````
//我在主线程
//我是异步执行的失败:失败啦
````
当状态变为失败时，就不会再变为成功，成功的函数也不会执行，反之亦然。

# Promise用法会封装请看下一篇。