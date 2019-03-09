---
title: css世界之层叠规则
date: 2018-10-02 17:06:46
categories: css世界
tags:
- css
---
这是关于css世界我总结的最后一篇。层叠规则
<!-- more -->
# z-index只是一小部分

CSS世界中，z-index属性只有和定位元素（position不为static的元素）在一起的时候才有作用，可以是正数也可以是负数。在CSS3中，flex盒子的子元素也可以设置z-index属性。

## 理解层叠上下文和层叠水平
层叠上下文，英文称作stacking context，是HTML中的一个三维概念。

1. 位于最下面的 background/border 特指层叠上下文元素的边框和背景色。每一个层叠顺序规则仅适用于当前层叠上下文元素的小世界。

2. inline 水平盒子指的是包括 inline/inline-block/inline-table 元素的“层叠顺序”，它们都是同等级别的。

3. 单纯从层叠水平上看，实际上 `z-index:0` 和 `z-index:auto` 是可以看成是一样的。注意这里的措辞—“单纯从层叠水平上看”，实际上，两者在层叠上下文领域有着根本性的差异。

### 层叠准则

1. 谁大谁上：当具有明显的层叠水平标识的时候，如生效的 z-index 属性值，在同一个层叠上下文领域，层叠水平值大的那一个覆盖小的那一个。

2. 后来居上：当元素的层叠水平一致、层叠顺序相同的时候，在 DOM 流中处于后面的元素会覆盖前面的元素。

### 层叠上下文特性

* 层叠上下文的层叠水平要比普通元素高。
* 层叠上下文可以阻断元素的混合模式。
* 层叠上下文可以嵌套，内部层叠上下文及其所有子元素均受制于外部的“层叠上下文”。
* 每个层叠上下文和兄弟元素独立，也就是说，当进行层叠变化或渲染的时候，只需要考虑后代元素。
* 每个层叠上下文是自成体系的，当元素发生层叠的时候，整个元素被认为是在父层叠上下文的层叠顺序中。

### 层叠上下文的创建

1. 天生派: 页面根元素天生具有层叠上下文，称之为根层叠上下文。

2. 正统派: z-index值为数值的定位元素的传统“层叠上下文”。

    对于 position 值为 relative/absolute 以及 Firefox/IE 浏览器（不包括 Chrome 浏览器）下含有 `position:fixed` 声明的定位元素，当其 z-index 值不是 auto 的时候，会创建层叠上下文。需要注意下，当两个相邻div的z-index为auto，将直接比较里面元素的z-index，当两个相邻div的层级都为相同数值的时候，父级的层级“后来居上”。Chrome 等 WebKit 内核浏览器下，`position:fixed` 元素天然层叠上下文元素，无须 z-index为数值。

3. 扩招派: 其他CSS3属性。

    * 元素为 `flex` 布局元素（父元素 display:flex|inline-flex），同时 z-index值不是 auto。
    * 元素的 `opacity` 值不是 1。
    * 元素的 `transform` 值不是 none。
    * 元素 `mix-blend-mode` 值不是 normal。
    * 元素的 `filter` 值不是 none。
    * 元素的 `isolation` 值是 isolate。
    * 元素的 `will-change` 属性值为上面 2～6 的任意一个（如 `will-change:opacity`、`will-chang:transform` 等）。
    * 元素的`-webkit-overflow-scrolling` 设为 `touch`。


![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/little_tricks/z_index_stack.png)

### z-index“不犯2”准则

对于非浮层元素，避免设置z-index值，z-index值没有任何道理需要超过2。

1. 定位元素一旦设置z-index值，就从普通定位元素变成了层叠上下文元素，相互间的层叠顺序就发生了根本的变化，很容易出现设置了巨大的 z-index 值也无法覆盖其他元素的问题。

2. 避免 z-index“一山比一山高”的样式混乱问题。