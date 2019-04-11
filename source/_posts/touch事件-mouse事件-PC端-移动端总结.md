---
title: touch事件&&mouse事件&&PC端&&移动端总结
date: 2018-07-12 17:04:01
categories: JS工具
tags: 
- JS基础事件
- 触摸
---
最近几天看的东西比较杂，抽空做下小总结。本来想研究下PC端的mousedown，mousemove，mouseup在移动端的浏览器的表现，如何用touch来替代，结果不知不觉，重新看了下JS高程的事件处理章节，关于各种浏览器的clinetX,pageX也看了不少，接下来就做下总结了。
<!-- more -->
# touches对象的各种参数
## 触摸事件
* `touchstart`: 手指放在屏幕上的时候出发：即使已经有一个手指放在屏幕上也会触发。
* `touchmove`：当手指在屏幕上连续地出发。期间调用`preventDefault()`会阻止默认滚动。
* `touchend`：当手指从屏幕上移开时出发。
* `touchcancel`：当系统停止跟踪触摸时出发。

这几个事件都会冒泡，也都能取消。

下面放一下移动端跟踪触摸的时候三个event事件属性。
* `touches`：表示当前跟踪的触摸操作的`Touch`对象的数组。
* `targetTouches`：特定于事件目标的`Touch`对象的数组。
* `changedTouches`：表示自上次触摸以来发生改变的`Touch`对象的数组。

### touches

下面是touches属性介绍：

|属性|属性含义|
|-----------|-------|
|clientX, Y|触摸目标在视口中的x, y坐标，不包括任何滚动偏移|
|force|触摸手指挤压触摸平面的压力大小0.0-1.0之间|
|identifier|表示触摸的唯一ID|
|pageX, Y|触摸目标在页面中的x, y坐标|
|radiusX, Y|能够包围用户和触摸平面的接触面的最小椭圆的水平轴(X轴),垂直轴(Y轴)半径|
|rotationAngle|上述椭圆的能精准覆盖用户与触摸平面的接触面的角度，取值0-90之间|
|screenX, Y|返回触点相对于屏幕左边沿的的X, y坐标. 不包含页面滚动的偏移量|
|target|触摸touchstart触摸在屏幕上返回的element|


