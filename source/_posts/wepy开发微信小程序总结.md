---
title: wepy开发微信小程序总结
date: 2017-12-27 16:44:25
categories: wepy
tags: 
- wepy
- 微信小程序
---
# 项目源代码
[https://github.com/hddhyq/One-per-day](https://github.com/hddhyq/One-per-day)
<!-- more -->

### 项目截图
![](https://github.com/hddhyq/One-per-day/raw/master/screenshot/oneList.gif)
![](https://github.com/hddhyq/One-per-day/raw/master/screenshot/oneGrap.gif)
![](https://github.com/hddhyq/One-per-day/raw/master/screenshot/oneEassy.gif)
![](https://github.com/hddhyq/One-per-day/raw/master/screenshot/oneMovie.gif)

# 认识wepy和微信小程序
wepy 相对于小程序原生开发框架相比有很多优点，缺点也许也有，这里我们先看看优点，毕竟可以wepy和微信自带的框架混合开发。
* 开发风格
  接近于 Vue.js，支持组件 Props 传值，自定义事件、组件分布式复用Mixin、计算属性函数computed、模板内容分发slot等等

* 组件化、第三方npm资源自动处理依赖关系、支持ES2015(Promise/Class/Async)等

* 对小程序本身的优化、支持样式编译器: Less/Sass/Stylus, 模板编译器：wx-ml/Pug，代码编译器：Babel/Typescript。支持插件如：文件压缩和图片压缩

至于细节还是去访问官网比较好。
* [wepy文档](https://tencent.github.io/wepy/document.html)
* [微信小程序文档](https://mp.weixin.qq.com/debug/wxadoc/dev/)

## scroll-view 必须指定高度
开发电影下拉加载的时候，scroll-view 没给高度，拉到页面底部没有触发新的数据请求，解决方法： 
```styl
.scroll-container
  position: absolute
  top: 0
  right: 0
  bottom: 0
  left: 0
```

## 小程序页面hidden不能用于flex布局
解决方法：将flex布局的view 重新包裹上一层view

## wepy孙子组件的传值
wepy的传值给孙子组件，孙子组件没有及时跟新，最后放弃了孙子组件，将本来的孙子组件一起写到了父组件。

## wepy的组件style不能添加scoped
给组件添加了scoped，不能正确显示相关的style，解决方法是把子组件的scoped去掉了。应该是scoped重复了两次，引用组件的时候应该不让组件scoped生成，或者父组件不对子组件引用添加scoped前缀，个人猜测。

## wepy props的传值
在评星组件中，传入数值的时候
```
<rating :score.sync="score"></rating>
```
score是数值，结果传递了一个对象的属性，传值失败，结果最后将分数单独设置为一个data中的Number类型的数据，传值成功。

## swiper的高度及跳转
图文的详情本来是想用swiper来作为项目容器，结果swiper组件的高度，必须固定，而我的图文详情页的高度没有固定，第二点，并没有查阅到swiper的跳转功能，那我在首页点击的图片进入的图片还是第一张，最后放弃了swiper这个想法，不过总的来说，swiper主要还是用来放几张图片在首页轮播的。

## 关于图文详情页的跳转
oneDetail 最后实现的只是数据改变，重新绘制页面，想的是实现swiper图片的切换效果，这里没有实现。不过有一个想法，把页面的宽度设置为300%的宽度，也就是三张图片的宽度，然后进行切换页面的transformＸ轴的计算和页面滑动用js相关联进行计算，上下两张图片再用懒加载，这样就实现了页面的切换。