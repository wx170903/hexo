---
title: JavaScript设计模式与开发实践（一）
date: 2018-05-02 22:34:41
categories: JS
tags:
- JS
- 设计模式
---
全书分为三部分，基础知识，设计模式，设计原则和编程技巧
下面我会简单的总结书中的一些有用的设计模式。
<!-- more -->
**第一部分 基础知识**

这一部分，我会快速总结相关的一些知识点，包括

* 面向对象的JavaScript
* this,call和apply
* 闭包和高阶函数

# 面向对象的JavaScript

首先我们要了解一些基本概念，关于动态类型语言，鸭子类型，原型委托和多态以及原型模式等。

## 动态类型语言和鸭子模式

所谓动态类型语言，必定是相对于静态类型语言所做的比较。

* 静态类型语言的优点首先是在编译时就能发现类型不匹配的错误,编辑器可以帮助我们提前避免程序在运行期间有可能发生的一些错误。静态类型语言的缺点首先是迫使程序员依照强契约来编写程序,为每个变量规定数据类型。
* 动态类型语言的优点是编写的代码数量更少,看起来也更加简洁,程序员可以把精力更多地放在业务逻辑上面。动态类型语言的缺点是无法保证变量的类型,从而在程序的运行期有可能发生跟类型相关的错误。

动态语言对变量类型的宽容建立在 **鸭子类型(duck typing)** 的概念。

所谓鸭子类型的主要思想就是：我们只需要关注对象的行为，或者叫需要对象实现的功能，而不需要关注对象本身。

## 多态

多态的实际含义是:同一操作作用于不同的对象上面,可以产生不同的解释和不同的执行结果。换句话说,给不同的对象发送同一个消息的时候,这些对象会根据这个消息分别给出不同的反馈。

静态语言中，多态的实现需要抽象一个超类，通过继承来实现。

多态的思想主要是“做什么”和“谁去做”，这样我们就能消除类型之间的耦合关系，完成多态的实现。

多态最根本的作用就是通过把过程化的条件分支语句转化为对象的多态性,从而消除这些条件分支语句。

下面来看一段简单的渲染map的代码，实现多态：

```js
var googleMap = {
    show: function(){
        console.log( '开始渲染谷歌地图' );
    }
};

var baiduMap = {
    show: function(){
        console.log( '开始渲染百度地图' );
    }
};

var renderMap = function( map ){
    if ( map.show instanceof Function ){
        map.show();
    }
};

renderMap( googleMap );
renderMap( baiduMap );

// 我们需要一个添加新的地图渲染，就死添加一个新的对象就行了，避免判断

var sosoMap = {
    show: function(){
        console.log( '开始渲染搜搜地图' );
    }
};
renderMap( sosoMap );
```

## 封装

在 JavaScript 或者其他语言中，封装有很多含义，不仅仅限定于封装数据，只要是将数据隐藏的“任何形式的封装”，我们都能够称之为封装。在 JS 中，我们常见的封装的封装形式就是作用域及闭包了。封装失对象之间的耦合变松散，我们只需要关注对象之间暴露的API，来实现所需功能了。

《设计模式》中有一段话用来描述封装

> “ 考虑你的设计中哪些地方可能变化,这种方式与关注会导致重新设计的原因相反。它不是考虑什么时候会迫使你的设计改变,而是考虑你怎样才能够在不重新设计的情况下进行改变。这里的关键在于封装发生变化的概念,这是许多设计模式的主题。”

这段话的核心也就是封装的本质， **“找到变化并封装之”**。

## 原型模式和基于原型继承的 JavaScript 对象系统

如同题目所示，原型模式早就融入到 JavaScript 中了。原型模式不单是一种设计模式,也被称为一种编程泛型。

原型模式是用于创建对象的一种模式。原型模式的核心就是克隆，原型模式的关键也是语言本身是否提供 clone 方法。ECMAScript 5 提供了 Object.create方法,可以用来克隆对象。

```js
var Plane = function(){
    this.blood = 100;
    this.attackLevel = 1;
    this.defenseLevel = 1;
};

var plane = new Plane();
plane.blood = 500;
plane.attackLevel = 10;
plane.defenseLevel = 7;

var clonePlane = Object.create( plane );
console.log( clonePlane );// 输出:Object {blood: 500, attackLevel: 10, defenseLevel: 7}
```

