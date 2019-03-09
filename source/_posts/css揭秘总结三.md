---
title: css揭秘总结(三)
date: 2018-01-03 18:16:27
categories: CSS
tags: CSS
---
主要总结了书里的字体排版，用户体验，结构与布局。字体排版由于介绍英文比较多，这里就直接跳过了几个段落。
<!-- more -->
# 字体排印
## 20. 连字符断行
CSS 文本（第三版）引入了一个新的属性 hyphens。它接受三个值：none、manual 和 auto。manual 是它的初始值，其行为正好对应了现有的工作方式：我们可以在任何时候手工插入软连字符，来实现断词折行的效果。
```css
hyphens: auto;
```

## 21. 插入换行
通过 CSS 来插入换行的需求通常与定义列表有关。

利用伪类。

```css
dd + dd::before {
 content: ', ';
 margin-left: -.25em;
 font-weight: normal;
}
```

## 22. 文本行的斑马条纹
> 背景知识: CSS 渐变，background-size，“条纹背景”，“灵活的背景定位”

### 难题
```css
tr:nth-child(even) {
 background: rgba(0,0,0,.2);
}
```

表格的“斑马条纹”，只需要一个伪类的选择就可以了。而文本呢。

### 解决方案
我们可以在CSS 中用渐变直接生成背景图像，为了让背景自动跟着内边距的宽度走，我们需要在解析 `background-position`时以 `content box` 的外沿作为基准。
```css
padding: .5em;
line-height: 1.5;
background: beige;
background-size: auto 3em;
background-origin: content-box;
background-image: linear-gradient(rgba(0,0,0,.2) 50%,
 transparent 0);
```

## 23. 调整 tab 的宽度
### 难题
调整网页code的宽度。

利用新属性。

```css
pre {
 tab-size: 2;
}
```

## 24. 连字
`font-variant-ligatures`来控制启用所有可能连字。

```css
font-variant-ligatures: common-ligatures
                        discretionary-ligatures
                        historical-ligatures;
```

## 25. 华丽的 & 符号
> 背景知识: 通过 @font-face 

规则实现基本的字体嵌入我们通常会在 font-family 声明中同时指定多个字体（即字体队列）。这样，即使我们指定的最优先字体不可用，浏览器还可以回退到其他符合整体设计风格的字体。

在这个规则之下，如果有一款字体只包含一个字符（你肯定猜到是哪个了吧），那这款字体将只用于显示这个字符，其他字符会由字体队列中排在第二位、第三位或更后面的字体来显示。

```css
@font-face {
 font-family: Ampersand;
 src: url("fonts/ampersand.woff");
}
h1 {
 font-family: Ampersand, Helvetica, sans-serif;
}
```

**还需要一个描述符**，`unicode-range`，要查出你想指定的这些字符的十六进制码位。

```css
@font-face {
 font-family: Ampersand;
 src: local('Baskerville'),
 local('Goudy Old Style'),
 local('Palatino'),
 local('Book Antiqua');
 unicode-range: U+26;
}
h1 {
 font-family: Ampersand, Helvetica, sans-serif;
}
```

## 26. 自定义下划线
> 背景知识: CSS 渐变，background-sizetext-shadow，“条纹背景”

### 难题
默认太丑，`text-decoration: underline;`。


`border-bottom`，会阻止正常的文本换行行为。`box-shadow: 0 -1px gray inset;`类似一样的会产生上述问题。

### 解决方案
最佳方案来自于`background-image` 及其相关属性。
```css
background: linear-gradient(gray, gray) no-repeat;
background-size: 100% 1px;
background-position: 0 1.15em;
```

新增：
* text-decoration-color 用于自定义下划线或其他装饰效果的颜色。
* text-decoration-style 用于定义装饰效果的风格（比如实线、虚线、波浪线等）。
* text-decoration-skip 用于指定是否避让空格、字母降部或其他对象。
* text-underline-position 用于微调下划线的具体摆放位置。

## 27. 现实中的文字效果
> 背景知识：基本的 text-shadow

### 凸版印刷效果
```css
background: hsl(210, 13%, 40%);
color: hsl(210, 13%, 75%);
text-shadow: 0 -1px 1px black;
```

### 空心字效果
```css
background: deeppink;
color: white;
text-shadow: 1px 1px black, -1px -1px black,
 1px -1px black, -1px 1px black;
```

### 文字外发光效果
```css
background: #203;
color: #ffc;
text-shadow: 0 0 .1em, 0 0 .3em;
```

### 文字凸起效果
```scss
@mixin text-3d($color: white, $depth: 5) {
 $shadows: ();
 $shadow-color: $color;
 @for $i from 1 through $depth {
 $shadow-color: darken($shadow-color, 10%);
 $shadows: append($shadows,
 0 ($i * 1px) $shadow-color, comma);
 }
 color: $color;
 text-shadow: append($shadows,
 0 ($depth * 1px) 10px black, comma);
}
h1 { @include text-3d(#eee, 4); }
```

