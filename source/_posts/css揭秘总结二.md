---
title: css揭秘总结(二)
date: 2018-01-02 20:49:13
categories: CSS
tags: CSS
---
主要总结了下CSS３的形状，视觉效果。
<!-- more -->
# 形状
## 9. 自适应的椭圆
> 背景知识：`border-radius`属性的基本用法

### 难题
一个圆很容易生成，设置一个足够大的`border-radius`就可以了,如果我们希望一个自适应的椭圆呢，根据内容自动调整并适应。如果它的宽高相等，就显示为一个圆；如果宽高不等，就显示为一个椭圆。

### 解决方案
border-radius，可以单独指定水平和垂直半径，只要用一个斜杠分开。
```css
border-radius: 50% 50%;
/* 简写 */
border-radius: 50%;
```

### 其他椭圆
我们可以为所有四个角提供完全不同的水平和垂直半径，方法是在斜杠前指定 1~4 个值，在斜杠后指定另外 1~4 个值。
```css
/* 半椭圆 */
border-radius: 50% / 100% 100% 0 0;
/* 沿纵轴劈开的半椭圆 */
border-radius: 100% 0 0 100% / 50%;
/* 四分之一椭圆 */
border-radius: 100% 0 0 0;
```
![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/csssercets/半椭圆.png) ![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/csssercets/四分之一椭圆.png) ![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/csssercets/纵轴半椭圆.png)

## 10. 平行四边形
> 背景知识：基本的CSS 变形

### 难题
平行四边形可以通过`skew()`的变形属性来对这个矩形进行斜向拉伸，这导致它的内容也发生了斜向变形，这很不好看，而且难读，有没有办法只让容器的形状倾斜，而保持其内容不变呢？

### 嵌套元素方案
我们可以对内容再应用一次反向的 `skew()` 变形，从而抵消容器的变形效果, 缺点：需要添加一层层额外的 HTML 元素来包裹内容。

### 伪元素方案
```css
.button {
 position: relative;
 /* 其他的文字颜色、内边距等样式…… */
}
.button::before {
 content: ''; /* 用伪元素来生成一个矩形 */
 position: absolute;
 top: 0; right: 0; bottom: 0; left: 0;
 z-index: -1;
 background: #58a;
 transform: skew(45deg);
}
```

适用于其他任何变形样式，当我们想变形一个元素而不想变形它的内容时就可以用到它

## 11. 菱形图片
> 背景知识: CSS 变形，“平行四边形”

### 难题
在视觉设计中，把图片裁切为菱形是一种常见的设计手法，但在 `CSS`中还没有一种简单直观的方法来实现它。事实上，直到最近，这种效果才基本成为可能。当网页设计师想要实现这种设计风格时，他们通常不希望在图像处理软件中预先把图片裁好。显然不用说你也知道，这个方法的可维护性并不好。如果未来有人想修改图片风格，将很难增加其他效果，而且最终往往会搞得一团糟。

### 基于变形的方案
主要的思路与前一篇攻略“平行四边形”中讨论的第一个解决方案一致：需要把图片用一个 <div> 包裹起来，然后对其应用相反的 `rotate()`变形样式，这个方法不是推荐的方法。

### 裁切路径方案
它的主要思路是使用 `clip-path` 属性。这个特性也是从 `SVG` 那里借鉴而来，已经可以应用在 `HTML` 元素上了（至少对于支持的浏览器来说是这样的）。

我们将会使用 polygon()（多边形）函数来指定一个菱形。实际上，它允许我们用一系列（以逗号分隔的）坐标点来指定任意的多边形。
```css
img {
 clip-path: polygon(50% 0, 100% 50%,
 50% 100%, 0 50%);
 transition: 1s clip-path;
}
img:hover {
 clip-path: polygon(0 0, 100% 0,
 100% 100%, 0 100%);
}
```

## 12. 切角效果
> 背景知识: CSS 渐变，background-size，“条纹背景”

### 难题
最常见的形态是把元素的一个或多个角切成 45°的缺口（也称作斜面切角）。

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/csssercets/切角.png)