不支持Object.create 方法的浏览器中,则可以使用以下代码：
```js
Object.create = Object.create || function( obj ){
    var F = function(){};
    F.prototype = obj;

    return new F();
}
```

**JavaScript中的原型继承**

JavaScript遵循着一些原型编程的基本规则：
* 所有的数据都是对象。
* 要得到一个对象,不是通过实例化类,而是找到一个对象作为原型并克隆它。
* 对象会记住它的原型。
* 如果对象无法响应某个请求,它会把这个请求委托给它自己的原型。

1. 所有的数据都是对象

    JavaScript 在设计的时候,模仿 Java 引入了两套类型机制:基本类型和对象类型。基本类型包括 undefined 、 number 、 boolean 、 string 、 function 、 object 。从现在看来,这并不是一个好的想法。
    
    按照 JavaScript 设计者的本意,除了 undefined 之外,一切都应是对象。为了实现这一目标,number 、 boolean 、 string 这几种基本类型数据也可以通过“包装类”的方式变成对象类型数据来处理。

    我们不能说在 JavaScript 中所有的数据都是对象,但可以说绝大部分数据都是对象。那么相信在 JavaScript 中也一定会有一个根对象存在,这些对象追根溯源都来源于这个根对象。

    事实上,JavaScript 中的根对象是 Object.prototype 对象。 Object.prototype 对象是一个空的对象。我们在 JavaScript 遇到的每个对象,实际上都是从 Object.prototype 对象克隆而来的,Object.prototype 对象就是它们的原型。比如下面的 obj1 对象和 obj2 对象:

    ```js
    var obj1 = new Object();
    var obj2 = {};
    ```

    可以利用 ECMAScript 5 提供的 Object.getPrototypeOf 来查看这两个对象的原型:

    ```js
    console.log( Object.getPrototypeOf( obj1 ) === Object.prototype ); // 输出:true
    console.log( Object.getPrototypeOf( obj2 ) === Object.prototype ); // 输出:true
    ```

2. 要得到一个对象,不是通过实例化类,而是找到一个对象作为原型并克隆它
    
    了解下，我们new运算符从构造琦中得到一个对象的过程。

    当使用 new 运算符来调用函数时,此时的函数就是一个构造器。 用new 运算符来创建对象的过程,实际上也只是先克隆 Object.prototype 对象,再进行一些其他额外操作的过程。

3. 对象会记住它的原型

    就 JavaScript 的真正实现来说，其实不能说对象有原型，而至能说对象的构造器有原型。对于”对象吧把请求委托给它自己的原型“这句话，更好的说法是对象委托给它的构造器的原型。那么队形如何把请求顺利的转交给它的构造器的原型呢？

    JavaScript 给对象提供了一个名为 __proto__ 的隐藏属性,某个对象的 __proto__ 属性默认会指向它的构造器的原型对象,即 {Constructor}.prototype 。

    ```js
    var a = new Object();
    console.log ( a.__proto__=== Object.prototype );
    // 输出:true
    ```

    实际上, __proto__ 就是对象跟“对象构造器的原型”联系起来的纽带。正因为对象要通过__proto__ 属性来记住它的构造器的原型,所以我们用上一节的 objectFactory 函数来模拟用 new创建对象时, 需要手动给 obj 对象设置正确的 __proto__ 指向。

