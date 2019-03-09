---
title: JS中排序和搜索算法
date: 2018-02-23 14:51:34
categories: JS
tags:
- JS
- 算法
---
看《学习JavaScript数据结构与算法》，总结了下常用的排序，如冒泡排序、选择排序、插入排序、归并排序和快速排序，以及顺序搜索和二分搜索算法。
<!-- more -->
# 排序算法
排序算法之前，我们需要创建一个数组表示待排序和搜索的数据结构。
```js
function ArrayList(){
  var array = []; //{1}
  this.insert = function(item){ //{2}
    array.push(item);
  };
  this.toString= function(){ //{3}
    return array.join();
  };
} 
```

## 冒泡排序
冒泡排序最简单，不过它的时间复杂度也是最大的。

冒泡排序比较任何两个相邻的项，如果第一个比第二个大，则交换它们。元素项向上移动至正确的顺序，就好像气泡升至表面一样，冒泡排序因此得名。

```js
this.bubbleSort = function(){
  var length = array.length; //{1}
  for (var i=0; i<length; i++){ //{2}
    for (var j=0; j<length-1; j++ ){ //{3}
      if (array[j] > array[j+1]){ //{4}
        swap(array, j, j+1); //{5}
      }
    }
  }
}; 

var swap = function(array, index1, index2){
  var aux = array[index1];
  array[index1] = array[index2];
  array[index2] = aux;
}; // 一个私有函数，只能用在ArrayList类的内部代码中

// 可以用ES6的数组拓展
// [array[index1], array[index2]] = [array[index2], array[index1]]; 
```
首先，声明一个名为length的变量，用来存储数组的长度（行{1}）。这一步可选，它能帮助我们在行{2}和行{3}时直接使用数组的长度。接着，外循环（行{2}）会从数组的第一位迭代至最后一位，它控制了在数组中经过多少轮排序（应该是数组中每项都经过一轮，轮数和数组长度一致）。然后，内循环将从第一位迭代至倒数第二位，内循环实际上进行当前项和下一项的比较（行{4}）。如果这两项顺序不对（当前项比下一项大），则交换它们（行{5}），意思是位置为j+1的值将会被换置到位置j处，反之亦然。

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/sortWithJs/冒泡算法.png)


**改进冒泡算法**
如果从内循环减去外循环中已跑过的轮数，就可以避免内循环中所有不必要的比较。
![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/sortWithJs/改进冒泡算法.png)

```js
this.modifiedBubbleSort = function(){
  var length = array.length;
  for (var i=0; i<length; i++){
    for (var j=0; j<length-1-i; j++ ){
      if (array[j] > array[j+1]){
        swap(j, j+1);
      }
    }
  }
};
```

最后，这个排序的事件复杂度是O(n²)。


## 选择排序
选择排序算法是一种原址比较排序算法。选择排序大致的思路是找到数据结构中的最小值并将其放置在第一位，接着找到第二小的值并将其放在第二位，以此类推。

```js
this.selectionSort = function(){
  var length = array.length, //{1}
    indexMin;
    for (var i=0; i<length-1; i++){ //{2}
      indexMin = i; //{3}
      for (var j=i; j<length; j++){ //{4}
      if(array[indexMin]>array[j]){ //{5}
        indexMin = j; //{6}
      }
    }
    if (i !== indexMin){ //{7}
    swap(i, indexMin);
    }
  }
}; 
```

首先声明一些将在算法内使用的变量（行{1}）。接着，外循环（行{2}）迭代数组，并控制迭代轮次（数组的第n个值——下一个最小值）。我们假设本迭代轮次的第一个值为数组最小值（行{3}）。然后，从当前i的值开始至数组结束（行{4}），我们比较是否位置j的值比当前最小值小（行{5}）；如果是，则改变最小值至新最小值（行{6}）。当内循环结束（行{4}），将得出数组第n小的值。最后，如果该最小值和原最小值不同（行{7}），则交换其值。

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/sortWithJs/选择排序.png)

选择排序同样也是一个复杂度为O(n²)的算法。和冒泡排序一样，它包含有嵌套的两个循环，这导致了二次方的复杂度。

## 插入排序
插入排序每次排一个数组项，以此方式构建最后的排序数组。假定第一项已经排序了，接着，它和第二项进行比较，第二项是应该待在原位还是插到第一项之前呢？这样，头两项就已正确排序，接着和第三项比较（它是该插入到第一、第二还是第三的位置呢？），以此类推。

