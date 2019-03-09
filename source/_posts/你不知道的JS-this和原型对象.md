---
title: 你不知道的JS-this和原型对象
date: 2018-05-14 15:11:34
categories: JS
tags:
- JS
- 你不知道的JS
---
这里我总结了下，《你不知道的javaScript上卷》第二部分的内容，总结的可能比较简短，相关的知识点，与设计模式中间有重合，设计模式中的相关设计模式，后期也会补上。
<!-- more -->
# 关于this

## 为什么要用this呢？
如果没有this，我们需要调用变量名，才能在函数或者方法中调用相关它自己，如果有了this，我们就能用一种更优雅的方式“传递”一个对象的引用。因此可以将API设计的更加简洁并且易于复用。

随着你使用的模式越来越复杂，显示传递上下文对象会使代码变得越来越混乱，使用this则不会这样。

## 误解
关于两种常见的对于this的解释，但是他们都是错误的。

### 指向自身
很多人很容易吧this联想到它的英文意思，指向函数本身，但是this的绑定是动态的！

看一个例子吧

```js
function foo(num) {
  console.log( "foo: " + num );
  // 记录 foo 被调用的次数
  this.count++;
}
foo.count = 0;
var i;
for (i=0; i<10; i++) {
  if (i > 5) {
    foo( i );
  }
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9
// foo 被调用了多少次？

console.log( foo.count ); // 0 -- WTF?
```

上面的this绑定到哪里了呢？答案是全局。

### 它的作用域
第二种常见的误解是，this指向函数的作用域。

再看一个例子。

```js
function foo() {
  var a = 2;
  this.bar();
}

function bar() {
  console.log( this.a );
}

foo(); // ReferenceError: a is not defined
```

这里我们既想用词法作用域，又想调用this来引入我们想用的函数体的变量。

这里稍微解析一下， `this.bar()` 引用的是外部的全局的 `bar()` 函数，这样里面的语句， `console.log(this.a)` 查找的也是全局的 `a` 变量。我们知道函数定义的 `a` 变量是影响不了全局的 `a` 变量的。所以这里我们会抛出一个引用错误。

### this到底是什么
排除了错误的理解后，我们看看this到底是什么样的机制。

之前我们说过this是在运行时进行绑定的，并不是在编写时绑定，他的上下文取决于函数调用时的各种条件。this的绑定和函数声明的位置灭有任何关系，只取决于函数的调用方式。

当一个函数被调用时，会创建一个活动记录（有时候也称为执行上下文）。这个记录会包含函数在哪里被调用（调用栈）、函数的调用方法、传入的参数等消息。this就是记录的其中一个属性，会在函数执行的过程中用到。

下一章我们会学习如何寻找函数的滴啊用位置，从而判断函数在执行的过程中会如何绑定this。

# this全面解析
## 调用位置
就像前面说的，调用位置就是函数在代码中被调用的位置（而不是声明的位置）。只有仔细分析调用位置才能回答这个问题：这个this到底引用的是什么？

通常来说，寻找滴啊用位置就是寻找“函数被调用的位置”，但是做起来没有这么简单，因为某些编程模式可能会隐藏真正的调用位置。

最重要的是要分析调用栈（就是为了到达当前执行位置所调用的所有函数）。我们关心的调用位置就在当前正在执行的函数的前一个调用中。

```js
function baz() {
  // 当前调用栈是：baz
  // 因此，当前调用位置是全局作用域
  console.log( "baz" );
  bar(); // <-- bar 的调用位置
}
function bar() {
  // 当前调用栈是 baz -> bar
  // 因此，当前调用位置在 baz 中
  console.log( "bar" );
  foo(); // <-- foo 的调用位置
}
function foo() {
  // 当前调用栈是 baz -> bar -> foo
  // 因此，当前调用位置在 bar 中 // 这里放一个断点可以查看当前调用栈，倒数第二个就是真正的调用位置。
  console.log( "foo" );
}

baz(); // <-- baz 的调用位置
```

## 绑定规则
这里也简要的解释四种规则：

### 1. 默认绑定
最常用的函数调用类型：独立函数调用。可以把这条规则看做是无法应用其他规则时的默认规则。

```js
function foo() {
  console.log( this.a );
}

var a = 2;
foo(); // 2
```

