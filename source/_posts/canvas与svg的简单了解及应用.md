---
title: canvas的简单了解及应用
date: 2018-07-28 15:22:33
categories: canvas
tags: 
- html5
- canvas
---
这个星期抽空看了下canvas，之前一直就对这两个东西望而却步，连简单的api和应用都没有了解。

工作中看到了[html2canvas](https://html2canvas.hertzen.com/)这个简单的工具可以根据DOM生成截图，于是抽空将MDN上的[canvas](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial)教程撸了一遍。
<!-- more -->
# canvas基本概念
`<canvas>`是HTML5新增的元素，可以通过JavaScript中的脚本来绘制图形。例如，它可以绘制图形，制作照片，创建动画，甚至可以进行实时视频处理或渲染。时间太短，没有看canvas的3d部分，下面的介绍都是关于canvas的2d部分的。

首先需要知道的是canvas的绘图栅格，canvas和svg一样，坐标都是一样的，坐标都是从左上角起始：

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/little_tricks/Canvas_default_grid.png)

# 绘制相关
|方法|功能|
|----|---|
|fillRect(x, y, width, height)|绘制一个填充的矩形|
|strokeRect(x, y, width, height)|绘制一个矩形的边框|
|clearRect(x, y, width, height)|清除指定矩形区域，让清除部分完全透明|
|beginPath()|新建一条路径，生成之后，图形绘制命令被指向到路径上生成路径|
|closePath()|闭合路径之后图形绘制命令又重新指向到上下文中|
|stroke()|通过线条来绘制图形轮廓|
|moveTo(x, y)|将笔触移动到指定的坐标x以及y上|
|lineTo(x, y)|绘制一条从当前位置到指定x以及y位置的直线|
|arc(x, y, radius, startAngle, endAngle, anticlockwise)|画一个以（x,y）为圆心的以radius为半径的圆弧（圆），从startAngle开始到endAngle结束，按照anticlockwise给定的方向（默认为顺时针）来生成|
|arcTo(x1, y1, x2, y2, radius)|根据给定的控制点和半径画一段圆弧，再以直线连接两个控制点|
|quadraticCurveTo(cp1x, cp1y, x, y)|绘制二次贝塞尔曲线，cp1x,cp1y为一个控制点，x,y为结束点|
|bezierCurveTo(cp1x, cp1y, cp2x, cp2y, x, y)|绘制三次贝塞尔曲线，cp1x,cp1y为控制点一，cp2x,cp2y为控制点二，x,y为结束点|
|rect(x, y, width, height)|绘制一个左上角坐标为（x,y），宽高为width以及height的矩形|
|Path2D.addPath(path [, transform])​|添加了一条路径到当前路径（可能添加了一个变换矩阵）|
|fillStyle = color|设置图形的填充颜色|
|strokeStyle = color|设置图形轮廓的颜色|
|globalAlpha = transparencyValue|这个属性影响到 canvas 里所有图形的透明度，有效的值范围是 0.0 （完全透明）到 1.0（完全不透明），默认是 1.0|
|lineWidth = value|设置线条宽度|
|lineCap = type|设置线条末端样式(butt，round和square)|
|lineJoin = type|设定线条与线条间接合处的样式(round，bevel和miter)|
|miterLimit = value|限制当两条线相交时交接处最大长度；所谓交接处长度（斜接长度）是指线条交接处内角顶点到外角顶点的长度|
|getLineDash()|返回一个包含当前虚线样式，长度为非负偶数的数组|
|setLineDash(segments)|设置当前虚线样式|
|lineDashOffset = value|设置虚线样式的起始偏移量|
|createLinearGradient(x1, y1, x2, y2)|createLinearGradient 方法接受 4 个参数，表示渐变的起点 (x1,y1) 与终点 (x2,y2)|
|createRadialGradient(x1, y1, r1, x2, y2, r2)|createRadialGradient 方法接受 6 个参数，前三个定义一个以 (x1,y1) 为原点，半径为 r1 的圆，后三个参数则定义另一个以 (x2,y2) 为原点，半径为 r2 的圆|
|gradient.addColorStop(position, color)|addColorStop 方法接受 2 个参数，position 参数必须是一个 0.0 与 1.0 之间的数值，表示渐变中颜色所在的相对位置。例如，0.5 表示颜色会出现在正中间。color 参数必须是一个有效的 CSS 颜色值（如 #FFF， rgba(0,0,0,1)，等等）|
|createPattern(image, type)|该方法接受两个参数。Image 可以是一个 Image 对象的引用，或者另一个 canvas 对象。Type 必须是下面的字符串值之一：repeat，repeat-x，repeat-y 和 no-repeat|
|shadowOffsetX/Y = float|shadowOffsetX 和 shadowOffsetY 用来设定阴影在 X 和 Y 轴的延伸距离，它们是不受变换矩阵所影响的。负值表示阴影会往上或左延伸，正值则表示会往下或右延伸，它们默认都为 0|
|shadowBlur = float|shadowBlur 用于设定阴影的模糊程度，其数值并不跟像素数量挂钩，也不受变换矩阵的影响，默认为 0|
|shadowColor = color|shadowColor 是标准的 CSS 颜色值，用于设定阴影颜色效果，默认是全透明的黑色|
|fillText(text, x, y [, maxWidth])|在指定的(x,y)位置填充指定的文本，绘制的最大宽度是可选的|
|strokeText(text, x, y [, maxWidth])|在指定的(x,y)位置绘制文本边框，绘制的最大宽度是可选的|
|font = value|当前我们用来绘制文本的样式. 这个字符串使用和 CSS font 属性相同的语法. 默认的字体是 10px sans-serif|
|textAlign = value|文本对齐选项. 可选的值包括：start, end, left, right or center. 默认值是 start|
|textBaseline = value|基线对齐选项. 可选的值包括：top, hanging, middle, alphabetic, ideographic, bottom。默认值是 alphabetic|
|direction = value|文本方向。可能的值包括：ltr, rtl, inherit。默认值是 inherit|