4. 如果对象无法响应某个请求,它会把这个请求委托给它的构造器的原型

    这条规则即是原型继承的精髓所在。

    在 JavaScript 中,每个对象都是从 Object.prototype 对象克隆而来的,如果是这样的话,我们只能得到单一的继承关系,即每个对象都继承自 Object.prototype 对象,这样的对象系统显然是非常受限的。

    实际上,虽然 JavaScript 的对象最初都是由 Object.prototype 对象克隆而来的,但对象构造器的原型并不仅限于 Object.prototype 上,而是可以动态指向其他对象。这样一来,当对象 a 需要借用对象 b 的能力时,可以有选择性地把对象 a 的构造器的原型指向对象 b ,从而达到继承的效果。下面的代码是我们最常用的原型继承方式:

    ```js
    var obj = { name: 'sven' };

    var A = function(){};
    A.prototype = obj;

    var a = new A();
    console.log( a.name );
    ```

    我们来看看执行这段代码的时候,引擎做了哪些事情。

    * 首先,尝试遍历对象 a 中的所有属性,但没有找到 name 这个属性。
    * 查找 name 属性的这个请求被委托给对象 a 的构造器的原型,它被 a. __proto__ 记录着并且
    指向 A.prototype ,而 A.prototype 被设置为对象 obj 。
    * 在对象 obj 中找到了 name 属性,并返回它的值。

    当我们期望得到一个“类”继承自另外一个“类”的效果时,往往会用下面的代码来模拟实现:

    ```js
    var A = function(){};
    A.prototype = { name: 'sven' };

    var B = function(){};
    B.prototype = new A();

    var b = new B();
    console.log( b.name ); // 输出:sven
    ```

    再看这段代码执行的时候,引擎做了什么事情。

    * 首先,尝试遍历对象 b 中的所有属性,但没有找到 name 这个属性。
    * 查找 name 属性的请求被委托给对象 b 的构造器的原型,它被 b. __proto__ 记录着并且指向B.prototype ,而 B.prototype 被设置为一个通过 new A() 创建出来的对象。
    * 在该对象中依然没有找到 name 属性,于是请求被继续委托给这个对象构造器的原型A.prototype 。
    * 在 A.prototype 中找到了 name 属性,并返回它的值。

## this 、 call 和 apply

在 JavaScript 的this总是指向一个对象，而具体指向那个对象是运行时 **基于函数的执行环境动态绑定的**，而非函数被声明时的环境。

### this 的指向

除去不常用的with和eval的情况，具体到实际应用中，this的指向大致可以分为以下四种。

* 作为对象的方法调用。
* 作为普通函数调用。
* 构造器调用。
* Function.prototype.call 或 Function.prototype.apply 调用。


### 1. 作为对象的方法调用
当函数作为对象的方法被调用时，this指向该对象：

```js
var obj = {
    a: 1,
    getA: function() {
        alert(this === obj); // 输出：true
        alert ( this.a ); // 输出: 1 
    }
};

obj.getA();
```

### 2. 作为普通函数调用

当函数不作为对象的属性被调用时，也就是我们常说的普通函数方式，此时的 this 总是指
向全局对象。在浏览器的 JavaScript 里，这个全局对象是 window 对象。

```js
window.name = 'globalName';

var getName = function() {
    return this.name;
};

console.log(getName());  // globalName
```

注意一下，下面这个例子阐释了this在runtime定义：
```js
window.name = 'globalName';

var myObject = {
    name: 'sven',
    getName: function() {
        return this.name;
    }
};

var getName = myObject.getName;
console.log(getName());    // globalName
```

### 3. 构造器调用
JavaScript 中没有类，但是能通过 new 运算符从构造器中创建对象。

除了宿主提供的一些内置函数，大部分 JavaScript 函数都可以当作构造器使用。构造器的外表跟普通函数一模一样，它们的区别在于被调用的方式。当用 new 运算符调用函数时，该函数总会返回一个对象，通常情况下，构造器里的 this 就指向返回的这个对象，见如下代码：
```js
var MyClass = function() {
    this.name = 'sven';
};

var obj = new MyClass();
alert(obj.name); // anne
```

但用 new 调用构造器时，还要注意一个问题，如果构造器显式地返回了一个 object 类型的对象，那么此次运算结果最终会返回这个对象，而不是我们之前期待的 this：

```js
var MyClass = function() {
    this.name = 'sven',
    return { // 显式地返回一个对象
        name: 'anne'
    }
};

var obj = new MyClass();
alert(obj.name);
```

如果构造器不显式地返回任何数据，或者是返回一个非对象类型的数据，就不会造成上述问题：
```js
var MyClass = function(){
    this.name = 'sven'
    return 'anne'; // 返回 string 类型
};
var obj = new MyClass();
alert ( obj.name ); // 输出：sven 
```

### 4.  Function.prototype.call 或 Function.prototype.apply 调用
跟普通的函数调用相比，用 Function.prototype.call 或 Function.prototype.apply 可以动态地改变传入函数的 this：

