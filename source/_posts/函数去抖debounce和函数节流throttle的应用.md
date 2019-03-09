---
title: 函数去抖debounce和函数节流throttle的应用
date: 2018-07-08 12:43:09
categories: JStricks
tags: 
- JS
- 优化
---
开始以为只有节流函数，原来还有去抖函数啊。两个概念第一次听的话肯定比较容易混淆，接下来就来研究一下它们吧。
<!-- more -->
# 函数去抖debounce
在一段时间内执行该动作，在单位时间内重新调用该动作，会重新计算时间。

我的理解就是，只要触发的够快，函数就追不上我，哈哈哈。

当我停下来的时候，函数会触发最后一次。所以其中肯定有一个定时器哈，下面展示一下简单实现的代码。

```js
function debounce(fn, delay) {
    // 定义一个定时器
    var timer = null;
    return function() {
        // 绑定函数调用时上下文及参数
        var context = this;
        args = arguments;
        clearTimeout(timer);
        timer = setTimeout(function() {
            console.log('debounce被触发了');
            fn.apply(context, args);
        }, delay);
    }
}
// 定义一个btn，然后去点它来测试
$('.btn-click-debounce').on('click', debounce(function() {
    console.log('200ms一次')
}, 200))
```

# 函数节流throttle
将倾斜而出的水流，一点一点的流出，这就是节流。放在函数触发中，就是本来快速l连续触发的事件以单位时间为间隔来触发。简单实现的源码：
```js
function throttle(fn, threshhold) {
    var last = null;
    var timer = null;
    threshhold || (threshhold = 200);
    return function () {
        var context = this;
        var args = arguments;
        var now = +new Date(); // Date.now();

        if(last && ((now < last + threshhold)) {
            // 判断时间间隔，以及是否第一次触发
            clearTimeout(timer);
            timer = setTimeout(function() {
                last = now;
                console.log('throttle被触发了');
                fn.apply(context, args);
            }, threshhold)
        } else {
            last = now;
            console.log('throttle被触发了');
            fn.apply(context, args);
        }
    }
}

$('.btn-click-throttle').on('click', throttle(function() {
    console.log('200ms一次')
}, 200))
```

# 使用的场景
## throttle函数节流
* Dom元素的拖拽实现（mousemove）
* 射击游戏的mousedown/keydown事件（单位时间只能发射一颗子弹）
* 计算鼠标移动的距离（mousemove）
* Canvas模拟画板功能
* 搜索联想（keyup）
* 监听滚动事件是否到底部自动加载更多（
监听滚动事件判断是否到页面底部自动加载更多：给 scroll 加了 debounce 后，只有用户停止滚动后，才会判断是否到了页面底部；如果是 throttle 的话，只要页面滚动就会间隔一段时间判断一次）

## debounce应用场景
* 每次resize/scroll触发统计
* 文本输入的验证（连续输入文字后发送ajax进行验证，验证一次就好）

# 参考
[http://www.cnblogs.com/zichi/p/5948936.html](http://www.cnblogs.com/zichi/p/5948936.html)

[https://www.cnblogs.com/fsjohnhuang/p/4147810.html](https://www.cnblogs.com/fsjohnhuang/p/4147810.html)










