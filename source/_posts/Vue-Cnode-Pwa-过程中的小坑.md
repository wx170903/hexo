---
title: Vue-Cnode-Pwa 过程中的小坑
date: 2018-02-21 21:34:46
categories: vue-items
tags: vue-cnode
---
总结下开发过程中的一些小坑。
<!-- more -->
# 项目概括
线上地址[https://brokenbones.xyz](https://brokenbones.xyz)
## 页面编写
页面编写主要采用的MVVM框架[vue.js](https://cn.vuejs.org/)开发，是直接用的vue-cli脚手架提供的PWA模板,页面样式采用的是[vuetify](https://vuetifyjs.com/zh-Hans/)，数据管理用的是[vuex](https://vuex.vuejs.org/zh-cn/intro.html)，api通讯采用的[axios](http://www.bootcdn.cn/axios/readme/),页面的路由采用的[vue-router](https://router.vuejs.org/zh-cn/)，其中我才用了history模式，这种模式更适合PWA。边写文章部分，采用了，Mavon-Editor的markdown编辑器。

## PWA部分
主要是自己编写了`mainfest.json`，sw.js则是利用**sw-precahe-webpack-plugin**生成。

## 域名注册
PWA应用必须采用https,这里我用的是腾讯云的免费的SSL证书，配置十分方便，推荐，免费一年。

## 项目结构
根据路由拆分=>项目的主要page有：

* 主题列表
  * 全部
  * 精华
  * 分享
  * 问答
  * 招聘
  * 测试
* 主题详情
  * 文章部分
  * 文章回复列表
  * 回复收藏文章、点赞和回复他人评论
* 个人中心
  * 最近回复
  * 最近发布
  * 话题收藏
* 我的消息
  * 已读消息
  * 未读消息
* 新建主题
  * Mavon-Editor
* 关于

**项目准备工作**

主要是，对常见的字体和ret.styl进行打包下载，也观察了其他一些人做相同的社区的页面的结构。

# 项目总结
## 路由部分
这个项目让我对vue-router的认识更加深了一步。
### HTML5 history模式
vue-router 默认 hash 模式 —— 使用 URL 的 hash 来模拟一个完整的 URL，于是当 URL 改变时，页面不会重新加载。

如果不想要很丑的 hash，我们可以用路由的 history 模式，这种模式充分利用 history.pushState API 来完成 URL 跳转而无须重新加载页面。

当你使用 history 模式时，URL 就像正常的 url，例如 http://yoursite.com/user/id　，也好看！

不过这种模式要玩好，还需要后台配置支持。因为我们的应用是个单页客户端应用，如果后台没有正确的配置，当用户在浏览器直接访问 http://oursite.com/user/id 就会返回 404，这就不好看了。

所以呢，你要在服务端增加一个覆盖所有情况的候选资源：如果 URL 匹配不到任何静态资源，则应该返回同一个 index.html 页面，这个页面就是你 app 依赖的页面。

后端配置见:[后端配置](https://router.vuejs.org/zh-cn/essentials/history-mode.html)

最后，给个警告，因为这么做以后，你的服务器就不再返回 404 错误页面，因为对于所有路径都会返回 index.html 文件。为了避免这种情况，你应该在 Vue 应用里面覆盖所有的路由情况，然后在给出一个 404 页面。
```js
const router = new VueRouter({
  mode: 'history',
  routes: [
    { path: '*', component: NotFoundComponent }
  ]
})
```

### 导航守卫和watch路由变化
导航守卫有：
* beforeRouteEnter
* beforeRouteUpdate (2.2 新增)
* beforeRouteLeave

这个项目中，我用了直接的`watch`路由变化和`beforeRouteUpdate`

主题详情界面有五个分路由，全部、问答和测试等。在路由设置中，我们保持的keep-alive，这样可以保持组件的状态保持，对于项目的体验比较好。可是页面的路由变化，如果使用的是同一组件的时候，页面的内容就不会变化，像主题的切换和主题详情的内容的变化等，这是我们就需要利用
`beforeRouteUpdate`或者`watch`路由的变化。

例如，ListView组件中的:
```js
beforeRouteUpdate(to, from, next) {
  this.postList = []
  this.getTabData(to.query.tab)
  next()
}
```
和TopicDetail中的`watch`:
```js
watch: {
  $route(to, from) {
    // 理解一下to＝＝要去的路由 记录TopicId,避免相同页面再重复加载
    // this.topic = []
    if (to.name === 'topic' && to.params.id !== this.topicId) {
      this.topic = {}
      this.getDetail(this.$route.params.id)
      if (from.name === 'topic') {
        this.setTopicId(from.params.id)
      }
    }
  }
}
```

### 页面的滚动状态保持
当我们，在主题列表滚动了一段距离之后，我们点击了一个主题，然后我们再次返回的时候，主题列表需要保持原来的滚动状态。

本来我的主题列表想要添加[better-scroll](https://ustbhuangyi.github.io/better-scroll/#/)的。但是每次页面滚动之后，点击新主题，再返回的时候，页面总会跳转到顶端，这样的话，浏览体验就非常不好。然后我了解了一下 better-scroll 的滚动原理：
![](http://static.galileo.xiaojukeji.com/static/tms/shield/scroll-4.png)
绿色部分为 wrapper，也就是父容器，它会有固定的高度。黄色部分为 content，它是父容器的第一个子元素，它的高度会随着内容的大小而撑高。那么，当 content 的高度不超过父容器的高度，是不能滚动的，而它一旦超过了父容器的高度，我们就可以滚动内容区了，这就是 better-scroll 的滚动原理。

这里，我们了解到了，better-scroll 的滚动原理中，我们需要一个固定的wrapper，这样的话，我们的内容在我们的页面中的话，就是在一个固定的窗口下。

vue-router 中有一个scrollBehavior ，[滚动行为](https://router.vuejs.org/zh-cn/advanced/scroll-behavior.html)，使用前端路由，当切换到新路由时，想要页面滚到顶部，或者是保持原先的滚动位置，就像重新加载页面那样。 vue-router 能做到，而且更好，它让你可以自定义路由切换时页面如何滚动。

我们需要记录scrollTop属性，上面采用 better-scroll 的话，我们的窗口是固定的，也就是，这里的scrollTop是相对于我们wrapper，而wrapper在页面中的位置是固定的就是没有scrollTop，这样的话，我们记录的scrollTop没有用。

如果用 better-scroll ，我们就需要页面跳转的话每次将scrollTop的值手动传给组件，十分麻烦，最后我放弃了并采用了原生的scroll。

### vue-router懒加载
没什么好说的，为了加载高效肯定要用的。

## 其他一些
### mavonEditor
论坛的文章编辑，现在大部分写文章的都是md了，所以我找了找，最好的还是这个，安装好npm包，加一个标签再加上一点配置接好了。地址：[mavonEditor](https://github.com/hinesboy/mavonEditor)

### 多处用的登录验证
没有登录的时候，很多地方都需要登录验证。因为有路由跳转逻辑，需要 js ，有了 mixin 就很好了。我们写好了 mixin 后，在我们需要添加登录页面的地方添加 mixin 和添加登录页面的组件。

最后，还想到什么有需要总结的还是会更新文章的。