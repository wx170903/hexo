---
title: vue-music目录及样式初始化
date: 2017-10-12 11:06:30
categories: vue-items
tags: vue-music
---
# 目录初始化
## 常用目录
* api
* base
* common
    + fonts
    + image
    + js
    + stylus 
* components
* router
* store
    + actions
    + getters
    + index
    + mutation-types
    + mutations
    + state

<!-- more -->
### api
放置后端获取数据的js文件，常用方式有jsonp/axios。
* [jsonp的github](https://github.com/webmodules/jsonp)
* [axios的github](https://github.com/axios/axios)
### base
存放常用的可复用的基础component
### common
存放网页所需字体，图片，复用的js文件，和css样式文件。<br>
其中需要注意stylus文件引用顺序
```
@import "./reset.styl"
@import "./base.styl"
@import "./icon.styl"
```
### components
存放网页所需component
### router
存放网页路由文件
### store
存放网页vuex的共享数据

# eslint常用配置
[eslint规则](http://eslint.cn/docs/rules/)<br>
可以在官网查询配置相关规则
```
'rules': {
    // allow paren-less arrow functions
    'arrow-parens': 0,
    // allow async-await
    'generator-star-spacing': 0,
    // allow debugger during development
    'no-debugger': process.env.NODE_ENV === 'production' ? 2 : 0,
    'eol-last': 0,
    'space-before-function-paren': 0
  }
```
eslint规则的等级有三种:
* "off" 或者 0：关闭规则。
* "warn" 或者 1：打开规则，并且作为一个警告（不影响exit code）。
* "error" 或者 2：打开规则，并且作为一个错误（exit code将会是1）。

我们所配置的rules:
* arrow-parens: 要求箭头函数的参数使用圆括号
* generator-star-spacing: 强制 generator 函数中 * 号周围使用一致的空格
* no-debugger: 禁用debugger
* eol-last: 要求或禁止文件末尾存在空行
* space-before-function-paren: 强制在 function的左括号之前使用一致的空格

# webpack.base.conf 配置引用别名
配置别名，项目中引用不用添加src了。

```
resolve: {
    extensions: ['.js', '.vue', '.json'],
    alias: {
      '@': resolve('src'),
      'common': resolve('src/common'),
      'components': resolve('src/components'),
      'base': resolve('src/base'),
      'api': resolve('src/api')
    }
  },
```

