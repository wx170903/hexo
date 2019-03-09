---
title: 'headless chrome Puppeteer 无头or无界面 '
date: 2017-11-20 14:21:17
categories: Puppeteer
tags: 
- chrome
- Puppeteer
---
# <a id="instruction-puppeteer"></a> 概述Puppeteer
Puppeteer 是一个用来控制无界面Chrome的Node库，它提供了很多高级的api来使我们跟好的控制无界面的Chrome。这个headless可以叫无头，也可以叫无界面吧
<!-- more -->
## <a id="what-can-we-do"></a> 我们能做什么？
大多数我们能在浏览器上做的都能用Puppeteer来做！
* 生成页面的截图和PDF文件。
* 抓取spa并生成预渲染内容（即“SSR”）。
* 从网页爬取内容。
* 自动化表单提交、UI测试、键盘输入等。
* 自动化更新表格创建，自动化测试环境。
* 捕获站点的时间线跟踪，以帮助诊断性能问题。

了解这么多，我主要是从爬虫知道的这个工具。

因为这个工具相关的几个类似的工具(像PhantomJS)也大多停止更新了，谷歌真厉害。。
## <a id="shotscreens-pdf"></a> shotscreens 和 PDF实例
### 截图
page.screenshot([options]) 这个api 可以

```js
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://example.com');
  await page.screenshot({path: 'example.png'});

  await browser.close();
})();
```
### PDF
page.pdf(options)这个api来打印，也有一些options可以设置
比较常用的format：A4 之类的

```js
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://news.ycombinator.com', {waitUntil: 'networkidle2'});
  await page.pdf({path: 'hn.pdf', format: 'A4'});

  await browser.close();
})();
```

### 获取页面上下文

page.evaluate(pageFunction, ...args)， 这个api， 来解析
```js
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://example.com');

  // Get the "viewport" of the page, as reported by the page.
  const dimensions = await page.evaluate(() => {
    return {
      width: document.documentElement.clientWidth,
      height: document.documentElement.clientHeight,
      deviceScaleFactor: window.devicePixelRatio
    };
  });

  console.log('Dimensions:', dimensions);

  await browser.close();
})();
```