复古风格的排印效果

```scss
@function text-retro($color: black, $depth: 8) {
 $shadows: (1px 1px $color,);
 @for $i from 2 through $depth {
 $shadows: append($shadows,
 ($i*1px) ($i*1px) $color, comma);
  }
 @return $shadows;
}
h1 {
 color: white;
 background: hsl(0,50%,45%);
 text-shadow: text-retro();
}
```

## 28. 环形文字
> 背景知识：基本的 SVG

### 解决方案
在 SVG 中，让文本按照路径排列的基本方法就是用一个 <textPath>元 素 来 包 裹 住 这 段 文 本， 再 把 它 们 装 进 一 个 <text> 元 素 中。 这 个<textPath> 元素还需要在它的 ID 属性中引用一个 <path> 元素，然后就可以用这个 <path> 元素来定义我们想要的路径。

```js
$$('.circular').forEach(function(el) {
 var NS = "http://www.w3.org/2000/svg";
 var xlinkNS = "http://www.w3.org/1999/xlink";
 var svg = document.createElementNS(NS, "svg");
 var circle = document.createElementNS(NS, "path");
 var text = document.createElementNS(NS, "text");
 var textPath = document.createElementNS(NS, "textPath");
 svg.setAttribute("viewBox", "0 0 100 100");
 circle.setAttribute("d", "M0,50 a50,50 0 1,1 0,1z");
 circle.setAttribute("id", "circle");
 textPath.textContent = el.textContent;
 textPath.setAttributeNS(xlinkNS, "xlink:href", "#circle");
 text.appendChild(textPath);
 svg.appendChild(circle);
 svg.appendChild(text);
 el.textContent = '';
 el.appendChild(svg);
});
```

# 用户体验

## 29. 选用合适的鼠标光标

### 难题
通过 `cursor` 属性来指定光标类型，比如 pointer 光标可以提示某个元素是可点击的，而 help 光标用来暗示这里有提示信息。

### 解决方案
![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/csssercets/cursor.png)

1. 提示禁用状态
```css
:disabled, [disabled], [aria-disabled="true"] {
 cursor: not-allowed;
}
```

2. 隐藏鼠标光标
```css
video {
 cursor: url(transparent.gif);
 cursor: none;
}
```

## 30. 扩大可点击区域

### 难题
其可点击区域（热区）向外扩张往往也可以带来可用性的提升。没有人愿意对一个狭小的按钮尝试点按很多次。

### 解决方案
首先`cursor: pointer`，然后伪元素。
```css
button {
 position: relative;
  /* [其余样式] */
}
button::before {
 content: '';
 position: absolute;
 top: -10px; right: -10px;
 bottom: -10px; left: -10px;
}
```

## 31. 自定义复选框
### 解决方案
当 <label> 元素与复选框关联之后，也可以起到触发开关的作用。

由于 label 不是复选框那样的替换元素，我们可以为它添加生成性内容（伪元素），并基于复选框的状态来为其设置样式。然后，就可以把真正的复选框隐藏起来（但不能把它从 tab 键切换焦点的队列中完全删除），再把生成性内容美化一番，用来顶替原来的复选框！

试一试
```html
<input type="checkbox" id="awesome" />
<label for="awesome">Awesome!</label>
```
```css
input[type="checkbox"] + label::before {
 content: '\a0'; /* 不换行空格 */
 display: inline-block;
 vertical-align: .2em;
 width: .8em;
 height: .8em;
 margin-right: .2em;
 border-radius: .2em;
 background: silver;
 text-indent: .15em;
 line-height: .65;
}
```
```css
input[type="checkbox"]:checked + label::before {
 content: '\2713';
 background: yellowgreen;
}
/* 把原来的复选框以一种不损失可
访问性的方式隐藏起来  */
input[type="checkbox"] {
 position: absolute;
 clip: rect(0,0,0,0);
}
/* 聚焦或禁用时改变它的样式 */
input[type="checkbox"]:focus + label::before {
 box-shadow: 0 0 .1em .1em #58a;
}
input[type="checkbox"]:disabled + label::before {
 background: gray;
 box-shadow: none;
 color: #555;
}
```
![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/csssercets/labelinput.png)

### 开关式按钮
其实只需要把 label 设置为按钮的样式即可，并不需要用到伪元素。
```css
input[type="checkbox"] {
 position: absolute;
 clip: rect(0,0,0,0);
}
input[type="checkbox"] + label {
 display: inline-block;
 padding: .3em .5em;
 background: #ccc;
 background-image: linear-gradient(#ddd, #bbb);
 border: 1px solid rgba(0,0,0,.2);
 border-radius: .3em;
 box-shadow: 0 1px white inset;
 text-align: center;
 text-shadow: 0 1px 1px white;
}
input[type="checkbox"]:checked + label,
input[type="checkbox"]:active + label {
 box-shadow: .05em .1em .2em rgba(0,0,0,.6) inset;
 border-color: rgba(0,0,0,.3);
 background: #bbb;
}
```

