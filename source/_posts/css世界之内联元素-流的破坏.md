---
title: css世界之内联元素&&流的破坏
date: 2018-10-01 13:16:11
categories: css世界
tags:
- css
---
这一部分总结了两处，一处是内联元素，其实内联元素这里主要让我重新认识了下vertical-align和line-height。另一处就是关于流的破坏，主要就是介绍了下float和absolute这些属性一些知识点了。最后国庆节快乐哈哈。
<!-- more -->
# 内联元素与流

在内联盒子中，涉及到垂直方向的排版或者对齐，都离不开基线。而字母x的下边缘就是我们的基线（baseline）。

`vertical-align: middle`对齐为字母x的中心，因为各种字体不同，所以不是绝对的垂直居中。内联元素垂直居中是对文字，而非外部的块级元素。

ex是CSS一个相对单位，指的是小写字母x的高度，就是指**x-height**。

## 内联元素的高度 line-height

对于非替换元素的纯内联元素，其可视高度完全由line-height决定。

由于上下半间距的存在，与设计师的文字上下边距的计算一般不同，需要我们重新计算下，文字一般偏下，所以文字下边距向上取整，文字上边距向上取整。在有内联替换元素的时候，line-height不能影响替换元素的高度，内联替换元素和内联元素混排的的时候。line-height只能决定行高的最小高度。

### line-height让内联元素“垂直居中”

要让单行文字垂直居中，其实不用设置height，只需要设置好line-height的高度就好了。

多行文字居中的话也有办法，嵌套的div，使用inline-block来表示。

```css
.box {
  line-height: 120px;
  background: #f0f3f9;
}

.content {
  display: inline-block;
  line-height: 20px;
  margin: 0 20px;
  vertical-align: middle;
}
```

为什么上面的代码能垂直居中呢，全凭内联盒子前面那个宽度为0的“幽灵空白节点”。

## line-height的属性值
`line-height: normal`是默认值。不同字体下的`line-height`的属性值normal计算量都是不一样的：

|字体|Chrome|Firefox|IE|
|---|------|-------|---|
|微软雅黑|1.32|1.321|1.32|
|宋体|1.141|1.142|1.141|

line-height有三种计算方式：
1. 数值。如`line-height: 1.5`，与当前font-size相乘后的值。

2. 百分比值。如`line-height: 150%`，与当前font-size相乘后的值。

3. 长度值。如`line-height:21px`或者`者 line-height:1.5em`。

使用数值的计算，和其他两种方式在继承方面有所不同。使用数值的话，那么所有的子元素继承的都是这个值；使用百分比值或者长度值作为属性值，所有
的子元素继承的是最终的计算值。

这里推荐的继承设置：

```css
body {
  line-height: 1.5;
}

input, button {
  line-height: inherit;
}
```

## 内联元素line-height的“大值特性”

```css
.box {
  line-height: 96px;
}

.box span {
  line-height: 20px;
}

/* 另一个 */
.box {
  line-height: 20px;
}

.box span {
  line-height: 96px;
}
```

一个子元素的行高是20px，一个是96px，假如一行文字，.box元素的高度是多少。答案是都是96px。

原因是，里层96px直接撑开，外层96px，“幽灵空白节点”继承96px，最终撑开。设置独立的`inline-block`避免“幽灵空白节点”影响。

## vertical-align

首先要知道，line-height的高度并不是元素的最终高度。

vertical-align分为下面四大类：

* 线类，如baseline、top、middle、bottom。

* 文本类，如text-top、text-bottom。

* 上标下标类，如sub、super。

* 数值百分比。如20px、2em、20%等。

默认是baseline，如果设置10px这样，就会往baseline往上偏移10px。负值设置则是往下偏移。

### vertical-align作用的前提

verticl-align作用的前提是，只能应用于内联元素以及display值为table-cell的元素。

这里举一个图片的vertical-align没有效果的例子。

```css
/* 无效果 */
.box {
  height: 128px;
}

.box > img {
  height: 96px;
  vertical-align: middle;
}

/* 设置line-height */

.box {
  height: 128px;
  line-height: 128px;
}

.box > img {
  height: 96px;
  vertical-align: middle;
}
```

### vertical-align和line-height
阐述一下容器高度不等于行高的例子

```css
.box {
  line-height: 32px;
}

.box > span {
  font-size: 24px;
}
```

因为“幽灵空白节点”的字体大小不等于span，所以字号不一样的参差位移导致的。如果在父元素设置好字体大小和子元素一样就不会有这样的情况啦。

图片底部存在间隙也是“幽灵空白节点”、line-height和vertical-align共同导致的。解决方法：

1. 图片块状化。

2. 容器line-height足够小。

3. 容器font-size足够小。

4. 图片设置其他vertical-align属性值。

关于图片的`margin-top: -200`无效这样的效果，原因是“幽灵空白节点”因为默认的`vertical-align: baseline`固定死在父级容器内。

### vertical-align线性类属性值

#### inline-block与baseline

