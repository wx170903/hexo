---
title: 了解MV*模式
date: 2018-12-23 18:58:03
categories: computer
tags:
- MVC
- MVP
- MVVM
- 设计模式
---
找了和看了许多关于MVC、MVP、MVVM的资料，趁着周末同样是做一下总结。
<!-- more -->
# MV*需要解决的问题
在传统的GUI程序中，图形界面展示数据和信息，用户的输入行为会触发一些应用逻辑（application logic）可能会触发一些业务逻辑（business logic），业务逻辑会对数据进行相关的变更操作。

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/basic_knowledge/gui.png)

所以这里需要解决的问题有：View如何同步Model的变更，View和Model之间如何粘合在一起。

# MVC
MVC把应用程序分为了view、model层以及controller层， **controller** 的主要职责是 **进行Model和View** 之间的协作（路由、输入预处理等）应用逻辑； **model** 进行处理业务逻辑。它们的如下：

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/basic_knowledge/mvc.png)


## MVC的调用关系
一般来说网页上用户的的调用关系如下图所示：

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/basic_knowledge/mvc1.png)

用户对View操作后，View捕获到这个操作，会把处理权给Controller；Controller会对来自View的数据进行预处理，决定调用哪一个Model的接口；然后由Model执行相关的业务逻辑；当Model变更之后，会通过 **观察者模式（Observer Pattern）通知View** ； **View通过观察者模式** 收到Model变更的消息后，会向Model请求最新的数据，然后重新更新界面。

需要注意的点：

1. View把控制权交给Controller，Controller执行应用程序相关的应用逻辑（对来自View数据进行预处理、决定调用哪个Model的接口等等）。

2. Controller操作Model，Model执行业务逻辑对数据进行处理。Controller不会直接操作View，它并不用了解View。

3. View和Model的同步消息是通过观察者模式进行，而同步操作是由View自己请求Model的数据然后对视图进行更新。

## MVC的优缺点

优点：
1. 把业务逻辑和展示逻辑分离，模块化程度高。且当应用逻辑需要变更的时候，不需要变更业务逻辑和展示逻辑，只需要Controller换成另外一个Controller就行了（Swappable Controller）。
2. 观察者模式可以做到多视图同时更新。

缺点：
1. Controller测试困难。因为视图同步操作是由View自己执行，而View只能在有UI的环境下运行。在没有UI环境下对Controller进行单元测试的时候，应用逻辑正确性是无法验证的：Model更新的时候，无法对View的更新操作进行断言。
2. View无法组件化。View是强依赖特定的Model的，如果需要把这个View抽出来作为一个另外一个应用程序可复用的组件就困难了。因为不同程序的的Domain Model是不一样的。

### MVC在服务端调用
服务端接收到来自客户端的请求，服务端通过路由规则把这个请求交由给特定的Controller进行处理，Controller执行相应的应用逻辑，对Model进行操作，Model执行业务逻辑以后；然后用数据去渲染特定的模版，返回给客户端。

相对于MVC中的输入就是Controller：

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/basic_knowledge/mvc2.png)

# MVP
MVP模式将Controller改名为 Presenter，同时改变了通信方向：

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/basic_knowledge/mvp.png)

和MVC模式一样，用户对View的操作都会从View交移给Presenter。Presenter会执行相应的应用程序逻辑，并且对Model进行相应的操作；而这时候Model执行完业务逻辑以后，也是通过观察者模式把自己变更的消息传递出去，但是是传给Presenter而不是View。Presenter获取到Model变更的消息以后，**通过View提供的接口更新界面**。

关键点：
1. View不再负责同步的逻辑，而是由Presenter负责。Presenter中既有应用程序逻辑也有同步逻辑。
2. View需要提供操作界面的接口给Presenter进行调用。（关键）

对比在MVC中，Controller是不能操作View的，View也没有提供相应的接口；而在MVP当中，Presenter可以操作View，View需要提供一组对界面操作的接口给Presenter进行调用；Model仍然通过事件广播自己的变更，但由Presenter监听而不是View。

这里 View 非常薄，不部署任何业务逻辑，称为"被动视图"（Passive View），即没有任何主动性，而 Presenter非常厚，所有逻辑都部署在那里

## MVP优缺点

优点：
1. 便于测试。Presenter对View是通过接口进行，在对Presenter进行不依赖UI环境的单元测试的时候。可以通过Mock一个View对象，这个对象只需要实现了View的接口即可。然后依赖注入到Presenter中，单元测试的时候就可以完整的测试Presenter应用逻辑的正确性。
2. View可以进行组件化。在MVP当中，View不依赖Model。这样就可以让View从特定的业务场景中脱离出来，可以说View可以做到对业务完全无知。它只需要提供一系列接口提供给上层操作。这样就可以做到高度可复用的View组件。

缺点：
1. Presenter中除了应用逻辑以外，还有大量的View->Model，Model->View的手动同步逻辑，造成Presenter比较笨重，维护起来会比较困难。

# MVVM
MVVM 模式将 Presenter 改名为 ViewModel，基本上与 MVP 模式完全一致。ViewModel的含义就是 "Model of View"，视图的模型。它的含义包含了领域模型（Domain Model）和视图的状态（State）。 在图形界面应用程序当中，界面所提供的信息可能不仅仅包含应用程序的领域模型。还可能包含一些领域模型不包含的视图状态，例如电子表格程序上需要显示当前排序的状态是顺序的还是逆序的，而这是Domain Model所不包含的，但也是需要显示的信息。

可以简单把ViewModel理解为页面上所显示内容的数据抽象，和Domain Model不一样，ViewModel更适合用来描述View。

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/basic_knowledge/mvvm.png)

## MVVM的调用关系
MVVM的调用关系和MVP一样。但是，在ViewModel当中会有一个叫Binder，或者是Data-binding engine的东西。以前全部由Presenter负责的View和Model之间数据同步操作交由给Binder处理。你只需要在View的模版语法当中，指令式地声明View上的显示的内容是和Model的哪一块数据绑定的。当ViewModel对进行Model更新的时候，Binder会自动把数据更新到View上去，当用户对View进行操作（例如表单输入），Binder也会自动把数据更新到Model上去。这种方式称为：Two-way data-binding，双向数据绑定。可以简单而不恰当地理解为一个模版引擎，但是会根据数据变更实时渲染。

![](https://images-1253206717.cos.ap-guangzhou.myqcloud.com/basic_knowledge/mvvm1.png)

也就是说，MVVM把View和Model的同步逻辑自动化了。以前Presenter负责的View和Model同步不再手动地进行操作，而是交由框架所提供的Binder进行负责。只需要告诉Binder，View显示的数据对应的是Model哪一部分即可。

## MVVM的优缺点

优点：

1. 提高可维护性。解决了MVP大量的手动View和Model同步的问题，提供双向绑定机制。提高了代码的可维护性。
2. 简化测试。因为同步逻辑是交由Binder做的，View跟着Model同时变更，所以只需要保证Model的正确性，View就正确。大大减少了对View同步更新的测试。

缺点：
1. 对于大型的图形应用程序，视图状态较多，ViewModel的构建和维护的成本都会比较高。
2. 数据绑定的声明是指令式地写在View的模版当中的，这些内容是没办法去打断点debug的。

# 引用
* https://github.com/livoras/blog/issues/11
* https://stackoverflow.com/questions/4415904/business-logic-in-mvc
* https://addyosmani.com/resources/essentialjsdesignpatterns/book/#detailmvcmvp
* http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html