怎么知道应用了默认绑定呢？可以通过调用位置来看看 `foo()` 是如何调用的。在代码中，`foo()`是直接使用不带任何修饰的函数引用进行调用的，因此只能使用默认绑定，无法应用其他规则。如果使用严格模式，将默认无法使用默认绑定的，因为this会绑定为undefined。

### 2. 隐式绑定
另一种需要考虑的规则是调用位置是否有上下文对象，或者说是否被某个对象拥有或者包含。

常见的对象的方法，就是隐式绑定。对象属性引用链只有最顶层或者说最后一层会影响调用位置。

```js
function foo() {
  console.log( this.a );
}
var obj2 = {
  a: 42,
  foo: foo
};
var obj1 = {
  a: 2,
  obj2: obj2
};

obj1.obj2.foo(); // 42
```

#### 隐式丢失
因为this的绑定会根据runtime，所以，思考下面的代码：

```js
function foo() {
  console.log(this.a);
}

var obj = {
  a: 2,
  foo: foo
};

var bar = obj.foo;

var a = "全局对象a";

bar(); // 全局对象a
```


还有一种是常见的回调函数中的this隐式丢失，看下面代码：
```js
function foo() {
  console.log( this.a );
}

var obj = {
  a: 2,
  foo: foo
};

var a = "oops, global"; // a 是全局对象的属性

setTimeout( obj.foo, 100 ); // "oops, global"

// JavaScript 环境中内置的 setTimeout() 函数实现和下面的伪代码类似：
function setTimeout(fn,delay) {
  // 等待 delay 毫秒
  fn(); // <-- 调用位置！
}
```

接下来，我们会介绍相关的通过固定this来fix,this所指向的对象或者说上下文context。

### 3. 显示绑定
分析隐式绑定时，我们必须在一个对象的内部包含一个指向函数的属性，并通过这个属性间接引用函数，从而把this间接（隐式）绑定到这个对象上。

我们可以通过call(..)和apply(..)方法，来在某个函数中强制指定到this的上下文。

```js
function foo() {
  console.log( this.a );
}
var obj = {
  a:2
};

foo.call( obj ); // 2
```

可惜，显式绑定仍然无法解决我们之前提出的丢失绑定问题。

#### 1. 硬绑定
思考下面代码：
```js
function foo() {
  console.log(this.a);
}

var obj = function() {
  foo.call(obj);
};

var bar = function() {
  foo.call(obj);
};

```

这里，我们在 `bar` 内部实现了一个 `foo.call(obj)`，因此强制把 `foo` 的 `this` 绑定到了 `obj` 。无论之后如何调用函数 `bar` ，它总会在 `obj` 上调用 `foo` 。这种绑定是一种显式的强制绑定，因此我们称之为 **硬绑定**。

ES5中提供了内置的方法，**Function.prototype.bind**

#### 2. API调用的“上下文”
第三方库的许多函数，以及 **JavaScript** 语言和宿主环境中许多新的内置函数，都提供了一
个可选的参数，通常被称为“上下文”（context），其作用和 bind(..) 一样，确保你的回调
函数使用指定的 `this`。

举个栗子：
```js
function foo(el) {
  console.log( el, this.id );
}
var obj = {
  id: "awesome"
};

// 调用 foo(..) 时把 this 绑定到 obj
[1, 2, 3].forEach( foo, obj );
// 1 awesome 2 awesome 3 awesome
```

这些函数实际上就是通过 call(..) 或者 apply(..) 实现了显式绑定，这样你可以少些一些
代码。

### 4. new绑定
在JavaScript中，构造函数只是一些使用 `new` 操作符时被调用的函数。它们并不会属于某个类，也不会实例化一个类。实际上，它们甚至不能说是一种特殊的函数类型，它们只是被 `new` 操作符调用的普通函数。

使用 `new` 来调用函数，或者说发生构造函数调用时，会自动执行下面的操作。

1. 创建（或者说构造）一个全新的对象。
2. 这个新对象会被执行[[原型]]连接。
3. 这个新对象会绑定到函数调用的 `this` 。
4. 如果函数没有返回其他对象，那么 `new` 表达式中的函数调用会自动返回这个新对象。

## 优先级
1. 函数是否在new中调用（new绑定）？如果是的话this绑定的是新创建的对象。
2. 函数是否通过call、apply（显示绑定）或者硬绑定调用？如果是的话，this绑定的是制定的对象。
3. 函数是否在某个上下文对象中调用（隐式绑定）？如果是的话，this绑定的是那个上下文对象。
4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到undefined，否则绑定到全局对象。