### 解决方案
第一种方案来自于无所不能的 CSS 渐变。
```css
background: #58a;
background:
 linear-gradient(135deg, transparent 15px, #58a 0)
 top left,
 linear-gradient(-135deg, transparent 15px, #58a 0)
 top right,
 linear-gradient(-45deg, transparent 15px, #58a 0)
 bottom right,
 linear-gradient(45deg, transparent 15px, #58a 0)
 bottom left;
background-size: 50% 50%; 
/* 如果不设置的话，背景会相互覆盖 */
background-repeat: no-repeat;
/* 如果不设置，每层渐变图案各自平铺了两次 */
```
现在我们会得到，切了四个角的div。不过可维护性并不高。

```scss
@mixin beveled-corners($bg,
 $tl:0, $tr:$tl, $br:$tl, $bl:$tr) {
 background: $bg;
 background:
 linear-gradient(135deg, transparent $tl, $bg 0)
 top left,
 linear-gradient(225deg, transparent $tr, $bg 0)
 top right,
 linear-gradient(-45deg, transparent $br, $bg 0)
 bottom right,
 linear-gradient(45deg, transparent $bl, $bg 0)
 bottom left;
 background-size: 50% 50%;
 background-repeat: no-repeat;
}
// 调用 传入 2~5 个参数：
@include beveled-corners(#58a, 15px, 5px);
```

### 弧形切角
很多人也把这种效果称为“内凹圆角”，因为它看起来就像是圆角的反向版本。用径向渐变来替代上述线性渐变：
```css
background: #58a;
background:
 radial-gradient(circle at top left,
 transparent 15px, #58a 0) top left,
 radial-gradient(circle at top right,
 transparent 15px, #58a 0) top right,
 radial-gradient(circle at bottom right,
 transparent 15px, #58a 0) bottom right,
 radial-gradient(circle at bottom left,
 transparent 15px, #58a 0) bottom left;
background-size: 50% 50%;
background-repeat: no-repeat;
```

### 内联 SVG 与 border-image 方案
```css
border: 20px solid transparent;
border-image: 1 url('data:image/svg+xml,\
 <svg xmlns="http://www.w3.org/2000/svg"\
 width="3" height="3" fill="%2358a">\
 <polygon points="0,1 1,0 2,0 3,1 3,2 2,3 1,3 0,2"/>\
 </svg>');
background: #58a;
background-clip: padding-box;
```

### 裁切路径方案
利用`clip-path`属性。
```css
background: #58a;
clip-path: polygon(
 20px 0, calc(100% - 20px) 0, 100% 20px,
 100% calc(100% - 20px), calc(100% - 20px) 100%,
 20px 100%, 0 calc(100% - 20px), 0 20px
);
```
### 补充多边形裁剪
https://www.w3cplus.com/preprocessor/creat-css-polygon-wiht-border-and-clip-path-property.html

## 13. 梯形标签页
> 背景知识: 基本的 3D 变形，“平行四边形”

### 难题
一直以来，梯形都是众所周知难以用 CSS 生成的形状，尽管它也十分常用，尤其是对于标签页来说。

### 解决方案
利用 CSS 中用 3D 旋转来模拟出这个效果`transform: perspective(.5em) rotateX(5deg);`。

对元素使用了 3D变形之后，其内部的变形效应是“不可逆转”的, 这里我们像上一节利用伪类就好了。而给元素应用变形效果会让这个元素以它自身的中心线为轴进行空间上的旋转。所以我们需要设置`transform-origin`属性。`transform-origin:bottom;`，当它在 3D 空间中旋转时，可以把它的底边固定住。
```css
nav > a {
 position: relative;
 display: inline-block;
 padding: .3em 1em 0;
}
nav > a::before {
 content: '';
 position: absolute;
 top: 0; right: 0; bottom: 0; left: 0;
 z-index: -1;
 background: #ccc;
 background-image: linear-gradient(
 hsla(0,0%,100%,.6),
 hsla(0,0%,100%,0));
 border: 1px solid rgba(0,0,0,.4);
 border-bottom: none;
 border-radius: .5em .5em 0 0;
 box-shadow: 0 .15em white inset;
 transform: perspective(.5em) rotateX(5deg);
 transform-origin: bottom;
}
```

