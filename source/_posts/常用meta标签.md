---
title: 常用meta标签
date: 2017-10-14 10:44:21
categories: HTML基础
tags: 
- HTML
- meta标签
---
## meta常用属性
[原地址](https://tink.gitbooks.io/fe-collections/content/ch02-html/s03-meta.html)<br>
作用：提供HTML文档的元数据，常用于告知浏览器如何显示内容和搜索引擎优化。
<!-- more -->
# 网页相关
## 声明编码
```
<meta charset='utf-8' />
```

## 优先使用IE最新版本和Chrome
```
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
<!-- 关于X-UA-Compatible -->
<meta http-equiv="X-UA-Compatible" content="IE=6" ><!-- 使用IE6 -->
<meta http-equiv="X-UA-Compatible" content="IE=7" ><!-- 使用IE7 -->
<meta http-equiv="X-UA-Compatible" content="IE=8" ><!-- 使用IE8 -->
```

## 浏览器内核控制
国内浏览器很多都是双内核(wekit和Trident)，wekit内核高速浏览，IE内核兼容网页和旧版网站。而添加meta标签的网站可以控制浏览器选择何种内核渲染。
```
<meta name="renderer" content="webkit|ie-comp|ie-stand">
```
国内双核浏览器默认内核模式如下:
1. 搜狗高速浏览器、QQ浏览器: IE内核（兼容模式）
2. 360极速浏览器、遨游浏览器: Webkit内核（极速模式）

## 禁止浏览器从本地计算机的缓存中访问页面内容
这样设定,访问着者无法脱机浏览。
```
<meta http-equiv="Pragma" content="no-cache">
```
## Windows 8
* Windows 8 磁贴颜色
```
<meta name="msapplication-TileColor" content="#000"/>
```
* Windows 8 磁贴图标
```
<meta name="msapplication-TileImage" content="icon.png"/>
```

## 站点适配
主要用于PC-手机页的对应关系。
```
<meta name="mobile-agent"content="format=[wml|xhtml|html5]; url=url">
```
* `[wml | xhtml | html5]`

根据手机页的协议语言，选择其中一种。
* `url="url"`

后者代表当前PC页所对应的手机页URL，两者必须是一一对应关系

## 避免百度转码声明
用百度打开网页可能会对其进行转码(比如贴广告)，避免转码可添加如下meta
```
<meta http-equiv="Cache-Control" content="no-siteapp" />
```

# 用于SEO优化
## 页面关键词
每个网页应具有面熟网页内容的一组唯一的关键字，不要太长也不要太短
```
<meta name="keywords" content="your tags" />
```

## 页面描述
每个网页都应该有一个不超过１５０字符的页面描述
```
<meta name="description" content="150 words" />
```
## 设置搜索引擎索引方式
```
<meta name="robots" content="index,follow" />
<!--
    all：文件将被检索，且页面上的链接可以被查询；
    none：文件将不被检索，且页面上的链接不可以被查询；
    index：文件将被检索；
    follow：页面上的链接可以被查询；
    noindex：文件将不被检索；
    nofollow：页面上的链接不可以被查询。
-->
```

# 移动设备
## viewport
viewport能优化移动浏览器的显示。如果不是响应式网站，不要使用initial-scale或者禁用缩放
* 大部分4.7-5寸设备的viewport宽设为360px；
* 5.5寸设备设为400px；
* iphone6设为375px；ipone6 plus设为414px。
```
<meta name="viewport" content="width=device-width, initial-scale=1.0,maximum-scale=1.0, user-scalable=no"/>
```
`width=device-width`会导致iPhone5添加到主屏后以WebApp全屏模式打开页面时出现黑边
* width宽度

数值或device-width，范围从200到10000，默认980像素

* height高度

数值或device-height， 范围从223到10000

* initial-scale 初始的缩放比例

范围从0到10

* minimum-scale 允许用户缩放到的最小比例
* maximum-scale 允许用户缩放到的最大比例
* user-scalable 用户是否可以手动缩

注意，很多人使用initial-scale=1运用到非响应式网站上，这会让网站以100%宽度渲染，用户需要手动移动页面或者缩放。如果nitial-scale=1同时使用user-scalable=no或maximum-scale=1，则用户将不能放大/缩小网页来看到全部的内容。

## WebApp全屏模式
启用全屏模式，伪装app，离线应用。
```
<meta name="apple-mobile-web-app-capable" content="yes" />
```

## 隐藏状态栏/设置状态栏颜色
只有在开启WebApp全屏模式时才生效。 content的值为default | black | black-translucent 。
```
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
```

## 添加到主屏后的标题
```
<meta name="apple-mobile-web-app-title" content="标题">
```

## 忽略数字自动识别为电话号码
```
<meta content="telephone=no" name="format-detection" />
```

## 忽略识别邮箱
```
<meta content="email=no" name="format-detection" />
```

## 添加智能 App 广告条 Smart App Banner
告诉浏览器这个网站对应的app，并在页面上显示下载banner
```
<meta name="apple-itunes-app" content="app-id=myAppStoreID,affiliate-data=myAffiliateData, app-argument=myURL">
```

## 针对不识别Viewport的手持设备进行优化
主要是针对一些老的不识别viewport的浏览器，比如黑莓
```
<meta name="HandheldFriendly" content="true">
```

## 适配微软的老式浏览器
```
<meta name="MobileOptimized" content="320">
```

## 强制竖屏
```
<!-- uc强制竖屏 -->
<meta name="screen-orientation" content="portrait">
<!-- QQ强制竖屏 -->
<meta name="x5-orientation" content="portrait">
```

## 强制全屏
```
<!-- UC强制全屏 -->
<meta name="full-screen" content="yes">
<!-- QQ强制全屏 -->
<meta name="x5-fullscreen" content="true">
```

## windows phone 点击无高光
```
<meta name="msapplication-tap-highlight" content="no">
```