```js
this.insertionSort = function(){
  var length = array.length, //{1}
      j, temp;
  for (var i=1; i<length; i++){ //{2}
    j = i; //{3}
    temp = array[i]; //{4}
    while (j>0 && array[j-1] > temp){ //{5}
      array[j] = array[j-1]; //{6}
      j--;
    }
    array[j] = temp; //{7}
  }
}; 
```
算法的第一行用来声明代码中使用的变量（行{1}）。接着，迭代数组来给第i项找到正确的位置（行{2}）。注意，算法是从第二个位置（索引1）而不是0位置开始的（我们认为第一项已排序了）。然后，用i的值来初始化一个辅助变量（行{3}）并将其值亦存储于一临时变量中（行{4}），便于之后将其插入到正确的位置上。下一步是要找到正确的位置来插入项目。只要变量j比0大（因为数组的第一个索引是0——没有负值的索引）并且数组中前面的值比待比较的值大（行{5}），我们就把这个值移到当前位置上（行{6}）并减小j。最终，该项目能插入到正确的位置上。

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/sortWithJs/插入排序.png)

排序小型数组时，此算法比选择排序和冒泡排序性能要好。

## 归并排序
归并排序是第一个可以被实际使用的排序算法。，其复杂度为O(nlogⁿ)。

> JavaScript的Array类定义了一个sort函数（Array.prototype.sort）用以排序JavaScript数组（我们不必自己实现这个算法）。ECMAScript没有定义用哪个排序算法，所以浏览器厂商可以自行去实现算法。例如，Mozilla Firefox使用归并排序作为Array.prototype.sort的实现，而Chrome使用了一个快速排序（下面我们会学习的）的变体。

归并排序是一种分治算法。其思想是将原始数组切分成较小的数组，直到每个小数组只有一个位置，接着将小数组归并成较大的数组，直到最后只有一个排序完毕的大数组。

```js
this.mergeSort = function(){
  array = mergeSortRec(array);
}; 

var mergeSortRec = function(array){
  var length = array.length;
  if(length === 1) { //{1}
    return array; //{2}
  }
  var mid = Math.floor(length / 2), //{3}
    left = array.slice(0, mid), //{4}
    right = array.slice(mid, length); //{5}
  return merge(mergeSortRec(left), mergeSortRec(right)); //{6}
}; 
```
归并排序将一个大数组转化为多个小数组直到只有一个项。由于算法是递归的，我们需要一个停止条件，在这里此条件是判断数组的长度是否为1（行{1}）。如果是，则直接返回这个长度为1的数组（行{2}），因为它已排序了。

如果数组长度比1大，那么我们得将其分成小数组。为此，首先得找到数组的中间位（行{3}），找到后我们将数组分成两个小数组，分别叫作left（行{4}）和right（行{5}）。left数组由索引0至中间索引的元素组成，而right数组由中间索引至原始数组最后一个位置的元素组成。

下面的步骤是调用merge函数（行{6}），它负责合并和排序小数组来产生大数组，直到回到原始数组并已排序完成。为了不断将原始数组分成小数组，我们得再次对left数组和right数组递归调用mergeSortRec，并同时作为参数传递给merge函数。

```js
var merge = function(left, right){
  var result = [], // {7}
      il = 0,
      ir = 0; 
  while(il < left.length && ir < right.length) { // {8}
    if(left[il] < right[ir]) {
      result.push(left[il++]); // {9}
    } else{
      result.push(right[ir++]); // {10}
    }
  }
  while (il < left.length){ // {11}
    result.push(left[il++]);
  }
  while (ir < right.length){ // {12}
    result.push(right[ir++]);
  }
  return result; // {13}
};
```

merge函数接受两个数组作为参数，并将它们归并至一个大数组。排序发生在归并过程中。首先，需要声明归并过程要创建的新数组以及用来迭代两个数组（left和right数组）所需的两个变量（行{7}）。迭代两个数组的过程中（行{8}），我们比较来自left数组的项是否比来自right数组的项小。如果是，将该项从left数组添加至归并结果数组，并递增迭代数组的控制变量（行{9}）；否则，从right数组添加项并递增相应的迭代数组的控制变量（行{10}）。

接下来，将left数组或者right数组所有剩余的项添加到归并数组中（行{11}和行{12}）。最后，将归并数组作为结果返回（行{13}）。

如果执行mergeSort函数，下图是具体的执行过程：
![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/sortWithJs/归并排序.png)

## 快速排序
快速排序也许是最常用的排序算法了。它的复杂度为O(nlogⁿ)，且它的性能通常比其他的复杂度为O(nlogⁿ)的排序算法要好。和归并排序一样，快速排序也使用分治的方法，将原始数组分为较小的数组（但它没有像归并排序那样将它们分割开）。

快速排序比到目前为止你学过的其他排序算法要复杂一些。让我们一步步地来学习。

1. 首先，从数组中选择中间一项作为主元。
2. 创建两个指针，左边一个指向数组第一个项，右边一个指向数组最后一个项。移动左指针直到我们找到一个比主元大的元素，接着，移动右指针直到找到一个比主元小的元素，然后交换它们，重复这个过程，直到左指针超过了右指针。这个过程将使得比主元小的值都排在主元之前，而比主元大的值都排在主元之后。这一步叫作划分操作。
3. 接着，算法对划分后的小数组（较主元小的值组成的子数组，以及较主元大的值组成的子数组）重复之前的两个步骤，直至数组已完全排序。