## 14. 简单的饼图
> 背景知识: CSS 渐变，基本的 SVG，CSS 动画，“条纹背景”，“自适应的椭圆”

### 难题
饼图在网页中的运用极为普遍，比如简单的统计图表、进度指示器、定时器等。

### 基于 transform 的解决方案
简单来说就是三个遮罩层的互相重叠，这里直接放代码。
```css
.pie {
 position: relative;
 width: 100px;
 line-height: 100px;
 border-radius: 50%;
 background: yellowgreen;
 background-image:
 linear-gradient(to right, transparent 50%, #655 0);
 color: transparent;
 text-align: center;
}
@keyframes spin {
 to { transform: rotate(.5turn); }
}
@keyframes bg {
 50% { background: #655; }
}
.pie::before {
 content: '';
 position: absolute;
 top: 0; left: 50%;
 width: 50%; height: 100%;
 border-radius: 0 100% 100% 0 / 50%;
 background-color: inherit;
 transform-origin: left;
 animation: spin 50s linear infinite,
 bg 100s step-end infinite;
 animation-play-state: paused;
 animation-delay: inherit;
}
```

### SVG 解决方案
一个圆形：
```html
<svg width="100" height="100">
<circle r="30" cx="50" cy="50" />
</svg>
```

基本样式
```css
circle {
 fill: yellowgreen;
 stroke: #655; 
 /* stroke: 描边颜色 */
 stroke-width: 30;
 /* stroke-width：描边宽度 */
}
```
这里我们还需要了解一个属性`stroke-dasharray`，它是为虚线描边而准备的。`stroke-dasharray`，第一个参数是线段长度，第二个是间隙长度。当我们把这个虚线描边的线段长度指定为 0，并且把虚线间隙的长度设置为等于或大于整个圆周的长度时，答案就会浮出水面了。（这里做个简单的计算，圆形的周长 C = 2πr，因此在这里C= 2π × 30 ≈ 189。）

这里直接放出优化版本。

我们可以给这个圆形指定一个特定的半径，从而让它的周长无限接近 100，这样就可以直接把比率的百分比值指定为 `strokedasharray`的长度，不需要做任何计算了。
```html
<svg viewBox="0 0 32 32">
 <circle r="16" cx="16" cy="16" />
</svg>
```
```css
svg {
 width: 100px; height: 100px;
 transform: rotate(-90deg);
 background: yellowgreen;
 border-radius: 50%;
}
circle {
 fill: yellowgreen;
 stroke: #655;
 stroke-width: 32;
 stroke-dasharray: 38 100; /* 可得到比率为38%的扇区 */
}
```

# 视觉效果

## 15. 单侧投影

### 难题
如何只在元素的一侧（偶尔是两侧）设置投影。

### 单侧投影
![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/csssercets/box_shadow.png)

上图演示了投影生成的过程。

最终的解决方案来自 `box-shadow` 鲜为人知的第四个长度参数, 这是我们需要了解的。这个参数会根据你指定的值去扩大或（当指定负值时）缩小投影的尺寸。

```css
box-shadow: 0 5px 4px -4px black;
```

### 邻边投影
一层藏起来，另一侧自然漏出。
```css
box-shadow: 3px 3px 6px -3px black;
```

### 双侧投影
```css
box-shadow: 5px 0 5px -5px black,
            -5px 0 5px -5px black;
```

## 16. 不规则投影
> 背景知识: box-shadow

### 难题
当我们想给一个矩形或其他能用 border-radius 生成的形状（在“自适应的椭圆”一节中可以看到一些示例）加投影时，box-shadow 的表现都堪称完美。但是，当元素添加了一些伪元素或半透明的装饰之后，它就有些力不从心了，因为 border-radius 会无耻地忽视透明部分。

### 解决方案
滤镜效果规范（http://w3.org/TR/filter-effects）为这个问题提供了一个解决方案。`filter`就是我们需要的属性，我们甚至可以用多个滤镜。

drop-shadow() 滤镜可接受的参数基本上跟 box-shadow 属性是一样的，不包括 inset 关键字，也不支持逗号分割的多层投影语法。 

我们需要知道的就是，任何非透明的部分都会被一视同仁地打上投影。小心文本的两层阴影。