# mouse事件的各种参数
DOM3级事件中有九个鼠标事件，顺便放一下[事件速查表](https://developer.mozilla.org/zh-CN/docs/Web/Events).

|事件名|触发条件|
|-----|--------|
|click|用户点击主鼠标按钮或者按下回车键触发|
|dbclick|用户双击主鼠标按钮触发|
|mousemove|鼠标指针在元素内部移动时重复触发|
|mouseup|用户释放鼠标shi|
|mousedown|用户按下任意鼠标按钮触发，不能通过键盘触发这个事件|
|mouseenter|指针移到有事件监听的元素内（不冒泡）|
|mouseleave|指针移出元素范围外（不冒泡）|
|mouseover|指针移到有事件监听的元素或者它的子元素内|
|mouseout|指针移出元素，或者移到它的子元素上|

这里了解一下click的顺序，`mousedown` -> `mouseup` -> `click`。这里click是依赖先行事件mousedown，mouseup运行的。也就是说，假如我们的拖动的div上绑定有一个click事件，拖动的过程中，就会在mouseup后面触发click事件。所以我们可以将click事件绑定到mouseup上面，判断是否有mousemove来考虑触发click上绑定的事件的时机。

# 事件处理，事件监听，事件委托
DOM事件流规定中，事件流包含三个阶段：事件捕获阶段、处于目标阶段和事件冒泡阶段。在DOM事件流中，实际的元素在捕获阶段不会接收到事件。这意味着事件处理会被当做冒泡阶段的一部分。

## 事件处理的几种常见方法

### 1. HTML事件处理程序
直接在html上绑定一个事件，事件处理程序一般以`'on'`开头。
```html
<!-- 绑定执行语句 -->
<input type="button" value="click me" onclick="alert('clicked')"/>
<!-- 绑定函数 -->
<input type="button" value="click me" onclick="showMessage()" />
<script>
  function showMessage() {
    alert('clickclickclicked!')
  }
</script>
```

上述事件处理程序会创建一个封装着元素属性值的函数。这个函数中有一个局部变量event，也就是事件对象。

通过event变量，可以直接访问到事件对象，你不用从函数的参数列表读取。在函数内部，this等于事件的目标元素。那么这里需要注意了，事件处理函数中的this指向的问题。所以在需要添加到this的函数中，我们有时会用词法作用域context来保存需要调用的上下文。像上一篇文章中[函数节流](https://hddhyq.github.io/2018/07/08/%E5%87%BD%E6%95%B0%E5%8E%BB%E6%8A%96debounce%E5%92%8C%E5%87%BD%E6%95%B0%E8%8A%82%E6%B5%81throttle%E7%9A%84%E5%BA%94%E7%94%A8/#more)，就用到了相关保存运行时this的方法。

### 2. DOM 0级事件处理
使用传统的JavaScript指定事件处理方式，先获取到相关的需要操作对象的引用。

每个元素都有自己的事件处理程序属性，这些属性通常全部小写。

```js
var btn = document.getElementById('myBtn');

btn.onclick = function() {
  alert('this.id');  // 'myBtn'
}

// 清空事件绑定
btn.onclick = null;
```

这种形式添加的事件处理程序会在事件流的冒泡阶段被处理。

### 3. DOM 2级事件处理程序
所有的DOM节点都包含两个方法，`addEventListener()`和`removeEventListener()`，并接受三个参数：事件名，作为事件处理程序的函数和一个布尔值。

```js
var btn = document.getElementById('myBtn');

btn.addEventListener('click', function() {
  console.log('hello hdd');
}, false);

btn.removeEventListener('click', function() { // 没有用
  console.log('hello hdd');
}, false); 
// 上述代码无法删除相关的事件处理函数，因为两个匿名函数不是同一个函数了。
var btn = document.getElementById('myBtn');

var handler =  function() {
  console.log('hello hdd');
};

btn.addEventListener('click', handler, false);
btn.removeEventListener('click', handler, false);
```
## 事件对象
event对象包含与创建它的特定事件有关的属性和方法。触发的事件类型不一样，可用的属性和方法也不一样。不过所有事件都会有下表列出的成员。下表的属性都是只读的。

|属性/方法|类型|说明|
|--------|----|----|
|bubbles|Boolean|表明事件是否冒泡|
|cancelable|Boolean|表明是否可以取消事件的默认行为|
|currentTarget|Element|其事件处理程序当前正在处理的那个元素|
|defaultPrevented|Boolean|是否调用preventDefault()|
|detail|Integer|与事件相关的细节信息|
|eventPhase|Interger|调用事件处理程序的阶段：1.捕获阶段2.处于目标3.冒泡阶段|
|preventDefault()|Function|取消事件的默认行为。如果cancelable是true，则可以使用这个方法|
|stopImmediatePropagation()|Function|取消事件的进一步捕获或冒泡，同时阻止任何使劲按处理程序被调用|
|stopPropagation|Function|取消使劲按的进一步捕获或冒泡。如果bubbles为true，则可以使用这个方法|
|target|Element|事件的目标|
|trusted|Boolean|true表示浏览器生成的，false表示开发者通过JavaScript创建的|
|type|String|被触发的事件类型|
|view|AbstractView|与事件关联的抽象试图。等同于发生对象的window对象|

在事件处理程序的内部，this始终等于`currentTarget`的值，而`target`则只包含事件的实际目标。下面点击子元素：
1. 子元素绑定事件处理程序，父元素绑定事件处理函数。`currentTarget`等于父元素，`target`等于子元素
2. 子元素不绑定事件处理程序，父元素绑定事件处理函数。`currentTarget`等于父元素，`target`等于子元素
3. 子元素绑定事件处理程序，父元素不绑定事件处理函数。`currentTarget`等于子元素，`target`等于子元素

下面看几种常见的事件处理的冒泡次序：

```js
var btn = document.getElementById('myBtn');

btn.onclick = function(event) {
  alert(event.eventPhase);   // 2
};

document.body.addEventListener('click', function(event) {
  alert(event.eventPhase);   // 1
}, true);

document.body.onclick = function(event) {
  alert(event.eventPhase);   // 3
};

```

最后，事件处理程序执行期间，event对象一直存在，一旦事件处理程序执行完成，event对象就会销毁。


## 事件委托
想页面添加大量的事件处理程序会占用大量的内存，内存中的对象越多，性能也就越差。而且，必须事先指定所有事件处理程序而导致的DOM访问次数会延迟整个页面的交互就绪时间。

要理解事件委托，事件冒泡和捕获必须梳理一下。

当一个事件发生在具有父元素的元素导航，现代浏览器运行两个不同的阶段-捕获阶段和冒泡阶段。

在现代浏览器中，默认情况下，多有时间处理程序都在冒泡阶段注册，因此当我们点击子元素的时候。会沿着这个事件冒泡线路：
* 发现了子元素的事件处理程序，并运行了它。
* 往外冒泡发现父元素的事件处理程序，并运行它。

避免这个问题的方法就是使用`stopPropagation()`修复问题。

利用了事件冒泡会发现父元素的事件处理程序。我们就可以利用事件委托。

如果我们有大量子元素，想要点击任何一个都可以运行一段代码，可以将事件监听器设置在父节点元素，而且对于新添加的元素，事件委托很好的能为动态添加的元素动态的绑定事件处理函数。下面是例子

```html
<ul id="lists">
  <li id="list1">列表第一项</li>
  <li id="list2">列表第二项</li>
  <li id="list3">列表第三项</li>
</ul>
```

```js
var list = document.getElementById('lists');

document.addEventListener('click',function(event) {  // 也可以绑定lists元素
  var target = event.target;

  switch(target.id) {
    case 'list1':
      console.log(target.innerHTML);
      break;
    case 'list2':
      console.log('我是第二项');
      break;
    case 'list3':
      alert('hello world');
      break;
  }
})

```

# 浏览器中事件相关的常见几种位置坐标
## clientX/Y

获取到的是触发点相对浏览器可视区域左上角距离，不随页面滚动而改变。

## pageX/Y

pageX/Y获取到的是触发点相对文档区域左上角距离，会随着页面滚动而改变。

## offsetX/Y

offsetX/Y获取到是触发点相对被触发dom的左上角距离，不过左上角基准点在不同浏览器中有区别，其中在IE中以内容区左上角为基准点不包括边框，如果触发点在边框上会返回负值，而chrome中以边框左上角为基准点。

## layerX/Y

layerX/Y获取到的是触发点相对被触发dom左上角的距离，数值与offsetX/Y相同，这个变量就是firefox用来替代offsetX/Y的，基准点为边框左上角，但是有个条件就是，被触发的dom需要设置为`position:relative`或者`position:absolute`，否则会返回相对html文档区域左上角的距离。

## screenX/Y

screenX/Y获取到的是触发点相对显示器屏幕左上角的距离，不随页面滚动而改变。

一个图解释：

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/little_tricks/screenX_Y.jpg)


# 简单的移动端/PC端拖动代码
{% jsfiddle brokenbonesdd/jabtn2g6 js,html,css,result dark %}

上面的代码中，我将事件的type做了一下判断，进而绑定相关的。

# 关于拖拽事件的常见bug及处理方式

介绍下常见的，拖拽小问题。

## 1. 鼠标移动过快，离开拖拽物体
由于拖拽的div太小了，拖拽物体不在随着鼠标移动，前面介绍事件的时候也提到了，鼠标移动元素，`mousemove`事件不再触发，这时我们可以将拖拽物体上的`mousemve`事件放到`document`上面，同时将`mouseup`也改为`document`上面的事件

## 2. 会出现将div脱出浏览器窗口
限制div的拖动距离

还有一些常见bug，一般阻止默认事件都可以解决。

# 部分参考
* [http://www.cnblogs.com/moqiutao/p/5050225.html](http://www.cnblogs.com/moqiutao/p/5050225.html)
* [http://www.cnblogs.com/yufann/p/JS-Summary9.html](http://www.cnblogs.com/yufann/p/JS-Summary9.html)