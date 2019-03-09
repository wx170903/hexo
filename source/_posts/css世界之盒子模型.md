---
title: css世界之盒子模型
date: 2018-09-27 23:59:15
categories: css世界
tags:
- css
---
这是这个总结系列的第二篇文章，关于content、padding、margin、border的。
<!-- more -->
# content

## 什么是替换元素

简单的来说，通过修改某个属性值，呈现的内容就可以被替换的元素就称之为**替换元素**。

1. 内容的外观不受页面的css影响。也就是说样式表现在css作用域之外，如何修改替换元素本身的外观呢。需要利用浏览器提供行的一些样式接口，如：`::-ms-check{}`，直接修改样式是不起作用的。

2. 有自己的尺寸。很多替换元素在没有明确尺寸的时候，其默认尺寸（不包括边框）为300像素×150像素。

3. 很多css属性有自己的一套表现方式。比如`vertical-align`属性，对于非替换内联元素来说，其一般是字符x的下边缘，而对于替换元素来说，一般来说是元素的下边缘。

### 替换元素的display值

替换元素都是内联元素，但是它们的display值却是有所不同。

|元素|Chrome|Firefox|IE|
|----|-----|--------|---|
|`<img>`|inline|inline|inline|
|`<iframe>`|inline|inline|inline|
|`<video>`|inline|inline|inline|
|`<select>`|inline-block|inline-block|inline-block|
|`<input>`|inline-block|inline|inline-block|
|`range(or)file <input>`|inline-block|inline-block|inline-block|
|`hidden <input>`|none|none|none|
|`<button>`|inline-block|inline-block|inline-block|
|`<video>`|inline-block|inline|inline-block|

### 替换元素的尺寸计算规则

1. 固有尺寸。这个固有尺寸指的是替换内容的原本尺寸。例如，图片、视频作为一个独立文件存在的时候，都是有着自己的宽度和高度的。这个宽度和高度的大小就是“固有尺寸”。

2. HTML尺寸。“HTML尺寸”只能通过HTML原生属性改变，这些原生属性包括`<img>`的width和height属性、`<input>`的size属性、`<textarea>`的cols和rows属性等。

3. CSS尺寸特指可以通过CSS的width和height。

css尺寸 > HTML尺寸 > 固有尺寸。即使替换元素设置`display: block`，尺寸规则仍然和内联状态一样，这也是为什么图片或者其他表单元素设置`display:block`却没有达到100%容器的原因。

如果一个图片没有src，那么其将不会有网络请求，这一点可以用在图片懒加载上面。而在Firefox中，这种没有src的img元素将会表现为一个普通的内联元素，所以宽高设置无效，所以需要将其设置为`display: inline-block`。

关于图片的width和height设置，影响图片的方式。图片的默认填充方式是fill，也就是说，你说设置的宽度和高度都会被默认填充满，所以尺寸的变化，改变的不是图片的固有尺寸，只是改变了图片填充的外部尺寸。CSS3中，替换元素的适配方式，可以通过`object-fit`来修改。

### 替换元素和非替换元素的区别

#### 观点1： 替换元素和非替换元素之间只隔了一个src属性。

这里举了一个小例子，利用到了伪元素的图片生成技术。

  * 不能有src属性（证明观点所在）

  * 不能使用content属性生成图片（针对Chrome）

  * 需要有alt属性并有值（针对Chrome）

  * Firefox下::before伪元素的content值会被无视，::after不会发生这种问题。

这里有一个展示的链接，http://demo.cssworld.cn/4/1-2.php 。这里可以演示图片没加载成功的时候，hover提示图片信息的技术。

#### 观点2：替换元素和非替换元素之间只隔了一个CSS content 属性

替换元素之可替换的部分是content box，对应css属性是content。所以从理论来说content熟悉决定了是替换元素或者非替换元素。直接添加了content属性的元素我们可以称之为“匿名替换元素”。

