---
title: 关于之前看看的一些css
date: 2018-06-12 14:44:28
categories: CSS
tags:
- CSS
---
记一下上个月看的图解CSS的一些简单的知识点哈。
<!-- more -->
# 元素百分比的相对计算

|相对的元素计算|属性|
|---|---|
|父元素宽度|`width`, `left`, `right`, `text-indent`, `padding`, `margin`等|
|父元素高度|`height`, `top`, `bottom`等|
|主轴长度|`flex-basis`|
|继承字号|`flex-size`|
|自身字号|`line-height`|
|自身宽高|`vertical-align`|
|自身设置|`background-position`; `border-image-slice`|
|特殊算法|`position: absolute`，相对于父元素最近的`position`不为`static`的祖先元素。如果没有这样的元素，则相对于视口。|

# 结构伪类选择器
|选择器|功能描述|
|---|---|
|E:first-child|作为父元素的第一个子元素E|
|E:last-child|作为父元素的最后一个子元素的元素E|
|E:root|选择匹配元素E所在文档的根元素|
|E F:nth-child(n)|选择父元素的第n个子元素F|
|E F:nth-last-child(n)|选择元素E的倒数第n个子元素F|
|E:nth-of-type(n)|选择父元素内具有指定类型的第n个E元素|
|E:nth-last-of-type(n)|选择父元素内具有指定类型的倒数第n个E元素|
|E:first-of-type|选择父元素内具有指定类型的的第一个E元素，与E:nth-of-type(1)相同|
|E:last-of-type|选择父元素内具有指定类型的的最后一个E元素，与E:nth-last-of-type(1)相同|
|E:only-child|选择父元素只包含一个字元素，且该子元素匹配E元素|
|E:only-of-type|选择父元素只包含一个同类型的子元素，且该子元素匹配E元素|
|E:empty|选择没有子类型的元素，而且该元素也不包含任何文本节点|

关于，伪类元素选择器中n的用法，也就是n取0到+∞，在子元素的个数范围内，取相对应的子元素。

# 伪元素
伪类一般反映无法在CSS中轻松或可靠地检测到的某个元素的属性或状态；伪元素则表示DOM外部的某种元素。

|伪元素|功能描述|
|---|---|
|::first-letter|选择文本块的第一个字母，除非在同一行中包含一些其他元素|
|::first-line|匹配元素的第一行文本|
|::before和::after|生成的内容不会成为DOM的一部分，但是可以设置样式|
|::selection|匹配突出显示的文本，可设置background和color|

# border相关
## border-shadow
|属性|功能描述|
|---|---|
|none|默认值，元素没有任何阴影效果|
|inset|阴影类型，可选值。默认外投影|
|x-offset|阴影水平偏移量，其值可以是正负值。正值在元素右边|
|y-offset|阴影垂直偏移量，其值可以是正负值。正值在元素底部|
|blur-radius|阴影模糊半径，可选参数。取值为0，表示阴影不具有模糊效果，取值越大，阴影的边缘越模糊|
|spread-radius|阴影扩展半径，可选参数。正值整个阴影都延展扩大，负值，整个阴影缩小|
|color|阴影颜色，不取值则是默认色，各个浏览器的默认色不一样的。|

# css3背景
主要有
* background-color 背景颜色
* background-image 背景图片
* background-repeat 背景图片展示方式
* background-attachment 背景图片是固定还是滚动
* background-position 背景图片位置

CSS3新增属性
* background-origin 指定绘制背景图片的起点（padding-box||border-box||content-box）
* background-clip 指定背景图片的显示范围（padding-box||border-box||content-box）
* background-size 指定背景图片的尺寸大小（auto||percentage||cover||contain）
* background-break 指定内联元素的背景图片进行平铺时的循环方式

# css文本类型
|属性|功能描述|取值|
|---|---|---|
|word-spacing|定义词与词之间的间距|normal，length（设置词与词之间间距，可以是负数）|
|letter-spacing|定义字符之间的间距|normal，length（设置词与词之间间距，可以是负数）|
|vertiacl-align|定义文本的垂直对齐方式|baseline、sub（上标对齐）、super（下标对齐）、bottom（行框底端对齐）、text-bottom（行内文本底端对齐）、top（顶端对齐）、middle（居中对齐）、百分比数字、长度|
|text-decoration|定义文本的修饰线|none、underline、overline、line-through、blink|
|text-indent|定义文本首行缩进|length（长度单位）和百分比|
|text-align|定义文本水平对齐方式|left（左对齐）、center（水平对齐）、right（右对齐）、justify（两端对齐）|
|line-height|定义文本行高|normal、长度值、百分比值、数字|
|text-transform|定义文本大小写|none、uppercase、lowercase、capitalize（首字母大写）|
|text-shadow|定义文本阴影效果||
|white-space|定义文字之间和文本之间的空白符间距|normal、nowrap（空白符合并，换行符忽略）、pre（空白符，换行符保留）、pre-wrap（空白符，换行符保留）、pre-line（空白符合，换行符保留）|
|direction|控制文本流入的方向|ltr、rtl（文本从右到左流入）、inhert（文本流入方向有继承获得）|


`text-overflow`：实现文本溢行处理，`word-break: break-all`来实现浏览器文本的换行

多行文本怎么实现文本溢出...,在webkit内核
```
display: -webkit-box
-webkit-line-clamp: 2
-webkit-box-orient: vertical
```

# 指定过渡属性transition-property
* none： 没有指定任何样式。
* all：默认值，表示指定元素所有支持transition-property属性的样式。
* `<single-transition-property>`：指定样式，其等于all或者<IDENT>。
* 颜色属性
* 具有长度值
* integer（离散步骤，整个数字，在真实的数字空间，以及使用floor()转换为整数时发生，如outline-offset，z-index）
* number
* 变形系列属性
* rectangle（通过x，y，width，height（转为数值）变换，如crop属性）
* visibility（离散步骤，在0~1范围内。0表示隐藏，1表示完全显示）
* 阴影（shadow，如text-shadow）
* 渐变（gradient）：通过每次停止时的位置和颜色进行变化。
* paint server(SVG)
* space-separated list of above
* 缩写属性