## 17. 染色效果
> 背景知识: HSL 色彩模型，background-size

### 难题
为一幅灰度图片（或是被转换为灰度模式的彩色图片）增加染色效果（color tint），是一种流行且优雅的方式，可以给一系列风格迥异的照片带来视觉上的一致性。

常见的解决方法就是给图层上面添加一层半透明的实色背景，但减少了图片的对比度。

### 基于滤镜的方案
多种滤镜组合。

https://developer.mozilla.org/zh-CN/docs/Web/CSS/filter

### 基于混合模式的方案
“混合模式”控制了上层元素的颜色与下层颜色进行混合的方式。要对一个元素设置混合模式，有两个属性可以派上用场：`mix-blendmode`可以为整个元素设置混合模式，`background-blend-mode` 可以为每层背景单独指定混合模式。

缺点：
* 图片的尺寸需要在 CSS 代码中写死。
* 在语义上，这个元素并不是一张图片，因此并不会被读屏器之类的设备读出来。

## 18. 毛玻璃效果
> 背景知识: RGBA/HSLA 颜色

### 难题
半透明颜色最初的使用场景之一就是作为背景。将其叠放在照片类或其他花哨的背层之上，可以减少对比度，确保文本的可读性。这种效果确实很有视觉冲击力，但仍然可能导致文字很难阅读。

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/csssercets/transparentPic1.png) 

### 解决方案
```css
body, main::before {
 background: url("tiger.jpg") 0 / cover fixed;
}
main {
 position: relative;
 background: hsla(0,0%,100%,.3);
 overflow: hidden;
}
main::before {
 content: '';
 position: absolute;
 top: 0; right: 0; bottom: 0; left: 0;
 filter: blur(20px);
 margin: -30px;
}
```
问题：
* 伪元素现在就覆盖在内容元素之上。可以用 z-index: -1; 来修正这个问题。
* `blur`模糊效果在中心区域看起来非常完美，但在接近边缘处会逐渐消退。为了补偿这种情况，我们需要让伪元素相对其宿主元素的尺寸再向外扩大至少 20px（即它的模糊半径）。这里取30px。
* 上一个解决方法会导致一圈模糊效果超出了容器。只要对 main 元素应用overflow: hidden;，就可以把多余的模糊区域裁切掉了。

最终得到上述代码。

## 19. 折角效果
> 背景知识: CSS 变形，CSS 渐变，“切角效果“

### 难题
把元素的一个角（通常是右上角或右下角）处理为类似折角的形状，再配上或多或少的拟物样式。

### 45°折角的解决方案
我们会从一个右上角具有斜面切角的元素开始，这个切角是由“切角效果”一节中的渐变方案实现的。
```css
background: #58a; /* 回退样式 */
background:
 linear-gradient(-135deg, transparent 2em, #58a 0);
```

接下来所需要做的就是增加一个暗色的三角形来实现翻折效果。注意长度斜的。
```css
background: #58a; /* 回退样式 */
background:
 linear-gradient(to left bottom,
 transparent 50%, rgba(0,0,0,.4) 0)
 no-repeat 100% 0 / 2em 2em,
 linear-gradient(-135deg,
 transparent 1.5em, #58a 0);
```

### 其他角度的解决方案
直接放`mixin`
```scss
@mixin folded-corner($background, $size,
 $angle: 30deg) {
position: relative;
background: $background; /* 回退样式 */
background:
 linear-gradient($angle - 180deg,
 transparent $size, $background 0);
border-radius: .5em;
$x: $size / sin($angle);
$y: $size / cos($angle);
&::before {
 content: '';
 position: absolute;
 top: 0; right: 0;
 background: linear-gradient(to left bottom,
 transparent 50%, rgba(0,0,0,.2) 0,
 rgba(0,0,0,.4)) 100% 0 no-repeat;
 width: $y; height: $x;
 transform: translateY($y - $x)
 rotate(2*$angle - 90deg);
 transform-origin: bottom right;
 border-bottom-left-radius: inherit;
 box-shadow: -.2em .2em .3em -.1em rgba(0,0,0,.2);
}
}
/* 当调用时... */
.note {
 @include folded-corner(#58a, 2em, 40deg);
}
```