## 绑定例外
在某些场景下 this 的绑定行为会出乎意料，你认为应当应用其他绑定规则时，实际上应用
的可能是默认绑定规则。

### 1. 被忽略的this
如果你把null或者undefined作为this的绑定对象传入call、apply或者bind，这些值在调用的时候会被忽略，实际应用的是默认绑定规则。

如果函数并不关心this的话，你仍然需要传入一个占位符，常见的就是用null，可是这里用null来忽略this绑定可能会产生一些副作用。如果这个函数中确实使用了this，那默认绑定规则会把this绑定到全局对象，在浏览器这个对象是window，这将产生不可预计的后果。

**更安全的this**

一种“更安全”的做法是，闯入一个特殊的对象，常见方法，`Object.create(null)`。如果引入的是这个空对象，这样就比较安全了，这样很明确的表示this是空，即使函数中调用了this，也不会更改全局对象。

```js
function foo() {
    this.a = 200
    console.log(this.a);
}

var a = 2;

var ø = Object.create(null)

foo.call(ø)

console.log(a)
// 200 2
```

### 2. 间接引用
另一个需要注意的是，你有可能（有意或者无意地）创建一个函数的“间接引用”，在这
种情况下，调用这个函数会应用默认绑定规则。

常见的就是赋值的时候发生。

注意：对于默认绑定来说，决定 this 绑定对象的并不是调用位置是否处于严格模式，而是函数体是否处于严格模式。如果函数体处于严格模式，this 会被绑定到 undefined，否则this 会被绑定到全局对象。

### 3. 软绑定
```js
if (!Function.prototype.softBind) {
  Function.prototype.softBind = function(obj) {
    var fn = this;
    // 捕获所有 curried 参数
    var curried = [].slice.call( arguments, 1 );

    var bound = function() {
      return fn.apply(
          (!this || this === (window || global)) ?
            obj : this
          curried.concat.apply( curried, arguments )
      );
    };

    bound.prototype = Object.create( fn.prototype );
    return bound;
  };
}
```

`softBind(..)`的其他原理和ES5内置的bind(..)类似。它会对制定函数进行封装，首先会检查调用时的this，如果this绑定到全局对象或者undefined，那就把指定的默认对象obj绑定到this，否则不会修改this。
```js
function foo() {
  console.log("name: " + this.name);
}

var obj = { name: "obj" },
    obj2 = { name: "obj2" },
    obj3 = { name: "obj3" };

var fooOBJ = foo.softBind( obj );

fooOBJ(); // name: obj

obj2.foo = foo.softBind(obj);

obj2.foo(); // name: obj2 <---- 看！！！

fooOBJ.call( obj3 ); // name: obj3 <---- 看！

setTimeout( obj2.foo, 10 );
// name: obj <---- 应用了软绑定
```

## this词法

这里介绍一下箭头函数: `() => {}`
```js
function foo() {
  return (a) => {
    console.log(this.a);
  };
}

var obj1 = {
  a: 2
};

var obj2 = {
  a: 3
};

var bar = foo.call(obj1);
bar.call(obj2);
```
`foo()` 内部创建的箭头函数会捕获调用时`foo()`的this。由于 `foo()` 的this绑定到obj1，bar（引用箭头函数）的this也会绑定到obj1，箭头函数的绑定无法被修改。（new也不行！）

# 对象
两种形式的定义：声明（文字）形式和构造形式。
## 类型
了解一下，简单基本类型：`string`、`number`、`boolean`、`null`和`undefined`。null有时会被当做一种对象类型，但是这其实只是语言本身的一个bug。

JS中有很多复杂基本类型。这些是一些特殊的对象子类型。函数就是对象的一个子类型（从技术角度来说就是“可调用的对象”）

Javascript中的函数是“一等公民”，因为它们本质上和普通的对象一样（只是可以调用），所以可以像操作其他对象一样操作函数（比如当做另一个函数的参数）。

### 内置对象
内置对象有：`String`、`Number`、`Boolean`、`Object`、`Function`、`Array`、`Date`、`RegExp`和`Error`。

关于字面量的基本类型调用 `Object.prototype.toString()` 都会转化成对应的包装类型。