# 绘制和使用图片
canvas具有图像操作能力，可以用于动态的图像合成或者作为图像的背景，已经游戏界面等等。

引入图像到canvas需要下面两个操作
1. 获得一个指向HTMLImageElement的对象或者另一个canvas元素的引用作为源，也可以通过提供一个URL的方式来使用图片
2. 使用drawImage()函数将图片绘制到画布上

## 可获取图片类型
* **HTMLImageElement**

  这些图片是由Image()函数构造出来的，或者任何的 `<img>` 元素

* **HTMLVideoElement**

  用一个HTML的 `<video>` 元素作为你的图片源，可以从视频中抓取当前帧作为一个图像

* **HTMLCanvasElement**

  可以使用另一个 `<canvas>` 元素作为你的图片源

* **ImageBitmap**

  这是一个高性能的位图，可以低延迟地绘制，它可以从上述的所有源以及其它几种源中生成

## 使用图像
我们可以创建一个新的`HTMLImageElement`元素，或者直接使用相同页面的图片。如果是其他域的图片，图片需要使用`crossOrigin`属性，需要图片的服务器允许跨域访问这个图片。

```js
var img = new Image();   // 创建img元素
img.onload = function(){
  // 执行drawImage语句
}
img.src = 'myImage.png'; // 设置图片源地址
```

新建的图片中，如果图片还没有加载，我们调用`drawImage()`什么都不会发生，所以我们的drawImage需要在`img.onload`里面执行，确保图片加载完毕。

还有一种方法就是通过`data:url`方式嵌入图片，可以使用base64格式。

## 使用视频帧
```js
function getMyVideo() {
  var canvas = document.getElementById('canvas');
  if (canvas.getContext) {
    var ctx = canvas.getContext('2d');

    return document.getElementById('myvideo');
  }
}
```

## 绘制图片
我们获取了源图对象，可以使用`drawImage`方法将它渲染到canvas里。

|方法|描述|
|-----|---|----|
|drawImage(image, x, y)|其中 image 是 image 或者 canvas 对象，x 和 y 是其在目标 canvas 里的起始坐标|
|drawImage(image, x, y, width, height)|这个方法多了2个参数：width 和 height，这两个参数用来控制 当向canvas画入时应该缩放的大小|
|drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight)|第一个参数和其它的是相同的，都是一个图像或者另一个 canvas 的引用。其它8个参数最好是参照右边的图解，前4个是定义图像源的切片位置和大小，后4个则是定义切片的目标显示位置和大小。|

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/little_tricks/Canvas_drawimage.jpg)

# canvas变形
变形之前，了解两个状态的保存和恢复`save()`和`restore()`。

