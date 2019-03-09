---
title: css揭秘总结(四)
date: 2018-01-05 14:15:37
categories: CSS
tags: CSS
---
主要是CSS揭秘的最后一章，过渡与动画。
<!-- more -->
# 过渡和动画
## 42. 缓动效果
> 背景知识：基本的 CSS 过渡，基本的 CSS 动画

### 难题
给过渡和动画加上缓动效果（比如具有回弹效果的过渡过程）是一种流行的表现手法，可以让界面显得更加生动和真实：在现实世界中，物体从 A点到 B 点的移动往往不是完全匀速的。所以我们主要要讨论的就是回弹效果。这里主要要讲的就是贝塞尔曲线。

常见需要回弹的效果体验：
* 尺寸变化（比如：元素在 :hover 时变大，弹出框从 transform:scale(0) 的状态开始放大显示，柱状图中的每根柱子动态地冒出来，等等）
* 角度变化（比如：元素的旋转动作，饼图中的各个扇区以动画的方式从 0°开始展开为实际大小，等等）

### 弹跳动画
常见的内置缓动曲线：
![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/csssercets/easeLinear.png)

ease-out 是 ease-in 是反向版本。这一对组合正好是实现回弹效果所需要的：每当小球的运动方向相反时，我们希望调速函数也是相反的。
```css
@keyframes bounce {
 60%, 80%, to {
   transform: translateY(400px);
 animation-timing-function: ease-out;
 }
 70% { transform: translateY(300px); }
 90% { transform: translateY(360px); }
}
.ball {
 /* 其余样式写在这里 */
 animation: bounce 3s ease-in;
}
```

### cubic-bezier() 函数
把控制锚点的水平坐标和垂直坐标互换，就可以得到任何调速函数的反向版本。

cubic-bezier.com

### 弹性过渡
假设有一个文本输入框，每当它被聚焦时，都需要展示一个提示框。

如果我们需要一个文本框先慢慢扩大至1.1倍，再回弹至1倍，那么收缩的时候，会有一个scale(-0.1)的收缩，而不是scale(1.1)。
![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/csssercets/收缩.png)

这里我们需要单独设置，盖 `transition-duration`,可以用 `transition` 这个简写属性来覆盖所有的值，就不需要显式指定 `ease`，因为它本来就是初始值。再指定一下`transition-property`就可以了。
```css
input:not(:focus) + .callout {
 transform: scale(0);
 transition: .25s transform;
}
.callout {
 transform-origin: 1.4em -.4em;
 transition: .5s cubic-bezier(.25,.1,.3,1.5) transform;
}
```

## 43. 逐帧动画
在很多时候，我们需要一个很难（或不可能）只通过某些 CSS 属性的过渡来实现的动画。比如一段卡通影片，或是一个复杂的进度指示框。在这种场景下，基于图片的逐帧动画才是完美的选择；不过想在网页中以一种灵活的方式来实现这种动画，可谓是一项惊人的挑战。

### GIF动画短板
* GIF 图片的所能使用的颜色数量被限制在 256 色。
* GIF 不具备 Alpha 透明的特性。当我们不确定 GIF 动画的下层是什么时，这往往是一个大问题。
* 我们无法在 CSS 层面修改动画的某些参数，比如动画的持续时间、循环次数、是否暂停等。

2004 年，Mozilla 发起了一个建议：在 PNG 格式中增加对逐帧动画的支持，就像 GIF 格式同时支持静态图像和动画一样。这种格式被称作APNG。

### 解决方案
假设我们已经把动画中的所有帧全部拼合到一张 PNG 图片中了，如图:
![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/csssercets/菊花图.png)

动画平滑特性恰恰毁掉了我们想实现的逐帧动画效果。此时我们需要`steps()` ：
![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/csssercets/steps.png)

```html
<div class="loader">Loading...</div>
```

```css
@keyframes loader {
 to { background-position: -800px 0; }
}
.loader {
 width: 100px; height: 100px;
 background: url(img/loader.png) 0 0;
 animation: loader 1s infinite steps(8);
 /* 把文本隐藏起来 */
 text-indent: 200%;
 white-space: nowrap;
 overflow: hidden;
}
```

## 44. 闪烁效果
> 背景知识：基本的 CSS 动画，“逐帧动画”

### 难题
就是通过数次闪烁（不超过三次）来提示用户界面中有某处发生了变化，或者用来凸显出当前链接的目标。

### 解决方案
#### 50%循环动画
![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/csssercets/生硬闪烁.png)，
将动画的起点，调整到50%来避免生硬的跳转。

```css
@keyframes blink-smooth { 50% { color: transparent } }
.highlight {
 animation: 1s blink-smooth 3;
}
```

#### alternate动画（alternate-reverse动画）
![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/csssercets/alternate闪烁.png)
```css
@keyframes blink-smooth { to { color: transparent } }
.highlight {
 animation: .5s blink-smooth 6 alternate;
}
```

