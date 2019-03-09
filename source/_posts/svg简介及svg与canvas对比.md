---
title: svg简介及svg与canvas对比
date: 2018-07-30 10:07:05
categories: svg
tags:
- html5
- svg
---
SVG(scalable、vector、graphics)是XML语言的一种形式，有点类似XHTML，它可以用来绘制矢量图形。
<!-- more -->

# SVG的基本应用
首先需要记住的是，svg在HTML中的写法是用的XML的语法，所以：
1. SVG的元素和属性必须按标准格式书写，因为XML是区分大小写的（这一点和html不同）
2. SVG里的属性值必须用引号引起来，就算是数值也必须这样做。

# svg文件的基本属性
## svg渲染及压缩
* 元素的渲染顺序，SVG文件全局有效的规则是“后来居上”，越后面的元素越可见。
* web上的svg文件可以直接展示，或者通过以下几种方法嵌入到HTML文件：
  * HTML是XHTML，并且声明类型为application/xhtml+xml
  * HTML是HTML5，并且浏览器支持HTML5，可以直接嵌入SVG
  * 通过`object`引入SVG文件。`<object data="image.svg" type="image/svg+xml" />`
  * 通过`iframe`引入SVG文件。`<iframe src="image.svg"></iframe>`
  * 可以使用img元素，但是在低于4.0版本的Firefox中不起作用
  * 最后SVG可以通过JavaScript动态创建并注入到HTML DOM中

svg有两种文件类型，一个是".svg"，一个是压缩的".svgz"。".svgz"在Firefox不能通过本地的机器加载，微软的IIS服务器对于gzip压缩的SVG文件，在用户代理上也不是很可靠。".svgz"多用于地图应用等需要大型SVG文件的应用。

## SVG的坐标定位及窗口
svg的网格布局和canvas是一样的，都是以一个左上角为原点，向下和向右延申的。

正常情况下，svg的一个像素对应显示屏的一个像素，不过这种情况是可以改变的，我们需要定义`viewBox`。

```html
<svg width="200" height="200" viewBox="0 0 100 100">
```

上面的画板尺寸是`200px*200px`，画板展示区域是`100px*100px`，所以`100*100`的区域会在`200*200`的画布上展示。

# svg基本形状
## 矩形 rect
* x 矩形左上角的x位置
* y 矩形左上角的y位置
* width 矩形的宽度
* height 矩形的高度
* rx 圆角的x方位的半径
* ry 圆角的y方位的半径

## 圆形 circle
* r 圆的半径
* cx 圆心的x位置
* cy 圆心的y位置

## 椭圆 ellipse
* rx 椭圆的x半径
* ry 椭圆的y半径
* cx 椭圆中心的x位置
* cy 椭圆中心的y位置

## 线条 line
* x1 起点的x位置
* y1 起点的y位置
* x2 终点的x位置
* y2 终点的y位置

## 折线 polyline
Polyline是一组连接在一起的直线。因为它可以有很多的点，折线的的所有点位置都放在一个points属性中：

`points` 点集数列。每个数字用空白、逗号、终止命令符或者换行符分隔开。每个点必须包含2个数字，一个是x坐标，一个是y坐标。所以点列表 (0,0), (1,1) 和(2,2)可以写成这样：“0 0, 1 1, 2 2”。

## 多边形 polygon
polygon和折线很像，它们都是由连接一组点集的直线构成。不同的是，polygon的路径在最后一个点处自动回到第一个点。

`points` 点集数列。每个数字用空白符、逗号、终止命令或者换行符分隔开。每个点必须包含2个数字，一个是x坐标，一个是y坐标。所以点列表 (0,0), (1,1) 和(2,2)可以写成这样：“0 0, 1 1, 2 2”。路径绘制完后闭合图形，所以最终的直线将从位置(2,2)连接到位置(0,0)。

# 路径
`<path>`元素是SVG基本形状中最强大的一个，path只需要少量的点就能创建和规划流畅的曲线

path的形状通过属性`'d'`定义，属性d的值是一个“命令+参数”的序列。

每一个命令都用一个关键字母来表示，每一个命令有两种形式。一种是**大写字母**，表示采用绝对定位。另一种是**小写字母**，表示采用相对定位。

属性`d`采用的是用户坐标系统，所以不需标明单位。

## 直线命令
|命令|功能|
|---|----|
|M x y (or m dx dy)|移动画笔坐标|
|L x y (or l dx dy)|L命令将会在当前位置和新位置（L前面画笔所在的点）之间画一条线段|
|H x (or h dx)|标明在x轴移动到的位置|
|V y (or v dy)|标明在y轴移动到的位置|
|Z (or z)|从当前点画一条直线到路径的起点|

## 曲线命令
有三个命令：
* 三次贝塞尔曲线
* 二次贝塞尔曲线
* 圆的一部分