save 和 restore 方法是用来保存和恢复 canvas 状态的，都没有参数。Canvas 的状态就是当前画面应用的所有样式和变形的一个快照。

每一次调用 restore 方法，上一个保存的状态就从栈中弹出，所有设定都恢复。在做变形之前先保存状态是一个良好的习惯，变形也是相对于整个画板的。

|方法|描述|
|---|---|
|translate(x, y)|translate 方法接受两个参数。x 是左右偏移量，y 是上下偏移量|
|rotate(angle)|这个方法只接受一个参数：旋转的角度(angle)，它是顺时针方向的，以弧度为单位的值|
|scale(x, y)|scale 方法接受两个参数。x,y 分别是横轴和纵轴的缩放因子，它们都必须是正值。值比 1.0 小表示缩小，比 1.0 大则表示放大，值为 1.0 时什么效果都没有|
|transform(m11, m12, m21, m22, dx, dy)|这个方法是将当前的变形矩阵乘上一个基于自身参数的矩阵|
|setTransform(m11, m12, m21, m22, dx, dy)|这个方法会将当前的变形矩阵重置为单位矩阵，然后用相同的参数调用 transform 方法。如果任意一个参数是无限大，那么变形矩阵也必须被标记为无限大，否则会抛出异常。从根本上来说，该方法是取消了当前变形,然后设置为指定的变形,一步完成。|
|resetTransform()|重置当前变形为单位矩阵|

# 合成和裁切
globalCompositeOperation 属性设置或返回如何将一个源（新的）图像绘制到目标（已有）的图像上。

源图像 = 您打算放置到画布上的绘图。

目标图像 = 您已经放置在画布上的绘图。

|值|描述|
|---|---|
|source-over|默认。在目标图像上显示源图像。|
|source-atop|在目标图像顶部显示源图像。源图像位于目标图像之外的部分是不可见的。|
|source-in|在目标图像中显示源图像。只有目标图像内的源图像部分会显示，目标图像是透明的。|
|source-out|在目标图像之外显示源图像。只会显示目标图像之外源图像部分，目标图像是透明的。|
|destination-over|在源图像上方显示目标图像。|
|destination-atop|在源图像顶部显示目标图像。源图像之外的目标图像部分不会被显示。|
|destination-in|在源图像中显示目标图像。只有源图像内的目标图像部分会被显示，源图像是透明的。|
|destination-out|在源图像外显示目标图像。只有源图像外的目标图像部分会被显示，源图像是透明的。|
|lighter|显示源图像 + 目标图像。|
|copy|显示源图像。忽略目标图像。|
|xor|使用异或操作对源图像与目标图像进行组合。|

## 裁切路径

`clip()`裁切路径和普通的 `canvas` 图形差不多，不同的是它的作用是遮罩，用来隐藏不需要的部分。如右图所示。红边五角星就是裁切路径，所有在路径以外的部分都不会在 `canvas` 上绘制出来。

# canvas动画
首先了解一下，canvas动画的基本步骤：
1. 清空canvas

  除非接下来要画的内容会完全充满 canvas （例如背景图），否则你需要清空所有。最简单的做法就是用 clearRect 方法。

2. 保存canvas状态

  如果你要改变一些会改变 canvas 状态的设置（样式，变形之类的），又要在每画一帧之时都是原始状态的话，你需要先保存一下。

3. 绘制动画图形

  这一步是重画动画帧。

4. 回复canvas状态

  如果已经保存了canvas的状态，可以先恢复它，然会重绘下一帧。

## 操作动画
实现动画，我们需要一些定时重绘的方法。简单的有两个方法`setInterval`和`setTimeout`方法来控制在设定的时间点上执行重绘。

可以用`window.setInterval()`, `window.setTimeout()`,和`window.requestAnimationFrame()`来设定定期执行一个指定函数

* `setInterval(function, delay)` 当设定好间隔时间后，function会定期执行。
* `setTimeout(function, delay)` 在设定好的事件之后执行函数。
* `requestAnimationFrame(callback)` 告诉浏览器你希望执行一个动画，并在重绘之前，请求浏览器执行一个特定的函数来更新动画。

如果不需要和用户互动，可以使用`setInterval()`方法，它可以定期的执行指定代码。如果我们需要做一个游戏，我们可以使用键盘或者鼠标事件配合`setTimeout()`方法来实现。通过设置事件监听，我们可以捕捉用户的交互，并执行相应的动作。

