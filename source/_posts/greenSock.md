---
title: greenSock
date: 2019-03-04 10:02:14
categories: js动画
description: 这是一个很好的动画库，全面的API。
tags: 
- JS动画库
---
<!-- more -->
 个人git仓库有相关包下载，打开终端clone下来
````
指令：git clone https://github.com/wx170903/greensock.git
````

 基本操作——TweenMax.to( );

````
	Jquery选择器选择1以变量保存起来var $box = $('#box');
	TweenMax.to($box, 7, { 'left': 300, backgroundColor: "pink" });
	上面的代码会在7秒之内把$box元素从CSS中定义的位置移动到距离body-300px的边缘并且换成pink颜色。
````

基本操作——TweenMax.from( );

```
var $box = $('#box');
TweenMax.from($box, 2, {x: '-=500px',repeat:-1, autoAlpha: 1});
在代码中autoAlpha: 0表示它会把元素初始化为opacity:0;visibility:hidden。当执行动画效果的时候它会把visibility的值设置为inherit以及opacity值设置为1。从而产生一个渐现的效果。
```

基本操作——TweenMax.set( );

```
有时候，我们只是想设置元素的一些CSS属性并不需要动画效果，比如，重设元素的位置。
这个时候就可以使用GreenSock提供的.set()方法。注：定义的变量就不写出来的。
TweenLite.set($box, {x: '-=200px', scale: 0.3});
TweenLite.set($box, {x: '+=100px', scale: 0.6, delay: 1});
TweenLite.set($box, {x: '-50%', scale: 1, delay: 2});
运行上面的代码，可以看到元素只是单纯的在改变属性并没有动画效果。
```

基本操作——TweenMax.fromTo( );

```
这个方法结合了上面的，从头到尾定义动画的起始位置并且不重复。
TweenLite.fromTo($box, 2, {x: '0'}, {x: 150});
把上面的代码放入到JS代码中，就可以看到运行的动画效果。
盒子从-=200px的位置走到150px的位置，在括号里面可以添置变化属性（比如颜色，大小，不透明度等）；
x:150会覆盖在CSS中定义的transform: translate(–50%, –50%)的样式，用新的transform: matrix(1, 0, 0, 1, 150, -50);样式来代替。
```

接下来，可以给元素加一些缓动曲线，使之更符合真实的物体运动效果。GreenSock也提供了各种的运动曲线，来使动画效果更加自然。