## 32. 通过阴影来弱化背景
> 背景知识：RGBA 颜色

设置一个足够大的`box-shadow`
```css
box-shadow: 0 0 0 50vmax rgba(0,0,0,.8);
```
然而没啥用，不能js交互。

作者推荐有限度地应用这个技巧，比如配合固定定位来使用，或者当页面没有滚动条时再用。

### 通过模糊来弱化背景
mask添加`filter`

## 34. 滚动提示
> 背景知识：CSS 渐变，background-size

给个网址：

https://www.w3cplus.com/css3/css-secrets/scrolling-hints.html


## 35. 交互式的图片对比控件
https://www.w3cplus.com/css3/css-secrets/interactive-image-comparison.html

# 结构与布局
## 36. 自适应内部元素
`min-content`
这个关键字将解析为这个容器内部最大的不可断行元素的宽度（即最宽的单词、图片或具有固定宽度的盒元素
```css
figure {
 max-width: 300px;
 max-width: min-content;
 margin: auto;
}
figure > img { max-width: inherit; }
```

## 37. 精确控制表格列宽
只需：
```css
table {
 table-layout: fixed;
 width: 100%;
}
```

## 38. 根据兄弟元素的数量来设置样式
### 难题
在某些场景下，我们需要根据兄弟元素的总数来为它们设置样式。最常见的场景就是，当一个列表不断延长时，通过隐藏控件或压缩控件等方式来节省屏幕空间，以此提升用户体验。
### 解决方法
利用两次伪类选择。
```scss
/* 定义mixin */
@mixin n-items($n) {
 &:first-child:nth-last-child(#{$n}),
 &:first-child:nth-last-child(#{$n}) ~ & {
 @content;
 }
}
/* 调用时是这样的： */
li {
 @include n-items(4) {
 /* 属性与值写在这里 */
 }
}
```

### 根据兄弟元素的数量范围来匹配元素
变量方式。
```css
li:first-child:nth-last-child(n+4),
li:first-child:nth-last-child(n+4) ~ li {
 /* 当列表至少包含四项时，命中所有列表项 */
}
```
假设我们希望在列表包含 2 ～ 6 个列表项时命中所有的列表项，可以
这样写：
```css
li:first-child:nth-last-child(n+2):nth-last-child(-n+6),
li:first-child:nth-last-child(n+2):nth-last-child(-n+6) ~ li {
 /* 当列表包含2～6项时，命中所有列表项 */
}
```

## 39. 满幅的背景，定宽的内容
### 难题
在过去的几年间，有一种网页设计手法逐渐流行起来，我将它称作背景宽度满幅，内容宽度固定

### 解决方法
利用算术表达式
```css
.wrapper {
 max-width: 900px;
 margin: 1em calc(50% - 450px);
}
```

如果我们将长度值应用到父元素的padding上。
```css
footer {
 max-width: 900px;
 padding: 1em calc(50% - 450px);
 background: #333;
}
.wrapper {}

/* 回退机制 */
footer {
 padding: 1em;
 padding: 1em calc(50% - 450px);
 background: #333;
}
```

## 40. 垂直居中
### 基于绝对定位的解决方案
```css
main {
 position: absolute;
 top: 50%;
 left: 50%;
 transform: translate(-50%, -50%);
}
```

### 基于视口单位的解决方案
```css
main {
 width: 18em;
 padding: 1em 1.5em;
 margin: 50vh auto 0;
 transform: translateY(-50%);
}
```

### 基于 Flexbox 的解决方案
```css
main {
 display: flex;
 align-items: center;
 justify-content: center;
 width: 18em;
 height: 10em;
}
```

## 41. 紧贴底部的页脚
> 背景知识：视口相关的长度单位（参见“垂直居中”），calc()

### calc() 函数
我们可以把 <header> 和 <main> 元素包在一个容器里，然后在算式中就只需要考虑页脚的高度了：
```css
#wrapper {
 min-height: calc(100vh - 7em);
}
```

### 更灵活的解决方案flex
```css
body {
 display: flex;
 flex-flow: column;
 min-height: 100vh;
}
main { flex: 1; }
```
这 个 `flex` 属性实际上是`flex-grow`、`flex-shrink` 和`flex-basis` 的简写语法。 任何元素只要设置了一个大于 0 的`flex` 值，就将获得可伸缩的特性；`flex` 的值负责控制多个可伸缩元素之间的尺寸分配比例。举例来说，在我们眼前的这个例子中， 如 果 `<main>` 是 flex: 2 而`<footer>` 是 flex: 1，那么内容区块的高度将是页脚高度的两倍。如果把它们的值从 2 和 1 改为 4 和 2，结果也是一样的，因为真正起作用的是它们的数值比例。