#### 普通闪烁效果
`steps(1)`本质上等同于 `steps(1, end)`。如果直接从0开始跳转，那么感觉就是没有任何效果。颜色值的切换只会发生在动画周期的末尾。因此，我们会看到起始值贯穿于整个动画周期，而终止值只在动画结尾的无限短的时间点处出现。

唯一的解决方案是调整动画的关键帧，让切换动作发生在 50% 处。
```css
@keyframes blink { 50% { color: transparent } }
.highlight {
  animation: 1s blink 3 steps(1); /* 或用step-end */
}
```

## 45. 打字动画
> 背景知识：基本的 CSS 动画，“逐帧动画”，“闪烁效果”

### 难题
有些时候，我们希望一段文本中的字符逐个显现，模拟出一种打字的效果。这个效果在技术类网站中尤为流行，用等宽字体可以营造出一种终端命令行的感觉。

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/csssercets/打字动画.png)

### 解决方案
核心思路就是让容器的宽度成为动画的主体：把所有文本包裹在这个容器中，然后让它的宽度从 0 开始以步进动画的方式、一个字一个字地扩张到它应有的宽度。你可能已经察觉到了，这个方法是有局限的：它并不适用于多行文本 。然而幸运的是，在绝大多数情况下，我们把这种效果应用在类似标题的单行文本上。

另外一件需要注意的事情是，动画的持续时间越长，动画效果越差：持续时间较短的动画会让界面显得更加精致，在某些场景下还是有益于可用性的。反之，动画的持续时间越长，越容易让用户感到厌烦。因此，即使这个技巧可以用在大段文本身上，也不一定是个好主意。

中间需要解决的问题有：
* 宽度的变化需要`steps`，所以`steps()`来修复。
* 我们已经用 em 单位指定了宽度，虽然它比像素单位要好一些，但仍然不够理想。通过 ch 单位来缓解。取值就是字符的数量。

```css
@keyframes typing {
 from { width: 0; }
}
h1 {
 width: 15ch; /* 文本的宽度 */
 overflow: hidden;
 white-space: nowrap;
 animation: typing 6s steps(15);
}
```

最后添加上一个闪烁的光标

```css
@keyframes typing {
 from { width: 0 }
}
@keyframes caret {
 50% { border-color: transparent; }
}
h1 {
 width: 15ch; /* 文本的宽度 */
 overflow: hidden;
 white-space: nowrap;
 border-right: .05em solid;
 animation: typing 6s steps(15),
 caret 1s steps(1) infinite;
}
```

## 46. 状态平滑的动画
> 背景知识: 基本的 CSS 动画，animation-direction（在“闪烁效果”中曾简要提及）

不是所有动画都是在页面一加载好就立即播放的。更常见的情况是，我们想通过动画来响应用户的动作，比如用户的鼠标悬停在某个元素上（:hover），或者按住鼠标（:active），等等。在这种场景下，我们将无法控制动画实际的循环次数，因为用户的动作会随时中断动画，而此时动画不可能刚好插放到我们事先指定的循环次数。举例来说，用户的鼠标可能会触发一个华丽的:hover 动画，而在动画还没有播完的时候，鼠标就从元素上移走了。在这种情况下，你觉得动画会如何收场呢？

### animation-play-state
暂停和继续一个一直存在的动画，重要的属性。
```css
animation-play-state: paused;
animation-play-state: running;
```

## 沿环形路径平移的动画
> 背景知识：CSS 动画，CSS 变形，“平行四边形”，“菱形图片”，“闪烁效果”

### 难题
![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/csssercets/环形路径.png)

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/Wallpapers/csssercets/404page.png)

```html
<div class="path">
 <img src="lea.jpg" class="avatar" />
</div>
```
```css
@keyframes spin {
 to { transform: rotate(1turn); }
}
.avatar {
 animation: spin 3s infinite linear;
 transform-origin: 50% 150px; /* 150px = 路径的半径 */
}
```
它不仅让头像沿着环形路径转动，同时还会让头像自身旋转

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/csssercets/自身旋转.png)

这里我只放出两个div方案。

### 需要两个元素的解决方案
用内层的变形来抵消外层的变形效果。

抵消作用是贯穿于整个动画的每一帧的

```html
<div class="path">
 <div class="avatar">
  <img src="lea.jpg" />
 </div>
</div>
```
```css
@keyframes spin {
 to { transform: rotate(1turn); }
}
@keyframes spin-reverse {
 from { transform: rotate(1turn); }
}
.avatar {
 animation: spin 3s infinite linear;
 transform-origin: 50% 150px; /* 150px = 路径的半径 */
}
.avatar > img {
  animation: spin-reverse 3s infinite linear;
}
```

利用一套动画css

```css
@keyframes spin {
 to { transform: rotate(1turn); }
 }
.avatar {
 animation: spin 3s infinite linear;
 transform-origin: 50% 150px; /* 150px = 路径的半径 */
}
.avatar > img {
 animation: inherit;
 animation-direction: reverse;
}
```