vertical-align 属性的默认值 baseline 在文本之类的内联元素那里就是字符 x 的下边缘，对于替换元素则是替换元素的下边缘。但是，如果是 inline-block 元素，则规则要复杂了：一个 inline-block 元素，如果里面没有内联元素，或者 overflow 不是 visible，则该元素的基线就是其 margin 底边缘；否则其基线就是元素里面最后一行内联元素的基线。

inline-block和baseline的利用，删除icon，`<i class="icon-delete">删除</i>`和`<i class="icon-delete"></i>`这里面一个有文字，一个没文字,由于有文字的一般设置为`overflow：hidden`但是他们的元素基线都是margin底边缘。下面是小技巧，将inline-block和文字对齐。

1. 图标高度和行高一样。一般设定一个固定的宽高。

2. 图标标签里面永远有字符。可以借助::before或者::after伪元素生成一个空格字符串。

3. 图标CSS不使用`overflow: hidden`保证基线为里面字符的基线，但是要让里面潜在的字符不可见。

# 流的破坏与保护

## float
float的特性：
* 包裹性。
* 块状化并格式化上下文。
* 破坏文档流。
* 没有任何margin合并。

元素设置float，会让父元素坍塌。使用浮动元素的时候，最好采用采用一些手段干净地清除浮动带来的影响。

### float的克星clear
语法如下：

clear: none | left | right | both

clear的官方解释是“元素盒子的边不能和前面的浮动元素相邻”。

* none: 默认值，左右浮动都有。
* left: 左侧康浮动。
* right: 右侧抗浮动。
* both: 两侧抗浮动。

一般使用，`clear: both`。`clear: both`的本质是让自己不和float元素在一行显示，并不是真正意义上的清除浮动。

1. 如果`clear: both`元素前面的元素就是float元素，则`margin-top`负值即使设置-9999px也没有效果。

2. `clear: both` 后面的元素一九可能发生文字环绕的现象。

## BFC

BFC全称**block formatting context**，中文是“块级格式化上下文”。与之对应的还有IFC，“内联格式化上下文”。

触发BFC的条件：

* `<html>`根元素。
* float的值不为none。
* overflow的值auto、scroll或hidden。
* display的值`table-cell`、`table-caption`和`inline-block`中任何一个。
* position的值不为relative和static。

BFC特点是及最重要用途是，实现更健壮、更智能的自适应布局。而不仅仅是去margin和清除float影响。

BFC的表现规则是，具有BFC特性的元素的子元素不会受外部元素影响，也不会影响外部元素。

## overflow
最适合产生BFC的属性就是`overflow: hidden`。当子元素内容超出容器高度限制的时候，剪裁的边界是`border box`的内边缘，而非`padding box`的内边缘。

介绍下overflow属性经典的不兼容问题。在chrome浏览器下，如果浏览器能滚动（假设是垂直滚动），则`padding-bottom`也算在滚动尺寸内，IE和Firefox浏览器忽略`padding-bottom`。

### overflow-x和overflow-y

* visible：默认值。
* hidden：剪裁。
* scroll：滚动条区域一直在。
* auto：不足以滚动时没有滚动条，可以滚动时滚动条出现。

关于overflow-x和overflow-y的设置，如果一个值设置visible而另一个值设置为scroll、auto和hidden，则visible会当成auto来解析。但是scroll、auto和hidden可以共存。

PC端的默认滚动条来自`<html>`，而不是`<body>`标签，移动端就不是这样啦。

PC端滚动条会占用容器的可用宽度和高度。移动端滚动条一般是悬浮的就不会。

### overflow与锚点定位
下面两种情况可以触发锚点定位:

### 锚点定位行为触发条件

1. URL地址中的锚链与锚点元素对应并有交互行为。

2. 可focus的锚点元素处于focus状态。

### 锚点定位作用的本质

锚点定位作用的发生，本质上是通过改变容器滚动高度或者宽度来实现的。

锚点定位可以发生在普通元素，而且定位行为的发生是由内而外的。“由内而外”指的是，普通元素和窗体同时可滚动的时候，会由内而外触发所有可滚动窗体的锚点定位行为。

设置`overflow: hidden`的元素也是可以滚动的，只是没有滚动条，如内部的scrollTop依旧可以使用。

## position: absolute
当一个元素同时拥有absolute和float的时候，float属性没有任何效果。具有块状话，一旦设置了absolute或者fixed，其元素display就变成block。

关于元素的宽度的详细计算规则：

1. 根元素（很对场景下可以看成`<html>`）被称为“初始包含块”，其尺寸等于浏览器可视窗口的大小。

2. 对于其它元素，如果该元素的position是relative或者static，则“包含块”由其最近的块容器祖先盒的content box边界形成。

3. 如果元素position: fixed，则包含块是“初始包含块”。

4. 如果元素的position: absolute，则“包含块”由最近的position不为static的祖先元素建立。

    如果该祖先元素是纯inline元素，则规则略复杂：

    * 假设给内联元素的前后各生成一个宽度为0的内联盒子（inline box），则这两个内联盒子的padding box外面的包围盒就是内联元素的“包围块”。
    * 如果该内联元素被跨行分割了，那么“包含块”是未定义的，也就是CSS2.1规范没有明确定义，浏览器自行发挥。