在MDN中对于[content](https://developer.mozilla.org/zh-CN/docs/Web/CSS/content)的介绍中，也直接说明了使用content 属性插入的内容都是匿名的可替换元素。

关于content的利用，这里放一个[三个点动态加载](http://demo.cssworld.cn/4/1-9.php)以及[counter()](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Lists_and_Counters/Using_CSS_counters)。计数器可以说也主要是生成文档的时候来用一下。最后最后，content能够混合的，也就是像这种形式`content: "(" attr(href) ")"`。

# padding

## 奇怪的首选最小宽度

关于padding里面有几个点需要注意的，即使设置了`box-sizing: border-box`，当padding计算值大于width时，width也会无效，最终宽度呈现为padding计算值，内容则表现为“首选最小宽度”。而且加了文字也不会变，一般来说是0吧。

## 内联元素的padding

内联元素的padding，会发生层叠，就是说，对上下元素的原本布局没有任何影响，却发生了层叠。

css中有很多不影响其他元素布局的层叠现象。如，relative元素的定位、盒阴影box-shadow以及outline等。

1. 纯视觉层叠。不影响外部尺寸，如box-shadow以及outline等。

2. 会影响视觉的层叠。inline元素的padding就是这种。

区分的方式也很简单，如果父容器设置为`overflow: auto`，层叠区域超出父元素的时候，没有滚动条出现，则是纯视觉的；如果有滚动条出现，则会影响尺寸，影响布局。

## 内联padding的一点小应用

### 利用内联padding实现高度可控的“管道分隔符”

[查看链接](http://demo.cssworld.cn/4/2-2.php)

```css
a + a::before {
  content: '';
  font-size: 0;
  padding: 10px 3px 1px;
  margin-left: 6px;
  border-left: 1px solid gray;
}
```

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/little_tricks/pipe_border.png)

## padding的百分比值及应用

块级元素的话，有一个纯css实现宽高5:1的头图， [效果图](http://demo.cssworld.cn/4/2-3.php)

```css
.box {
  padding: 10% 50%;
  position: relative;
}

.box > img {
  position: absolute;
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
}
```

这里在讨论一下内联元素的padding百分比。

* 同样的相对宽度计算。
* 默认的高度和宽度细节有差异。
* padding会断行。

内联元素的padding首先计算的是左边，那么左边的长度够了的话，元素就会换行，换行之后，新的padding会形成背景，如果是背景色不是transparent的话，就会根据先来后到的产生层叠现象，而且层叠等级比较高。会产生相应的奇怪的现象，一般的开发过程中，遇到这种情况的概率比较小。

空白的内联元素的表现也会有不同的表现，因为内联元素前面有一个宽度为0高度自适应的“幽灵空白节点元素”，所以，设置`padding: 50%`最终的变现为一个长方形，这种现象，在我们知道了“幽灵空白节点元素”的存在就不会显得那么难以理解了。

## padding与图形绘制

padding属性和background-clip属性配合，可以在有限的标签下实现一些css图形绘制效果。

不使用伪元素，仅一层标签实现菜单图标。下面是十倍大小模拟。

双层圆心选中效果也可以直接模拟。

```css
.icon-menu {
  display: inline-block;
  width: 140px;
  height: 10px;
  padding: 35px 0;
  border-top: 10px solid;
  border-bottom: 10px solid;
  background-color: currentColor;
  background-clip: content-box;
}

.icon-dot {
  display: inline-block;
  width: 100px;
  height: 100px;
  padding: 10px;
  border: 10px solid;
  border-radius: 50%;
  background-color: currentColor;
  background-clip: content-box;
}
```

[效果图](http://demo.cssworld.cn/4/2-4.php)

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/little_tricks/menu_dot.png)

# margin

## 元素尺寸的相关概念

* 元素尺寸：对应jQuery的`$().outerWidth()`和`$().outerHeight()`方法，是元素的**border box**尺寸。在原生DOM API中写作offsetWidth和offsetHeight，所以有时也称作“元素偏移尺寸”。

* 元素内部尺寸：对应jQuery的`$().innerWidth()`和`$().innerHeight()`，就是元素的**padding box**的尺寸。在原生DOM API中写作clientWidth和clientHeight。

* 元素外部尺寸：对应jQuery的`$().outerWidth(true)`和`$(true).outerHeight(true)`，是元素的**margin box**的尺寸。原生DOM API中没有对应的。

当元素的尺寸表现为充分利用可利用空间的时候，margin会影响元素的尺寸。对于普通流体元素，margin只能改变元素水平方向尺寸。对于具有可拉伸的绝对定位元素，则水平和垂直方向都可以。

## margin合并

margin合并的三种场景：

1. 相邻兄弟元素margin合并。

2. 父级和第一个、最后一个子元素。

3. 空块级元素的margin合并。块级元素为空，上下margin会合并。

margin合并规则，“正正取最大”“正负相加”“负负取最负”。

margin合并的意义主要是为了文档的展示，即css2的主要内容。
v
## margin: auto

平分规则：

1. 如果一侧定值，一侧为auto，则auto为剩余空间大小。

2. 如果两侧均是auto，则平分剩余空间。

这里让某个元素右对齐就可以用，`margin-left：auto`这个属性啦。

这里介绍一下水平垂直同时居中的一个方法。容器尺寸固定且，`position: relative`。

```css
.son {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  width: 200px;
  height: 100px;
  margin: auto;
}
```

在这种格式化的流体布局中，margin的自动分配剩余空间就起作用啦，所以最后会垂直水平居中。

## margin无效情况

1. display计算值为inline的非替换元素的垂直margin是无效的。对于内联替换元素的垂直margin有效。

2. 表格的<tr>和<td>元素设置display值是`table-cell`或`table-row`的margin无效。如果计算值是`tab-caption`、`inline-table`，可以通过margin控制外边距，甚至::first-letter伪元素也可以解析margin。

3. margin合并时候，margin小于合并值设置无效。

4. 绝对定位元素非定位方位的margin值“无效”。绝对定位元素无法影响兄弟元素，所以看起来无效。

5. 定高容器的子元素的margin-bottom或者宽度定死的子元素的margin-right的定位“失效”。正常流的情况下，margin-top和margin-left有效，而margin-bottom和margin-right则会无效。虽然有，但是影响不到自己，只会影响兄弟元素的定位。

6. float鞭长莫及的margin。`float: right`和`margin-right: 20px`这里20px小于元素的宽度的时候，就会无效。

7. 内联特性导致的margin无效。


# border

border和padding与margin不同，border-width不支持百分比的数值。border相当于盒子的宽度，包装盒哪里有根据内部容器变大的情况呢。其他诸如outline、box-shadow、text-shadow等都是不支持百分比的数值的。

border-width还支持若干关键字。thin、medium、thick等。

border-style有几个点，关于border-style: none为默认值。border-style: double表现规则记录一下：双线宽度永远相等，中间间隔±1。

借助border也能实现菜单menu啦。

```css
.icon {
  width: 120px;
  height: 20px;
  border-top: 60px double;
  border-bottom: 20px solid;
}
```

## border三角形绘制。
等腰三角形的绘制：

```css
div {
  width: 0;
  border: 10px solid;
  border-color: #f30 transparent transparent;
}
```

首先我们需要知道border的绘制过程：

```css
div {
  width: 100px;
  height: 100px;
  border: 100px solid;
  border-color: #f30 #00f #396 #0f0;
}
```

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/little_tricks/border_color.png) 

所以其他三边透明和宽高为0的等宽三角想也就出来了：
```css
/* 梯形 */

div {
  width: 100px;
  height: 100px;
  border: 100px solid;
  border-color: #f30 transparent transparent;
}

/* 三角形 */

div {
  width: 0;
  border: 100px solid;
  border-color: #f30 transparent transparent;
}


```

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/little_tricks/border_color_transparent.png) ![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/little_tricks/border_single.png)

其他一些其他的三角形利用，则是利用两个倾斜度不同的三角形叠加。