## 内容
内容听名字似乎存储在对象内部，其实在语言中，这些值的储存方式是多种多样的，一般不会存在对象容器的内部。存储在对象容器的内部的是这些属性的名称，它们就像指针（从技术角度来说就是引用）一样，指向这些值真正的存储位置。

`.a`通常指的是属性访问，["a"] 语法通常被称为“键访问”。在 `[".."]` 语法使用字符串来访问属性，所以可以在程序中构造这个字符串。

### 可计算属性名
```js
var prefix = "foo";
var myObject = {
  [prefix + "bar"]: "hello",
  [prefix + "baz"]: "world"
};

myObject["foobar"]; // hello
myObject["foobaz"]; // world
```

### 属性与方法
如果访问的对象是一个函数，在JS中，我们喜欢称之为 “方法”，实际上呢，这个所谓的“方法”也仅仅是对方法的引用。

### 数组
数组支持[]访问形式，不过数组期待的是数字下标。所以你添加的属性值并不会使数组的`length`变长。你完全可以把数组当做一个普通的键/值对来使用。

而且要注意了：**如果你试图向数组添加一个属性，但是属性名“看起来”像一个数字，那它会变成
一个数值下标（因此会修改数组的内容而不是添加一个属性）**

```js
var myArray = [ "foo", 42, "bar" ];

myArray["3"] = "baz";

myArray.length; // 4

myArray[3]; // "baz"
```

### 复制对象
复制不可避免需要讨论到的就是深拷贝和浅拷贝。

对于JSON安全的对象，这有一种方法可以用：`var newObj = JSON.parse( JSON.stringify( someObj ) );`

ES6中定义的 `Object.assign(..)` 可以用在浅拷贝上，`Object.assign(..)` 方法的第一个参数是目标对象，之后还可以跟一个或多个源对象。它会遍历一个或者多个源对象。它会遍历一个或多个源对象的多有自由键并把它们复制（使用 = 操作符赋值）到目标对象，最后返回目标对象。

#### 属性描述符
在ES5之前，JavaScript语言本身并没有提供可以直接检测属性特性的方法，比如判断属性是否是只读。

```js
var myObject = {
a:2
};
Object.getOwnPropertyDescriptor( myObject, "a" );

// {
// value: 2,
// writable: true,
// enumerable: true,
// configurable: true
// }
```

`Object.getOwnPropertyDescriptor(..)`和`Object.defineProperty(..)`了解一下。

常见配置：
1. Writable 是否可以修改属性的值
2. Configurable 只要属性是可配置的，就可以使用 `defineProperty(..)` 方法来修改属性描述符。关于`Configurable`配置为false，`writable`可以由`true`变为`false`，并且不能再变回来啦！操作不可逆哈。
3. Enumerable 最后一个属性描述符（还有两个，我们会在介绍 getter 和 setter 时提到）
是 `enumerable`。

放一下总结好了，这本书就总结到这里了：

### 小结
JavaScript 中的对象有字面形式（比如 var a = { .. }）和构造形式（比如 var a = newArray(..)）。字面形式更常用，不过有时候构造形式可以提供更多选项。

许多人都以为“JavaScript 中万物都是对象”，这是错误的。对象是 6 个（或者是 7 个，取决于你的观点）基础类型之一。对象有包括 function 在内的子类型，不同子类型具有不同的行为，比如内部标签 [object Array] 表示这是对象的子类型数组。

对象就是键 / 值对的集合。可以通过 .propName 或者 ["propName"] 语法来获取属性值。访问属性时，引擎实际上会调用内部的默认 [[Get]] 操作（在设置属性值时是 [[Put]]），[[Get]] 操作会检查对象本身是否包含这个属性，如果没找到的话还会查找 [[Prototype]]链（参见第 5 章）。

属性的特性可以通过属性描述符来控制，比如 writable 和 configurable。此外，可以使用Object.preventExtensions(..)、Object.seal(..) 和 Object.freeze(..) 来设置对象（及其属性）的不可变性级别。

属性不一定包含值——它们可能是具备 getter/setter 的“访问描述符”。此外，属性可以是可枚举或者不可枚举的，这决定了它们是否会出现在 for..in 循环中。

你可以使用 ES6 的 for..of 语法来遍历数据结构（数组、对象，等等）中的值，for..of会寻找内置或者自定义的 @@iterator 对象并调用它的 next() 方法来遍历数据值。