```js
var obj1 = {
    name: 'sven',
    getName: function(){
        return this.name;
    }
};

var obj2 = {
    name: 'anne'
};

console.log( obj1.getName() ); // 输出: sven
console.log( obj1.getName.call( obj2 ) ); // 输出：anne 
```

### 丢失的this
何为丢失的this呢？

其实主要this的指向是runtime的，前面的对象的方法的函数在外部调用也是这个意思哈。

### call 和 apply
call和apply的区别主要就是在传入的参数形式不同。

```js
var func = function( a, b, c ){
    alert ( [ a, b, c ] ); // 输出 [ 1, 2, 3 ]
};

func.apply( null, [ 1, 2, 3 ] ); 

func.call( null, 1, 2, 3 ); 
```

**call和apply的用途**
1. 改变 this 指向
2. Function.prototype.bind

bind()方法会创建一个新函数，称为绑定函数，当调用这个绑定函数时，绑定函数会以创建它时传入 bind()方法的第一个参数作为 this，传入 bind() 方法的第二个以及以后的参数加上绑定函数运行时本身的参数按照顺序作为原函数的参数来调用原函数。

bind, 没错就是一个语法糖。当你希望改变上下文环境之后并非立即执行，而是回调执行的时候，使用 bind() 方法。而 apply/call 则会立即执行函数。

3. 借用其他对象的方法
借用方法的第一种场景是“借用构造函数”，通过这种技术，可以实现一些类似继承的效果：
```js
var A = function( name ){
    this.name = name;
};
var B = function(){
    A.apply( this, arguments );
};
B.prototype.getName = function(){
    return this.name;
};
var b = new B( 'sven' );
console.log( b.getName() ); // 输出： 'sven' 
```

```js
(function(){
    Array.prototype.push.call( arguments, 3 );
    console.log ( arguments ); // 输出[1,2,3]
})( 1, 2 ); 
```
想把 arguments 转成真正的数组的时候，可以借用 Array.prototype.slice 方法；想截去
arguments 列表中的头一个元素时，又可以借用 Array.prototype.shift 方法。那么这种机制的内
部实现原理是什么呢？我们不妨翻开 V8 的引擎源码，以 Array.prototype.push 为例，看看 V8 引
擎中的具体实现：

```js
function ArrayPush() {
    var n = TO_UINT32( this.length ); // 被 push 的对象的 length
    var m = %_ArgumentsLength(); // push 的参数个数
    for (var i = 0; i < m; i++) {
        this[ i + n ] = %_Arguments( i ); // 复制元素 (1)
    }
    this.length = n + m; // 修正 length 属性的值 (2)
    return this.length;
}; 
```

Array.prototype.slice 实现条件：
* 对象本身要可以存取属性
* 对象的 length 属性可读写

## 闭包和高阶函数

### 闭包

对于 JavaScript 程序员来说，闭包（closure）是一个难懂又必须征服的概念。闭包的形成与变量的作用域以及变量的生存周期密切相关。下面我们先简单了解这两个知识点。

#### 1. 变量的作用域

当再函数中声明一个变量的时候，如果该变量前面没有带上关键字var，这个变量就会成为全局变量。所以避免变量声明前面不加声明关键词。

#### 2. 变量的生存周期

除了变量的作用域之外，另外一个跟闭包有关的概念是变量的生存周期。

对于全局变量来说，全局变量的生存周期当然是永久的，除非我们主动销毁这个全局变量。

而对于在函数内用 var 关键字声明的局部变量来说，当退出函数时，这些局部变量即失去了它们的价值，它们都会随着函数调用的结束而被销毁：


```js
var func = function() {
    var a = 1;
    return function() {
        a++;
        alert(a);
    }
};

var f = func();

f(); // 输出：2
f(); // 输出：3
f(); // 输出：4
f(); // 输出：5 
```

在这里我们推出函数后，局部变量 a 并没有消失，而是似乎一直在某个地方存活着。原因是：当执行 var发= function();时，f返回一个匿名函数的引用，它可以访问到func()被调用是产生的环境，而局部变量a一直处在这个环境里。既然局部变量所在环境还能被外界访问，这个局部变量就有了不被销毁的理由。在这里就产生了一个闭包结构，局部变量的生命看起来被延续了。