# <a id="common-api"></a> Puppeteer常用api分类
## <a id="summary"></a> 概览
![](https://user-images.githubusercontent.com/746130/31592143-089f6f9a-b1db-11e7-9a20-16b7fc754fa1.png)

* Puppeteer 使用开发者工具和浏览器通信
* Browser 拥有多个界面的实例
* Page 至少有一个框架：主框架。可能还有其他由iframe和frame标签创建的frames
* Frame 至少有一个执行上下文，默认的上下文是JavaScript的执行框架。框架可能有与执行相关联的附加执行上下文。

## <a id="environment-variable"></a> 环境变量
由`npm config`设置proxy代理，或关于下载的Url和版本之类的。

## <a id="class-puppeteer"></a> 类：Puppeteer
Puppeteer模块提供了一种方法来登录Chromium实例。

## <a id="class-browser"></a> 类：Browser
一个Browser 在Puppeteer连接到Chromium实例时被创建,通过`puppeteer.launch` 或 `puppeteer.connect`.
常用`browser.newPage()`新界面 ,`browser.close()`关闭browser和`browser.wsEndpoint()`断点打开。

## <a id="class-page"></a> 类：Page
Page提供了于Chromium单选项交互的方法。一个Browser有多个Page实例
基础的方法有：
一些原生的事件，还有`page.$(selector)`，`page.$$(selector)`来进行页面的元素选择，`page.$eval(selector, pageFunction[, ...args])`和`page.$$eval(selector, pageFunction[, ...args])`

```js
const searchValue = await page.$eval('#search', el => el.value);
const preloadHref = await page.$eval('link[rel=preload]', el => el.href);
const html = await page.$eval('.main-container', e => e.outerHTML);
```
`page.click(selector[, options])`鼠标点击，相关的点击事件也可以通过options来设置。

`page.content()`返回整个HTML的page内容，包括doctype。

`page.cookies(...urls)`,
`page.deleteCookie(...cookies)`

page.emulate(options)媒体设置。
```js
const puppeteer = require('puppeteer');
const devices = require('puppeteer/DeviceDescriptors');
const iPhone = devices['iPhone 6'];

puppeteer.launch().then(async browser => {
  const page = await browser.newPage();
  await page.emulate(iPhone);
  await page.goto('https://www.google.com');
  // other actions...
  await browser.close();
});
```

`page.evaluate(pageFunction, ...args)`对页面的上下文进行操作。
```js
const bodyHandle = await page.$('body');
const html = await page.evaluate(body => body.innerHTML, bodyHandle);
await bodyHandle.dispose();
```

`page.goto(url, options)`,`page.goForward(options)`和`page.goBack(options)`。

还可以对页面的js,css,html进行行管的操作。

`page.setUserAgent(userAgent)`,
`page.setViewport(viewport)`亦可以进行相关操作。

## <a id="class-keyboard"></a> 类：Keyboard
键盘相关输入，
```js
await page.keyboard.type('Hello World!');
await page.keyboard.press('ArrowLeft');

await page.keyboard.down('Shift');
for (let i = 0; i < ' World'.length; i++)
  await page.keyboard.press('ArrowLeft');
await page.keyboard.up('Shift');

await page.keyboard.press('Backspace');
// Result text will end up saying 'Hello!'
```

## <a id="class-mouse"></a> 类：Mouse
鼠标点击拖住事件。和键盘事件类似。

## <a id="class-touchscreen"></a> 类: Touchscreen
点击屏幕位置。
`touchscreen.tap(x, y)`

## <a id="class-dialog"></a> 类：Dialog
弹出框确认，取消以及相关的信息，输入都客气取得。

## <a id="class-console-message"></a> 类：ConsoleMessage
页面的console信息获取。

## <a id="class-frame"></a> 类：Frame
大部分page都有Frame的方法，相当于
`page.mainFrame()`。

`frame.childFrame()`，
`frame.name()`，
`frame.parentFrame()`相关操作。属于frame的。

## <a id="class-execution-context"></a> 类：ExecutionContext
类表示JavaScript执行的上下文。

## <a id="class-jshandle"></a> 类：JSHandle
JSHandle表示页面中的JavaScript对象。JSHandles可以由page.evaluateHandle方法创建。

## <a id="class-elementhandle"></a> 类：ElementHandle
elementhandle代表一个页面的DOM元素。elementhandles可以 `page.$`由方法。

`elementHandle.getProperties()`
`elementHandle.getProperty(propertyName)`
获取元素的属性值, 爬取页面的url可用。

`elementHandle.screenshot([options])`，
这个会将页面滚动到可以显示元素的地方，来截图。

`elementHandle.uploadFile(...filePaths)`上出文件。

## <a id="class-request"></a> 类：Request
当一个页面发送一个request，puppeteer's page会派发一下以下几个事件
* 'request'当页面发出请求时发出。
* 'response' 当响应请求接收到响应时发出。
* 'requestfinished' 当响应体被下载并请求完成时发出。

## <a id="class-response"></a> 类： Response
响应类表示由页面接收的响应。

# <a id="api-detail"></a> api详情

- [Overview](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#overview)
- [Environment Variables](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#environment-variables)
- [class: Puppeteer](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-puppeteer)
  * [puppeteer.connect(options)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#puppeteerconnectoptions)
  * [puppeteer.executablePath()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#puppeteerexecutablepath)
  * [puppeteer.launch([options])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#puppeteerlaunchoptions)
- [class: Browser](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-browser)
  * [event: 'disconnected'](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#event-disconnected)
  * [event: 'targetchanged'](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#event-targetchanged)
  * [event: 'targetcreated'](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#event-targetcreated)
  * [event: 'targetdestroyed'](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#event-targetdestroyed)
  * [browser.close()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#browserclose)
  * [browser.disconnect()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#browserdisconnect)
  * [browser.newPage()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#browsernewpage)
  * [browser.pages()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#browserpages)
  * [browser.targets()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#browsertargets)
  * [browser.version()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#browserversion)
  * [browser.wsEndpoint()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#browserwsendpoint)
- [class: Page](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-page)
  * [event: 'console'](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#event-console)
  * [event: 'dialog'](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#event-dialog)
  * [event: 'error'](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#event-error)
  * [event: 'frameattached'](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#event-frameattached)
  * [event: 'framedetached'](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#event-framedetached)
  * [event: 'framenavigated'](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#event-framenavigated)
  * [event: 'load'](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#event-load)
  * [event: 'metrics'](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#event-metrics)
  * [event: 'pageerror'](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#event-pageerror)
  * [event: 'request'](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#event-request)
  * [event: 'requestfailed'](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#event-requestfailed)
  * [event: 'requestfinished'](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#event-requestfinished)
  * [event: 'response'](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#event-response)
  * [page.$(selector)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageselector)
  * [page.$$(selector)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageselector)
  * [page.$$eval(selector, pageFunction[, ...args])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageevalselector-pagefunction-args)
  * [page.$eval(selector, pageFunction[, ...args])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageevalselector-pagefunction-args)
  * [page.addScriptTag(options)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageaddscripttagoptions)
  * [page.addStyleTag(options)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageaddstyletagoptions)
  * [page.authenticate(credentials)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageauthenticatecredentials)
  * [page.bringToFront()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagebringtofront)
  * [page.click(selector[, options])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageclickselector-options)
  * [page.close()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageclose)
  * [page.content()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagecontent)
  * [page.cookies(...urls)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagecookiesurls)
  * [page.deleteCookie(...cookies)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagedeletecookiecookies)
  * [page.emulate(options)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageemulateoptions)
  * [page.emulateMedia(mediaType)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageemulatemediamediatype)
  * [page.evaluate(pageFunction, ...args)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageevaluatepagefunction-args)
  * [page.evaluateHandle(pageFunction, ...args)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageevaluatehandlepagefunction-args)
  * [page.evaluateOnNewDocument(pageFunction, ...args)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageevaluateonnewdocumentpagefunction-args)
  * [page.exposeFunction(name, puppeteerFunction)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageexposefunctionname-puppeteerfunction)
  * [page.focus(selector)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagefocusselector)
  * [page.frames()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageframes)
  * [page.goBack(options)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagegobackoptions)
  * [page.goForward(options)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagegoforwardoptions)
  * [page.goto(url, options)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagegotourl-options)
  * [page.hover(selector)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagehoverselector)
  * [page.keyboard](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagekeyboard)
  * [page.mainFrame()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagemainframe)
  * [page.metrics()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagemetrics)
  * [page.mouse](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagemouse)
  * [page.pdf(options)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagepdfoptions)
  * [page.queryObjects(prototypeHandle)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagequeryobjectsprototypehandle)
  * [page.reload(options)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagereloadoptions)
  * [page.screenshot([options])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagescreenshotoptions)
  * [page.select(selector, ...values)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageselectselector-values)
  * [page.setContent(html)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagesetcontenthtml)
  * [page.setCookie(...cookies)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagesetcookiecookies)
  * [page.setExtraHTTPHeaders(headers)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagesetextrahttpheadersheaders)
  * [page.setJavaScriptEnabled(enabled)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagesetjavascriptenabledenabled)
  * [page.setOfflineMode(enabled)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagesetofflinemodeenabled)
  * [page.setRequestInterception(value)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagesetrequestinterceptionvalue)
  * [page.setUserAgent(userAgent)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagesetuseragentuseragent)
  * [page.setViewport(viewport)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagesetviewportviewport)
  * [page.tap(selector)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagetapselector)
  * [page.title()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagetitle)
  * [page.touchscreen](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagetouchscreen)
  * [page.tracing](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagetracing)
  * [page.type(selector, text[, options])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagetypeselector-text-options)
  * [page.url()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageurl)
  * [page.viewport()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageviewport)
  * [page.waitFor(selectorOrFunctionOrTimeout[, options[, ...args]])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagewaitforselectororfunctionortimeout-options-args)
  * [page.waitForFunction(pageFunction[, options[, ...args]])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagewaitforfunctionpagefunction-options-args)
  * [page.waitForNavigation(options)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagewaitfornavigationoptions)
  * [page.waitForSelector(selector[, options])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagewaitforselectorselector-options)
- [class: Keyboard](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-keyboard)
  * [keyboard.down(key[, options])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#keyboarddownkey-options)
  * [keyboard.press(key[, options])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#keyboardpresskey-options)
  * [keyboard.sendCharacter(char)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#keyboardsendcharacterchar)
  * [keyboard.type(text, options)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#keyboardtypetext-options)
  * [keyboard.up(key)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#keyboardupkey)
- [class: Mouse](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-mouse)
  * [mouse.click(x, y, [options])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#mouseclickx-y-options)
  * [mouse.down([options])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#mousedownoptions)
  * [mouse.move(x, y, [options])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#mousemovex-y-options)
  * [mouse.up([options])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#mouseupoptions)
- [class: Touchscreen](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-touchscreen)
  * [touchscreen.tap(x, y)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#touchscreentapx-y)
- [class: Tracing](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-tracing)
  * [tracing.start(options)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#tracingstartoptions)
  * [tracing.stop()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#tracingstop)
- [class: Dialog](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-dialog)
  * [dialog.accept([promptText])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#dialogacceptprompttext)
  * [dialog.defaultValue()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#dialogdefaultvalue)
  * [dialog.dismiss()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#dialogdismiss)
  * [dialog.message()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#dialogmessage)
  * [dialog.type](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#dialogtype)
- [class: ConsoleMessage](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-consolemessage)
  * [consoleMessage.args](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#consolemessageargs)
  * [consoleMessage.text](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#consolemessagetext)
  * [consoleMessage.type](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#consolemessagetype)
- [class: Frame](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-frame)
  * [frame.$(selector)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#frameselector)
  * [frame.$$(selector)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#frameselector)
  * [frame.$$eval(selector, pageFunction[, ...args])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#frameevalselector-pagefunction-args)
  * [frame.$eval(selector, pageFunction[, ...args])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#frameevalselector-pagefunction-args)
  * [frame.addScriptTag(options)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#frameaddscripttagoptions)
  * [frame.addStyleTag(options)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#frameaddstyletagoptions)
  * [frame.childFrames()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#framechildframes)
  * [frame.evaluate(pageFunction, ...args)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#frameevaluatepagefunction-args)
  * [frame.executionContext()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#frameexecutioncontext)
  * [frame.isDetached()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#frameisdetached)
  * [frame.name()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#framename)
  * [frame.parentFrame()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#frameparentframe)
  * [frame.select(selector, ...values)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#frameselectselector-values)
  * [frame.title()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#frametitle)
  * [frame.url()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#frameurl)
  * [frame.waitFor(selectorOrFunctionOrTimeout[, options[, ...args]])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#framewaitforselectororfunctionortimeout-options-args)
  * [frame.waitForFunction(pageFunction[, options[, ...args]])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#framewaitforfunctionpagefunction-options-args)
  * [frame.waitForSelector(selector[, options])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#framewaitforselectorselector-options)
- [class: ExecutionContext](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-executioncontext)
  * [executionContext.evaluate(pageFunction, ...args)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#executioncontextevaluatepagefunction-args)
  * [executionContext.evaluateHandle(pageFunction, ...args)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#executioncontextevaluatehandlepagefunction-args)
  * [executionContext.queryObjects(prototypeHandle)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#executioncontextqueryobjectsprototypehandle)
- [class: JSHandle](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-jshandle)
  * [jsHandle.asElement()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#jshandleaselement)
  * [jsHandle.dispose()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#jshandledispose)
  * [jsHandle.executionContext()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#jshandleexecutioncontext)
  * [jsHandle.getProperties()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#jshandlegetproperties)
  * [jsHandle.getProperty(propertyName)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#jshandlegetpropertypropertyname)
  * [jsHandle.jsonValue()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#jshandlejsonvalue)
- [class: ElementHandle](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-elementhandle)
  * [elementHandle.$(selector)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#elementhandleselector)
  * [elementHandle.$$(selector)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#elementhandleselector)
  * [elementHandle.asElement()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#elementhandleaselement)
  * [elementHandle.boundingBox()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#elementhandleboundingbox)
  * [elementHandle.click([options])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#elementhandleclickoptions)
  * [elementHandle.dispose()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#elementhandledispose)
  * [elementHandle.executionContext()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#elementhandleexecutioncontext)
  * [elementHandle.focus()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#elementhandlefocus)
  * [elementHandle.getProperties()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#elementhandlegetproperties)
  * [elementHandle.getProperty(propertyName)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#elementhandlegetpropertypropertyname)
  * [elementHandle.hover()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#elementhandlehover)
  * [elementHandle.jsonValue()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#elementhandlejsonvalue)
  * [elementHandle.press(key[, options])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#elementhandlepresskey-options)
  * [elementHandle.screenshot([options])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#elementhandlescreenshotoptions)
  * [elementHandle.tap()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#elementhandletap)
  * [elementHandle.toString()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#elementhandletostring)
  * [elementHandle.type(text[, options])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#elementhandletypetext-options)
  * [elementHandle.uploadFile(...filePaths)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#elementhandleuploadfilefilepaths)
- [class: Request](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-request)
  * [request.abort([errorCode])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#requestaborterrorcode)
  * [request.continue([overrides])](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#requestcontinueoverrides)
  * [request.failure()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#requestfailure)
  * [request.headers](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#requestheaders)
  * [request.method](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#requestmethod)
  * [request.postData](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#requestpostdata)
  * [request.resourceType](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#requestresourcetype)
  * [request.respond(response)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#requestrespondresponse)
  * [request.response()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#requestresponse)
  * [request.url](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#requesturl)
- [class: Response](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-response)
  * [response.buffer()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#responsebuffer)
  * [response.headers](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#responseheaders)
  * [response.json()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#responsejson)
  * [response.ok](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#responseok)
  * [response.request()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#responserequest)
  * [response.status](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#responsestatus)
  * [response.text()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#responsetext)
  * [response.url](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#responseurl)
- [class: Target](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-target)
  * [target.page()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#targetpage)
  * [target.type()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#targettype)
  * [target.url()](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#targeturl)