与常规元素相比，absolute绝对定位元素的“包含块”有以下3个差异
1. 内联元素也可以作为“包含块”所在的元素。
2. “包含块”所在的元素不是父级块元素，而是最近position不为static的祖先元素或者根元素。
3. 边界是padding box而不是content box。

内联元素一般不做“包含块”。
1. 一般使用absolute绝对定位都适合布局有关，而内联元素主要是图文展示。

2. 理解和学习成本高。内联元素的“包含块”不能按照常规块级元素的“包含块”来理解。

3. 兼容性问题。无论内联元素是单行还是跨行都存在兼容性问题。单行的兼容性问题存在于“包含块”是一个空的内联元素的时候。

提示信息一行使用absolute定位的时候，利用`white-space: nowrap`让信息单行显示。

### 具有相对特性的无依赖absolute绝对定位

absolute是非常独立的CSS属性，其样式和行为表现不依赖其他任何CSS属性就可以完成。

absolute定位效果完全不需要父元素设置position为relative或者其他属性就能实现。这种情况下，子元素也不要设置left/top/right/bottom的属性值。其实这种“无依赖绝对定位”本质上就是“相对定位”，仅仅是不占据CSS流的尺寸空间而已。

简单的例子

1. 图标的的定位

2. 超越常规布局的排版

  ![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/little_tricks/absoluteLayout.png)

3. 下拉列表的定位

  ![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/little_tricks/absoluteSelect.png)

```css
.datalist {
  position: absolute;
}

.search-result {
  display: none;
}
input:focus ~ .search-result {
  display: block;
}
```

虽然“无依赖绝对定位”好处多多，但建议只用在静态交互效果上，比方说，导航二级菜单的显示与定位。如果是动态呈现的列表，建议还是使用 JavaScript 来计算和定位。

### absolute与overflow

overflow与absolute元素的裁剪规则用一句话表述就是：绝对定位元素不总是被父级 overflow 属性剪裁，尤其当 overflow 在绝对定位元素及其包含块之间的时候。

转述就是：如果overflow不是定位元素，同时绝对定位元素和overflow容器之间也没有定位元素，则overflow无法对absolute元素经行剪裁。

```html
<!-- 不会剪裁 -->
<div style="overflow: hidden;">
  <img src="1.jpg" style="position: absolute;">
</div>
<div style="position: relative;">
  <div style="overflow: hidden;">
    <img src="1.jpg" style="position: absolute;">
  </div>
</div>

<!-- 会剪裁 -->
<div style="overflow: hidden; position: relative;">
  <img src="1.jpg" style="position: absolute;">
</div>
<div style="overflow: hidden;">
  <div style="position: relative;">
    <img src="1.jpg" style="position: absolute;"> <!-- 剪裁 -->
  </div>
</div>
```

当元素出现transform属性的时候，近似于添加了“定位元素”。所以需要检查，absolute元素是否会被剪裁，fixed定位失效。

### absolute与clip

CSS 世界中有些属性或者特性必须和其他属性一起使用才有效，比如clip。

`clip: rect(top, right, bottom, left)`表示据每个边缘的距离开始剪裁。作用有：

1. fixed固定定位的剪裁。

2. 最佳可访问性隐藏。例如隐藏一些文字之类的。


clip 隐藏仅仅是决定了哪部分是可见的，非可见部分无法响应点击事件等；然后，虽然视觉上隐藏，但是元素的尺寸依然是原本的尺寸，在 IE 浏览器和 Firefox 浏览器下抹掉了不可见区域尺寸对布局的影响，Chrome 浏览器却保留了。

### absolute流体特性

绝对定位元素在“对立方向同时发生定位的时候”。

当绝对定位元素处于流体状态的时候，各个盒模型相关属性的解析和普通流体元素都是一模一样的，margin 负值可以让元素的尺寸更大，并且可以使用 margin:auto 让绝对定位元素保持居中。

```css
.element {
  width: 300px; height: 200px;
  position: absolute;
  left: 0; right: 0; top: 0; bottom: 0;
  margin: auto;
}
```

## position: relative
relative的定位有两点值得一提：
* 相对定位元素的left/top/right/bottom的百分比计算值是相对包含块计算的，而不是本身。所以如果包含块的高度是auto，那么计算值是0，便宜无效。
* 相对元素同时应用对立方向定位值的时候，按照默认文档流，自上而下、从左往右，top
、left作用大。

### relative最小化影响原则

1. 尽量不适用relative，想定位某些元素，首先看能否使用“无依赖的绝对定位”。
2. 如果场景受限，一定要使用relative，则该relative务必最小化。

原因有，一个普通元素变成相对定位元素，元素的层叠顺序提高了。“relative 的最小化影响原则”不仅规避了复杂场景可能出现样式问题的隐患，从日后的维护角度讲也更方便。

## position: fixed固定定位
position:fixed 固定定位元素的“包含块”是根元素，我们可以将其近似看成`<html>`元素。