闭包的一个经典应用是之前var声明的循环遍历中使用。
```js
var func = function() {
    for (var j = 0; j < 5; j++) {
        setTimeout((function(i) {
            console.log(i)
        })(j), 0)
    }
}

func() // 0, 1, 2, 3, 4
```
#### 闭包的更多作用

1. 封装变量
闭包可以帮助把一些不需要暴露在全局的变量封装乘“私有变量”。假设一个计算乘积的简单函数。
```js
var mult = (function() {
    var cache = {};
    var calculate = function() {
        var a = 1;
        for( var i = 0, l = arguments.length; i < l; i++) {
            a = a * arguments[i];
        }

        return a;
    };

    return function() {
        var args = Array.prototype.join.call(arguments, ',');
        if ( args in cache ){
            return cache[ args ];
        }

        return cache[args] = calculate.apply( null, arguments );
    }
})
```

2. 延续局部变量的寿命

####　闭包和面向对象设计
过程与数据的结合是形容面向对象中的“对象”时经常使用的表达。对象以方法的形式包含了过程,而闭包则是在过程中以环境的形式包含了数据。通常用面向对象思想能实现的功能,用闭包也能实现。反之亦然。

```js
// 闭包写法
var extent = function() {
    var value = 0;
    return {
        call: function() {
            value++;
            console.log(value)
        }
    }
};

var extent = extent();

extent.call();
extent.call();
extent.call();

// 面向对象写法
var extent = {
    value: 0,
    call: function() {
        this.value++;
        console.log(this.value);
    }
};

extent.call();
extent.call();
extent.call();

// 原型写法
var Extent = function() {
    this.value = 0;
};

Extent.prototype.call = function() {
    this.value++;
    console.log(this.value);
};

var extent = new Extent();

extent.call();
extent.call();
extent.call();
```

关于闭包域内存管理，将变量设置为 null 意味着切断变量与它此前引用的值之间的连接。当垃圾收集器下次运行时,就会删除这些值并回收它们占用的内存。

### 高阶函数

高阶函数是指至少满足下列条件之一的函数。

* 函数可以作为参数被传递;
* 函数可以作为返回值输出。


#### 函数作为参数传递

把函数当作参数传递,这代表我们可以抽离出一部分容易变化的业务逻辑,把这部分业务逻辑放在函数参数中,这样一来可以分离业务代码中变化与不变的部分。其中一个重要应用场景就是常见的回调函数。

1. 回调函数
2. Array.prototype.sort

```js
[ 1, 4, 3 ].sort( function( a, b ){
    return a - b;
});

// 输出: [ 1, 3, 4 ]
```

#### 函数作为返回值输出

相比把函数当作参数传递,函数当作返回值输出的应用场景也许更多,也更能体现函数式编程的巧妙。让函数继续返回一个可执行的函数,意味着运算过程是可延续的。

#### 高阶函数实现AOP

这一部分到后面装饰者模式再细讲。

#### 高阶函数的其他应用
1. currenying

首先我们讨论的是函数柯里化。currying 又称部分求值。一个 currying 的函数首先会接受一些参数,接受了这些参数之后,该函数并不会立即求值,而是继续返回另外一个函数,刚才传入的参数在函数形成的闭包中被保存起来。待到函数被真正需要求值的时候,之前传入的所有参数都会被一次性用于求值。

2. uncurrying

在 JavaScript 中，当我们调用对象的某个方法时，其实不用去关心该对象原本是否被设计为拥有这个方法，这是动态类型语言的特点，也是常说的鸭子类型思想。

同理，一个对象也未必只能使用它自身的方法，那么有什么办法可以让对象去借用一个原本不属于它的方法呢？

这里我们借用了一个泛化this的过程：

```js
Function.prototype.uncurrying = function() {
    var self = this;
    return function() {
        var obj = Array.prototype.shift.call(arguments);
        return self.apply(obj, arguments)
    };
};

// var push = Array.prototype.push.uncurrying();

// (function() {
//     push(arguments, 4);
//     console.log(arguments);
// })(1,2,3)

for (var i = 0, fn, arr = ['push', 'shift', 'forEach']; fn = arr[i++];) {
    Array[fn] = Array.prototype[fn].uncurrying();
};

var obj = {     
    'length': 3,
    '0': 1,
    '1': 2,
    '2': 3
}

Array.push(obj, 4);
console.log(obj.length)
```

3. 函数节流