```js
this.quickSort = function(){
 quick(array, 0, array.length - 1);
}; 
```

就像归并算法那样，开始我们声明一个主方法来调用递归函数，传递待排序数组，以及索引
0及其最末的位置（因为我们要排整个数组，而不是一个子数组）作为参数。

```js
var quick = function(array, left, right){
  var index; //{1}
  if (array.length > 1) { //{2}
    index = partition(array, left, right); //{3}
    if (left < index - 1) { //{4}
      quick(array, left, index - 1); //{5}
    }
    if (index < right) { //{6}
      quick(array, index, right); //{7}
    }
  }
}; 
```

首先声明index（行{1}），该变量能帮助我们将子数组分离为较小值数组和较大值数组，这样，我们就能再次递归的调用quick函数了。partition函数返回值将赋值给index（行{3}）。

如果数组的长度比1大（因为只有一个元素的数组必然是已排序了的（行{2}），我们将对给定子数组执行partition操作（第一次调用是针对整个数组）以得到index（行{3}）。如果子数组存在较小值的元素（行{4}），则对该数组重复这个过程（行{5}）。同理，对存在较大值得子数组也是如此，如果存在子数组存在较大值，我们也将重复快速排序过程（行{7}）。

### 1. 划分过程
第一件要做的事情是选择主元（pivot），有好几种方式。最简单的一种是选择数组的第一项（最左项）。然而，研究表明对于几乎已排序的数组，这不是一个好的选择，它将导致该算法的最差表现。另外一种方式是随机选择一个数组项或是选择中间项。
```js
var partition = function(array, left, right) {
  var pivot = array[Math.floor((right + left) / 2)], //{8}
      i = left, //{9}
      j = right; //{10}
  while (i <= j) { //{11}
    while (array[i] < pivot) { //{12}
      i++;
    }
    while (array[j] > pivot) { //{13}
      j--;
    }
    if (i <= j) { //{14}
      swap(array, i, j); //{15}
      i++;
      j--;
    }
  }
  return i; //{16}
}; 
```

在本实现中，我们选择中间项作为主元（行{8}）。我们初始化两个指针：left（低——行{9}），初始化为数组第一个元素；right（高——行{10}），初始化为数组最后一个元素。

只要left和right指针没有相互交错（行{11}），就执行划分操作。首先，移动left指针直到找到一个元素比主元大（行{12}）。对right指针，我们做同样的事情，移动right指针直到我们找到一个元素比主元小。

当左指针指向的元素比主元大且右指针指向的元素比主元小，并且此时左指针索引没有右指针索引大（行{14}），意思是左项比右项大（值比较）。我们交换它们，然后移动两个指针，并重复此过程（从行{11}再次开始）。

在划分操作结束后，返回左指针的索引，用来在行{3}处创建子数组。

### 2. 快速排序实战
![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/sortWithJs/快排1.png)
![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/sortWithJs/快排2.png)
![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/sortWithJs/快排3.png)
![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/sortWithJs/快排4.png)
![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/sortWithJs/快排5.png)

# 搜索算法
## 顺序搜索
顺序或线性搜索是最基本的搜索算法。它的机制是，将每一个数据结构中的元素和我们要找的元素做比较。顺序搜索是最低效的一种搜索算法。

```js
this.sequentialSearch = function(item){
  for (var i=0; i<array.length; i++){
    if (item === array[i])
      return i;
  }
  }
  return -1;
}; 
```
## 二分搜索
二分搜索算法的原理和猜数字游戏类似，就是那个有人说“我正想着一个1到100的数字”的游戏。我们每回应一个数字，那个人就会说这个数字是高了、低了还是对了。

这个算法要求被搜索的数据结构已排序。以下是该算法遵循的步骤。
1. 选择数组的中间值。
2. 如果选中值是待搜索值，那么算法执行完毕（值找到了）。
3. 如果待搜索值比选中值要小，则返回步骤1并在选中值左边的子数组中寻找。
4. 如果待搜索值比选中值要大，则返回步骤1并在选种值右边的子数组中寻找。

```js
this.binarySearch = function(item){
  this.quickSort();
  var low = 0,
    high = array.length - 1,
    mid, element;

  while (low <= high){
    mid = Math.floor((low + high) / 2);
    element = array[mid];
    if (element < item) {
      low = mid + 1;
    } else if (element > item) {
      high = mid - 1;
    } else {
      return mid;
    }
  }
  return -1;
}; 
```

开始前需要先将数组排序，我们可以选择快速排序等。

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/sortWithJs/排序时间复杂度.png)