### 三次贝塞尔曲线
三次贝塞尔曲线需要定义一个点和两个控制点，所以用C命令创建三次贝塞尔曲线，需要设置三组坐标参数：

`C x1 y1, x2 y2, x y (or c dx1 dy1, dx2 dy2, dx dy)`

最后一个坐标(x, y)表示的是曲线的终点，另外两个坐标是控制点，(x1, y1)是起点的控制点，(x2, y2)是终点的控制点。

如何创建平滑的曲线呢？通常情况下，一个点是另一个点的控制点的堆成。这样，你就可以使用一个简写的贝塞尔曲线S:

`S x2 y2, x y (or s dx2 dy2, dx dy)`

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/little_tricks/ShortCut_Cubic_Bezier.png)
### 二次贝塞尔曲线
二次贝塞尔曲线只需要一个控制点。

`Q x1 y1, x y (or q dx1 dy1, dx dy)`

二次贝塞尔曲线简写`T x y (or t dx dy)`。

### 弧形
`A rx ry x-axis-rotation large-arc-flag sweep-flag x y`

`a rx ry x-axis-rotation large-arc-flag sweep-flag dx dy`

x轴旋转角度（x轴旋转角度）, large-arc-flag（角度大小） 和sweep-flag（弧线方向）

## Fill和Stroke属性
这两个属性，可以用直接的css颜色，还有一个就是直接添加颜色的透明度。属性fill-opacity控制填充色的不透明度，属性stroke-opacity控制描边的不透明度。

### 描边
![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/little_tricks/SVG_Stroke_Linecap_Example.png)

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/little_tricks/SVG_Stroke_Linejoin_Example.png)

### stroke-dasharray
是一组用逗号分割的数字组成的数列
1. 第一个用来表示填色区域的长度
2. 第二个用来表示非填色区域的长度
3. 继续定义增加长度

### 使用css
除了定义对象的属性外，你也可以通过CSS来样式化填充和描边。语法和在html里使用CSS一样，只不过你要把background-color、border改成fill和stroke。注意，不是所有的属性都能用CSS来设置。上色和填充的部分一般是可以用CSS来设置的，比如fill，stroke，stroke-dasharray等，但是不包括下面会提到的渐变和图案等功能。另外，width、height，以及路径的命令等等，都不能用css设置。判断它们能不能用CSS设置还是比较容易的。

可以利用`:hover`伪类完成触发效果。

## 渐变
线性渐变沿着直线改变颜色，要插入一个线性渐变，你需要在SVG文件的defs元素内部，创建一个`<linearGradient>` 节点。

还有径向渐变也类似，创建径向渐变需要在文档的defs中添加一个`<radialGradient>`元素。

## 图案
patterns（图案）是SVG中用到最让人混淆的填充类型。

## 剪切和遮罩
Clipping用来移除在别处定义的元素的部分内容。在这里，任何半透明效果都是不行的。它只能要么显示要么不显示。

Masking允许使用透明度和灰度值遮罩计算得的软边缘。

Web开发工具箱中有一个很有用的工具是display:none。它虽然几无悬念，但是依然可以在SVG上使用该CSS属性，连同CSS2定义的visibility和clip属性。为了恢复以前设置的display:none，知道这一点很重要：所有的SVG元素的初始display值都是inline。

## 其它SVG内容
### 嵌入光栅图像

    很像在HTML中的img元素，SVG有一个image元素，用于同样的目的。你可以利用它嵌入任意光栅（以及矢量）图像。它的规格要求应用至少支持PNG、JPG和SVG格式文件。
    嵌入的图像变成一个普通的SVG元素。这意味着，你可以在其内容上用剪切、遮罩、滤镜、旋转以及其它SVG工具：

### 嵌入任意XML
因为SVG是一个XML应用，所以你总是可以在SVG文档的任何位置嵌入任意XML。但是你没有必要定义周围的SVG需要怎样反作用于这个内容。

## 滤镜效果
有一种情况，基本形状不能提供你想要达到的效果的灵活性。投阴影、提供一个十分流行的示例，利用一个小变组合无法合理地创建它。滤镜是SVG的机制，允许创建精密的效果。

## 字体
字体中需要了解的就是[text-path](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/textPath),即tref样式化。

# svg和canvas对比
这部分参考W3school

## Canvas
* 依赖分辨率
* 不支持事件处理器
* 弱的文本渲染能力
* 能够以 .png 或 .jpg 格式保存结果图像
* 最适合图像密集型的游戏，其中的许多对象会被频繁重绘

## SVG
* 不依赖分辨率
* 支持事件处理器
* 最适合带有大型渲染区域的应用程序（比如谷歌地图）
* 复杂度高会减慢渲染速度（任何过度使用 DOM 的应用都不快）
* 不适合游戏应用

> 参考
> * [MDN的svg教程](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Tutorial/Introduction)
> * [W3school的svg教程](http://www.w3school.com.cn/svg/svg_intro.asp)