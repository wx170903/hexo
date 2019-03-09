---
title: css揭秘总结(－)
date: 2017-12-31 13:20:41
categories: CSS
tags: CSS
---
# 引言
这个总结，基本主要记录作者给的解决方法和主要的code。思路的确很重要。共有47个小难题。
<!-- more -->

## CSS3的诸多模块
* CSS2(原有模块)=>CSS3
  * CSS语法(http://w3.org/TR/css-syntax-3)
  * CSS层叠与继承(http://w3.org/TR/css-cascade-3)
  * CSS颜色(http://w3.org/TR/css3-color)
  * 选择符(http://w3.org/TR/selectors)
  * CSS背景与边框(http://w3.org/TR/css3-background)
  * CSS值与单位(http://w3.org/TR/css-values-3)
  * CSS文本排版(http://w3.org/TR/css-text-3)
  * CSS文本装饰效果(http://w3.org/TR/css-text-decor-3)
  * CSS字体(http://w3.org/TR/css3-fonts)
  * CSS基本UI特性(http://w3.org/TR/css3-ui)
* CSS3新增
  * CSS变形(http://w3.org/TR/css-transforms-1)
  * 图像混合效果(http://w3.org/TR/compositing-1)
  * 滤镜效果(http://w3.org/TR/filter-effects-1)
  * CSS遮罩(http://w3.org/TR/css-masking-1)
  * CSS伸缩盒布局(http://w3.org/TR/css-flexbox-1)
  * CSS网格布局(http://w3.org/TR/css-grid-1)

## 浏览器前缀的由来
刚开始学习css的时候，只是看教学的说需要添加前缀，可是前缀的由来记得不是很清楚。其实，在一条规范正式添加到规范中，会有一些实验性的特性。网页开发者可以根据浏览器给的前缀自由的添加相关特性，并将试用结果返还给工作组。结果开发者发现，这些有了前缀的特性能轻易的实现以前需要大费周折所实现的效果，所以，这些特性就被滥用。后来发现只写出当前适用的新特性时，还需要时不时回来打补丁，后来干脆将所有可能的浏览器前缀添加上去变成：
```css
-moz-border-radius: 10px;
-ms-border-radius: 10px;
-o-border-radius: 10px;
-webkit-border-radius: 10px;
border-radius: 10px;
```
后来出现的预处理器成功解决了这些问题，不用写这么丑的代码了。

## CSS编码技巧
### 尽量减少代码重复
在实践中，代码可维护性的最大要素是尽量减少改动时要编辑的地方。最简单的例如字体，用`em`表示，`line-height`与字体相关等。所以我们需要审视到底哪些效果是需要动态的变化的，哪一些又是固定不变的。

1. 代码易维护 vs 代码量少

有时候，代码易维护和代码量少不可兼得。如下：我们为一个元素添加一道10px宽的边框，但左侧不加边框。
```css
/* 方法一 */
border-width: 10px 10px 10px 0; 
/* 方法二 */
border-width: 10px;
border-left-width: 0;
```
方法一的代码只需要一条声明就能完成，可日后我们需要改动变宽的宽度的时候需要同时改动三个地方。如果如方法二，改动起来就容易多了。

2. currentColor

这个值是CSS中有史以来的第一个变量，很多属性，例如`border-color`、`outline-color`等的初始值就是这个，它自动从文本获得颜色。

3. 继承`inherit`

大多数人知道`inherit`这个关键字，但很容易遗忘。`inherit`可以用在任何CSS属性中，而且他还总是绑定到父元素的计算值，对于伪元素来说则会取得生成该伪元素的宿主元素。

### 相信你的眼睛，而不是数字
关于，垂直居中稍微往上一点看起来更居中。内边距顶部和底部的内边距需要稍微比左右的小一点，这样看起来更整齐等。主要是视觉上的错觉。

### 关于响应式网页设计
1. 媒体查询的端点不应该由具体的设备来决定。
2. 使用百分比长度来取代固定长度。或于是口相关的单位（`vw`、`vh`、`vmin`和`vmax`）。
3. 需要在较大设备获得固定宽度，使用`max-width`而不是`width`,前者可以使用较小的分辨率。
4. 为替换元素(img、objuct、video、iframe等)设置一个max-width。
5. 背景图片`background-size: cover`可以铺满，但大图缩小带宽会消耗。
6. 图片或其他元素行列式布局时，`display: inline-block`或`flexbox`来布局。
7. 多列文本，指定`column-width`（列宽）而不是指定`column-count`（列数）。

### 合理使用简写
```css
background: rebeccapurple;
background-color: rebeccapurple
```
前者简写可以确保，你获得rebeccapurple的纯色背景，而后者，可能是一个图案，因为可能有一条`background-image`在起作用。你也许能将所有的展开式属性全部设置一遍，但你可能会漏几个；又或者，未来CSS工作组会引入更多的展开式属性。所以合理的使用简写是一种良好的防卫性编码方式，可以抵御未来的风险。如果我们需要明确的覆盖某个具体的展开式属性，就需要使用展开式属性，如前面的`border-width`。

怪异的简写语法 => 可能会产生歧义的属性值用`'/'`分隔。

# 背景与边框

## １．半透明边框
> 背景知识: RGBA/HSLA颜色

### 难题
也许我们会写出这样的代码来实现半透明边框：
```css
border: 10px solid hsla(0,0%,100%,.5);
background: white;
```
然而半透明边框却没有实现半透明边框？

### 解决方案
其实我们的边框是存在的，然而我们所需要做的是，将body的背景从半透明的白色边框透出来。我们可以通过`background-clip`来调整上述的特性，这个属性的初始值是`border-box`,意味着背景会被元素的`border-box`（即边框的外沿框）裁切掉。我们修改为`padding-box`就可以用内边距的外沿来把背景裁切掉。
```css
border: 10px solid hsla(0,0%,100%,.5);
background: white;
bcakground-clip: padding-box;
```

## 2. 多重边框
> 背景知识：box-shadow的基本用法

### 难题
有时我们需要建立多个边框。

### `box-shadow` 方案
我们需要了解`box-shadow`的第四个参数，称为"扩张半径",可以让投影面积变大或者变小。

优点：它的支持逗号分隔，我们可以创建任意数量的投影。适用于多层边框需求

```css
background: yellowgreen;
box-shadow: 0 0 0 10px #655, 0 0 0 15px deeppink;
```

`box-shadow`是层层叠加的，第一层投影位于最顶端，以此类推。

缺点：
1. 投影的行为和边框不一致。
2. 假投影出现在元素外圈，不会影响相关事件。

### `outline` 方案
当我们只需要两层边框的时候，可以在设置常规的边框后添加`outline`描边。优点是`outline`十分灵活，而`box-shadow`只能模拟实现边框。
```css
background: yellowgreen;
border: 10px solid #655;
outline: 5px solid deeppink;
```
缺点：
1. 只适用于双层边框。
2. 边框不一定贴合`border-radius`产生的圆角。
3. 尽管说描边可以不是矩形，但大多数情况需要测试。

## 3. 灵活的背景定位
### 难题
很多时候，我们想针对容器的某个角对背景图案做偏移定位,例如想要类似内边距的效果怎么做呢。

### `background-position`的扩展语法方案
```css
background: url(...) no-repeat #58a;
background-position: right 20px bottom 10px;

/* 退回方案 */
background: url(...) no-repeat bottom right #58a;
background-position: right 20px bottom 10px;
```

### background-origin 方案
需要图片的偏移量和内边距相同时。
```css
padding: 10px;
background: url(...) no-repeat #58a;
background-position: right 10px bottom 10px;
```
代码很繁琐，一个改动需要改三处。

默认情况下`background-position`是以`padding box` 为准的，即`left` `top`等是以`padding box`为准的。每个元素有三个矩形框，`boeder box`, `padding box`和`content box`。
```css
padding: 10px;
background: url(...) no-repeat #58a bottom right; 
background-origin: content-box;
```

### `calc` 方案
```css
background: url(...) no-repeat;
background-position: calc(100% - 20px) calc(100% - 10px)
```

## 4. 边框内圆角
> 背景知识：box-shadow, outline "多重边框"

### 难题
有时我们需要一个容器，只在内侧有圆角，而边框或描边的四个角在外部依然保持矩形的形状。这里放弃两个`div`元素的方案。

### 解决方案
两个`div`的确很优秀，如果我们只是需要纯色的边框，只需要一个元素的方法呢。
```css
background: tan;
border-radius: .8em;
padding: 1em;
box-shadow: 0 0 0 .6em #655
outline: .6em solid #655
```
主要就是利用`box-shadow`的扩展半径填充元素与`outline`之间的缝隙。
根据勾股定理大约取值为半径的一半。

## 5. 条纹背景
> 背景知识：CSS线性渐变， background-size

### 难题
简单的条纹背景，我们用CSS自己创建。

### 解决方案
1. 水平条纹
```css
background: linear-gradient(#fb3 33.3%, #58a 0, #58a 66.6%, yellowgreen 0);
background-size: 100% 45px
```
避免每次修改条纹宽度的捷径：
> 如果某个色标的位置值比整个列表中在它之前的色标的位置值都要小，则该色标的位置值会被设置为它前面所有色标位置值的最大值。

2. 垂直条纹
```css
linear-gradient(to right, /* 或 90deg*/ #fb3 50%, #58a 0);
background-size: 30px 100%;
```

3. 斜向条纹

由于贴片问题，我们需要设置四个色块。贴片的面积也需要重新计算相当于1.4倍的需要的半径。
```css
background: linear-gradient(45deg, #fb3 25%, #58a 0, #58a 50%, #fb3 0, #fb3 75%, #58a 0);
background: 42px 42px;                        
```

4. 更好的斜向条纹

`linear-gradient()` 和 `radial-gradient()`还有循环加强版：`repeating-linear-gradient()` 和 `repeating-radial-gradient()`

```css
background: repeating-linear-gradient(60deg, #fb3, #fb3 15px, #58a 0, #58a 30px);
```

5. 灵活的同色系条纹
```
background: #58a;
background-image: repeating-linear-gradient(30deg, hsla(0,0%,100%,.1), hsla(0,0%,100%,.1) 15px, transparent 0, transparent 30px)
```
利用transparent实现条纹的深色，直接显示背景。

## 6. 复杂的背景图案
> 背景知识：CSS渐变，“条纹背景”

### 难题
不仅仅有各种条纹图案，一些稍复杂的几何图案，CSS也能实现。

### 网格
```css
background: white;
background-image: linear-gradient(90deg, rgba(200,0,0,05) 50%, transparent 0),linear-gradient(rgba(200,0,0,05) 50%, transparent 0);
background-size: 30px 30px;
/* 长度单位线条 */
background: #58a;
background-image: 
  linear-gradient(white 1px, transparent 0),
  linear-gradient(90deg, white 1px, transparent 0);
background-size: 30px 30px;
/* 上面会形成网格图案，网格的长度还可以用长度来表示，线条的宽度可以控制 */
```

### 波点
直接放上波点的`mixin`, 波点需要两个贴片重叠。
```scss
@mixin polka($size, $dot, $base, $accent) {
  background: $base;
  background-image: 
    radial-gradient($accent $dot, transparent 0),
    radial-gradient($accent $dot, transparent 0);
  background-size: $size $size;
  background-position: 0 0, $size/2 $size/2;
}
// 调用例子
@include polka(30px, 30%, #655, tan)
```

### 棋盘
这里我们也只放上`mixin`, 它也需要贴片的重叠。
```scss
@mixin checkerboard($size, $base, $accent: rgba(0,0,0,.25)) {
  background: $base;
  background-image: 
    linear-gradient(45deg, 
      $accent 25%, transparent 0,
      transparent 75%, $accent 0)
    linear-gradient(45deg,
      $accent 25%, transparent 0,
      transparent 75%, $accent 0)
    background-position: 0 0, $size $size,
    background-size: 2*$size 2*$size;
}
// 调用
@include checkerboard(15px, #58a, tan);
```

## 7. 伪随机背景
> 背景知识：CSS渐变，“条纹背景”，“复杂的背景图案”

### 难题
其实自然界的事物都不是以无限平铺的方式存在的。CSS本身没有提供随机功能。

### 解决方案
通过质数来增加随机真实性，多个质数的话，那么只有经过这几个质数相乘才会重复一次。

## 8. 连续的图像边框
> 背景知识：CSS渐变，基本的`border-image`，“条纹背景”，基本的CSS动画

### 难题
有时我们想把一副图案或者图片应用为边框，而不是背景。

### 解决方案
也许两个元素，一个做背景一个做内容很容易做到，但我们CSS方案，尽量，每个`html`元素和内容不重叠, 主要思路：在石雕背景图案之上，再叠加一层纯色的实色背景。给两层背景指定不一样的`background-clip`值，实色用css渐变来实现。

方法一：
```css
/* background-origin 消除拼接效果。 */
padding: 1em;
border: 1em solid transparent;
background: linear-gradient(white, white),
            url(...);
background-size: cover;
background-clip: padding-box, border-box;
background-origin: border-box; 

/* 简写 */
padding: 1em;
border: 1em solid transparent;
background:
  linear-gradient(white, white) padding-box,
  url(...) border-box 0 / cover;

/* 信封边框图案 */
padding: 1em;
border: 16px solid transparent;
border-image: 16 repeating-linear-gradient(-45deg,
                    red 0, red 1em, 
                    transparent 0, transparent 2em,
                    #58a 0, #58a 3em,
                    transparent 0, transparent 4em);
```

如果利用动画还能形成蚂蚁行军效果。即选中不停转动的虚线框。
```css
@keyframes ants {to {background-position: 100%}}

.marching-ants {
  padding: 1em;
  border: 1px solid transparent;
  background: 
    linear-gradient(white, white) padding-box,
    repeating-linear-gradient(-45deg,
      black 0, black 25%, white 0, white 50%
      ) 0 / .6em .6em;
  animation: ants 12s linear infinite;
}
```