`window.requestAnimationFrame()`实现动画效果。这个方法提供了更加平滑更加有效率的方式来执行动画，当系统准备好了重绘条件的时候，才调用绘制动画帧。一般每秒钟回调函数执行60次，也有可能会被降低。

还有一个`window.cancelAnimationFrame()`用于取消动画。

# 像素操作
执行像素操作，我们需要了解一下ImageData对象。

ImageData对象中存储着canvas对象真实的像素数据，它包含以下几个只读属性：

* **width** 图片宽度，单位是像素
* **height** 图片高度，单位是像素
* **data** `Uint8ClampedArray`类型的一维数组，包含着RGBA格式的整型数据，范围在0至255之间（包括255）

data属性返回一个`Uint8ClampedArray`,它可以被使用作为查看初始像素数据。每个像素用四个1bytes值（按照red，green，blue和alpha），rgba的顺序来排列。

每个部分的颜色值部分用0至255来表示，每个部分被分配到一个数组内连续的索引，左上角像素的红色部分在数组的索引0位置。像素从左到右被处理，然后往下，遍历整个数组。

如要读取，第50行，第200列的像素的蓝色部分，你会写以下代码：

```js
blueComponent = imageData.data[((50*(imageData.width*4)) + (200*4)) + 2];
```

## 创建一个ImageData对象
去创建一个新的，空白的ImageData对象，你应该会使用`createImageData()`方法。

```js
var myImageData = ctx.createImageData(width, height);
// 创建了一个新的具体特定尺寸的ImageData对象。所有像素被预设为透明黑

var myImageData = ctx.createImageData(anotherImageData)
// 尺寸相同，依旧是透明黑
```

## 得到场景像素数据
```js
// 获取指定区域的画布对象
var myImageData = ctx.getImageData(left, top, width, height)
```

下面看一个颜色选择器的代码

```js
var img = new Image();
img.src = './cat.jpg';
var canvas = document.getElementById('canvas');
var ctx = canvas.getContext('2d');
img.onload = function() {
  ctx.drawImage(img, 0, 0);
  img.style.display = 'none';
};
var color = document.getElementById('color');
function pick(event) {
  var x = event.layerX;
  var y = event.layerY;
  var pixel = ctx.getImageData(x, y, 1, 1);
  var data = pixel.data;
  var rgba = 'rgba(' + data[0] + ',' + data[1] + ',' + data[2] + ',' + (data[3] / 255) + ')';
  color.style.background =  rgba;
  color.textContent = rgba;
}
canvas.addEventListener('mousemove', pick);
```

## 在场景中写入数据
可以利用putImageData()方法对场景进行像素数据的写入。

`ctx.putImageData(myImageData, dx, dy);`

dx和dy参数表示你希望在场景内左上角绘制的像素数据得到的设备坐标。

## 缩放和反锯齿
`imageSmoothingEnabled`默认是启用的，想要关闭需要添加不同的浏览器前缀来手动关闭。

# 保存图片
`HTMLCanvasElement`提供一个toDataURL()方法，此方法在保存图片的时候有用。

|方法|描述|
|---|----|
|canvas.toDataURL('image/png')|默认设定。创建一个PNG图片。|
|canvas.toDataURL('image/jpeg', quality)|创建一个JPG图片。你可以有选择地提供从0到1的品质量，1表示最好品质，0基本不被辨析但有比较小的文件大小。|
|canvas.toBlob(callback, type, encoderOptions)|这个创建了一个在画布中的代表图片的Blob对象|

# 性能优化
* 在离屏canvas上预渲染相似的图形或重复的对象
* 避免浮点数的坐标点，用整数取而代之
* 不要在用drawImage时缩放图像
* 使用多层画布去画一个复杂的场景
* 用CSS设置大的背景图
* 用CSS transforms特性缩放画布
* 使用moz-opaque属性(仅限Gecko)

## 更多贴士
* 将画布的函数调用集合到一起（例如，画一条折线，而不要画多条分开的直线）
* 避免不必要的画布状态改变
* 渲染画布中的不同点，而非整个新状态
* 尽可能避免 shadowBlur特性
* 尽可能避免text rendering
* 使用不同的办法去清除画布(clearRect() vs. fillRect() vs. 调整canvas大小)
*  有动画，请使用window.requestAnimationFrame() 而非window.setInterval()
* 请谨慎使用大型物理库
* 用[JSPerf](https://jsperf.com/)测试性能