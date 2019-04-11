---
title: transform/transtion/animation
date: 2017-02-06 14:50:01
categories: css3动画属性
description: 初步css3自带的动画属性。
tags:
- css3
- 动画
---
## fransform
在CSS3中transform主要包括以下几种：旋转rotate、扭曲skew、缩放scale和移动translate以及矩阵变形matrix。一起来看看CSS3中transform的旋转rotate、扭曲skew、缩放scale和移动translate具体如何实现，从transform的语法开始吧。是构成transtion和animation的基础。

#### 语法
  ```
  transform: rotate | scale | skew | translate |matrix;
  ```

#### 一、旋转rotate
  retate(value):通过指定的角度参数对原元素指定一个2D rotation(2D旋转)，需先有transform-origin属性的定义。transform-origin定义的是旋转点的基点，其中value是指旋转的角度，如果设置的值为正数表示顺时针，反之逆时针；

  ```
  transform:rolate(30deg);
  ```

#### 二、移动translate(x,y)
  移动tranlate(x,y),x水平和y垂直移动；
  
  translate(x,y):通过矢量(x,y)指定一个2D translation，x是第一个过渡值参数，y是第二个过渡值参数,如果 未被提供，则y以 0 作为其值。按照设定的x,y参数值,当值为负数时，反方向移动物
  体，其基点默认为元素 中心点，也可以根据transform-origin进行改变基点。如

  ```
  transform:translate(100px,20px)
  ```

#### 三、缩放scale(value,value)
  缩放scale和移动translate是极其相似，也具有三种情况：scale(x,y)使元素水平方向和垂直方向同时缩放（也就是X轴和Y轴同时缩放）；scaleX(x)元素仅水平方向缩放（X轴缩放）；scaleY(y)元素仅垂直方向缩放（Y轴缩放），但它们具有相同的缩放中心点和基数，其中心点就是元素的中心位置，缩放基数为1，如果其值大于1元素就放大，反之其值小于1，元素缩小。下面我们具体来看看这三种情况具体使用方法：
  ```
  transform:scale(2,1.5)
  transform:scaleX(2)
  transform:scaleY(1.5)
  ```

#### 四、扭曲skew
  第一个参数对应X轴，第二个参数对应Y轴。如果第二个参数未提供，则值为0，也就是Y轴方向上无斜切。skew是用来对元素进行扭曲变行，第一个参数是水平方向扭曲角度，第二个参数是垂直方向扭曲角度。其中第二个参数是可选参数，如果没有设置第二个参数，那么Y轴为0deg。同样是以元素中心为基点，我们也可以通过transform-origin来改变元素的基点位置。
  ```
  transform:skew(30deg,10deg);
  ```

## transition

#### 一、transition-property
  transition-property是用来指定当元素其中一个属性改变时执行transition效果，其主要有以下几个值：none(没有属性改变)；all（所有属性改变）这个也是其默认值；indent（元素属性名）。当其值为none时，transition马上停止执行，当指定为all时，则元素产生任何属性值变化时都将执行transition效果，ident是可以指定元素的某一个属性值。
  ````
  color: 通过红、绿、蓝和透明度组件变换（每个数值处理）如：background-color,border-color,color,outline-color等css属性；

  length: 真实的数字 如：word-spacing,width,vertical-align,top,right,bottom,left,padding,outline-width,margin,min-width,min-height,max-width,max-height,line-height,height,border-width,border-spacing,background-position等属性；

  percentage:真实的数字 如：word-spacing,width,vertical-align,top,right,bottom,left,min-width,min-height,max-width,max-height,line-height,height,background-position等属性；

  integer离散步骤（整个数字），在真实的数字空间，以及使用floor()转换为整数时发生 如：outline-offset,z-index等属性；

  number真实的（浮点型）数值，如：zoom,opacity,font-weight,等属性；

  transform list:详情请参阅：《CSS3 Transform》

  rectangle:通过x, y, width 和 height（转为数值）变换，如：crop

  visibility: 离散步骤，在0到1数字范围之内，0表示“隐藏”，1表示完全“显示”,如：visibility

  shadow: 作用于color, x, y 和 blur（模糊）属性,如：text-shadow

  gradient: 通过每次停止时的位置和颜色进行变化。它们必须有相同的类型（放射状的或是线性的）和相同的停止数值以便执行动画,如：background-image

  paint server (SVG): 只支持下面的情况：从gradient到gradient以及color到color，然后工作与上面类似

  space-separated list of above:如果列表有相同的项目数值，则列表每一项按照上面的规则进行变化，否则无变化

  a shorthand property: 如果缩写的所有部分都可以实现动画，则会像所有单个属性变化一样变化
  ````
#### 二、transition-duration
  语法：
  ````
  transition-duration:time 如(transition-duration: 2.2s;)
  ````
#### 三、transition-timing-function
  语法：
  ````
  transition-timing-function:ease | linear | ease-in | ease-out | ease-in-out |
  ````
  取值：
  ````
  transition-timing-function的值允许你根据时间的推进去改变属性值的变换速率，transition-timing-function有6个可能值：

  ease：（逐渐变慢）默认值，ease函数等同于贝塞尔曲线(0.25, 0.1, 0.25, 1.0).

  linear：（匀速），linear 函数等同于贝塞尔曲线(0.0, 0.0, 1.0, 1.0).

  ease-in：(加速)，ease-in 函数等同于贝塞尔曲线(0.42, 0, 1.0, 1.0).

  ease-out：（减速），ease-out 函数等同于贝塞尔曲线(0, 0, 0.58, 1.0).

  ease-in-out：（加速然后减速），ease-in-out 函数等同于贝塞尔曲线(0.42, 0, 0.58, 1.0)

  cubic-bezier：（该值允许你去自定义一个时间曲线）， 特定的cubic-bezier曲线。 (x1, y1, x2, y2)四个值特定于曲线上点P1和点P2。所有值需在[0, 1]区域内，否则无效。
  如（cubic-bezier(0.94,-0.1, 0.29, 1.2)）
  ````

<!-- #### 四、transition-delay -->


  