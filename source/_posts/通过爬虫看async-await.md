---
title: 通过爬虫看async/await
date: 2017-11-24 15:23:43
categories: 爬虫
tags: 
- node
- ES6
- 爬虫
---

# <a id="crawler"></a>爬虫
先放下小练习地址 [小爬虫爬取图片](https://github.com/hddhyq/node-spider-test)，然后来研究下，爬虫需要注意些什么。爬虫始于种子（所需爬取url列表），通过`requset`从中爬取所需的页面的每一个帖子数，然后从中获取图片的url,然后下载。
<!-- more -->

## <a id="Request-Cheerio"></a> Request&&Cheerio
```ts
// 单个分页获取包涵图片URL
function getPages(url: string): Promise<string[]> {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      R(url, (err, res, body) => {
        const $ = cheerio.load(body)
        let liDom = $('body > div.content.clr > div.content_left > div.post_list_block_div > ul > li')
        const Pages = getPagesFromDom(liDom)
        resolve(Pages)
      })
    }, 1000)
  })
}
```

`const $ = cheerio.load(body)`就是Cheerio最基本的用法然后看一下`getPagesFromDom(liDom)`

```ts
function getPagesFromDom(dom: Cheerio) {
  return Array.from(dom).map((v, k) => {
    let t = v.attribs['onclick']
    let url = uri + t.substring(24, t.length - 2)
    return url
  })
}
```

中间我们传入的是一个`dom: Cheerio`对象，便于下一步的解析，对页面的帖子数相关连接的url进行记录。

接下来就是对每个帖子的发帖的前面的图片进行下载。

# <a id="delay-control"></a> 延时控制
```ts
async function getImgPages(pages: string[]) {
  const taskQ = pages.map(async (v, k) => {
    await sleep(1000 * k)   // 第二个节流方法
    return getPages(v)
  })

  let pageArray = await Promise.all(taskQ)
  const pagesArray = pageArray.reduce((prev, current) => prev.concat(current), [])
  return pagesArray
}

function getPages(url: string): Promise<string[]> {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      R(url, (err, res, body) => {
        const $ = cheerio.load(body)
        let liDom = $('body > div.content.clr > div.content_left > div.post_list_block_div > ul > li')
        const Pages = getPagesFromDom(liDom)
        resolve(Pages)
      })
    }, 1000)
  })
}

function sleep(ms: number) {
  return new Promise(resolve => setTimeout(resolve, ms))
}
```

这里我用了一个setTimeout定时和map定时来获取每个页面的时间进行限制。这样的方法并不是很好，也是**不推荐**的方法，具体推荐的方法看下面对于async/await的分析，最要是因为，后面用的`Promiseall(taskQ)`这样定时的效果其实很差，这里详细看一下后面async异步注意项第三点。

第二个我为了时间控制的函数是一个递归函数

```ts
async function getSingleImg(imgUrls: string[], i: number = 0) {
  if (i< imgUrls.length) {
    await sleep(1000)
    i++
    let urls = await getImgUrl(imgUrls[i])
    await urls.forEach((item) => {
      saveImg(item)
      console.log('单页面打印成功')
    })
    imageUrls = await imageUrls.concat(urls)
    await getSingleImg(imgUrls, i++)
  }
}
```

依旧利用了async中的顺序调用，感觉实现的方法很怪，不过好在也是实现了。

# <a id="async-await"></a> async/await
ES2017 标准引入了 async 函数，使得异步操作变得更加方便。

async 函数是什么？一句话，它就是 Generator 函数的语法糖。
引用[async 函数](http://es6.ruanyifeng.com/?search=promise.all&x=0&y=0#docs/async)

async函数对 Generator 函数的改进，体现在以下四点。
1. 内置执行器。

Generator 函数的执行必须靠执行器，所以才有了co模块，而`async`函数自带执行器。也就是说，`async`函数的执行，与普通函数一模一样，只要一行。
```js
asyncReadFile();
```
上面的代码调用了`asyncReadFile`函数，然后它就会自动执行，输出最后结果。这完全不像 Generator 函数，需要调用`next`方法，或者用co模块，才能真正执行，得到最后结果。

2. 更好的语义。

`async`和`await`，比起星号和yield，语义更清楚了。`async`表示函数里有异步操作，`await`表示紧跟在后面的表达式需要等待结果。

3. 更广的适用性。

co模块约定，yield命令后面只能是 Thunk 函数或 Promise 对象，而`async`函数的`await`命令后面，可以是 Promise 对象和原始类型的值（数值、字符串和布尔值，但这时等同于同步操作）。

4. 返回值是 Promise。
async函数的返回值是 Promise 对象，这比 Generator 函数的返回值是 Iterator 对象方便多了。你可以用then方法指定下一步的操作。

进一步说，async函数完全可以看作多个异步操作，包装成的一个 Promise 对象，而await命令就是内部then命令的语法糖。

## <a id="warn-point"></a>使用注意点
第一点，前面已经说过，await命令后面的Promise对象，运行结果可能是rejected，所以最好把await命令放在try...catch代码块中。
```js
async function myFunction() {
  try {
    await somethingThatReturnsAPromise();
  } catch (err) {
    console.log(err);
  }
}

// 另一种写法

async function myFunction() {
  await somethingThatReturnsAPromise()
  .catch(function (err) {
    console.log(err);
  });
}
```

第二点，多个await命令后面的异步操作，如果不存在继发关系，最好让它们同时触发。
```js
let foo = await getFoo();
let bar = await getBar();
```
上面代码中，getFoo和getBar是两个独立的异步操作（即互不依赖），被写成继发关系。这样比较耗时，因为只有getFoo完成以后，才会执行getBar，完全可以让它们同时触发。
```js
// 写法一
let [foo, bar] = await Promise.all([getFoo(), getBar()]);

// 写法二
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
```
上面两种写法，getFoo和getBar都是同时触发，这样就会缩短程序的执行时间。

第三点，await命令只能用在async函数之中，如果用在普通函数，就会报错。(也就是我们需要用到的点。)
```js
async function dbFuc(db) {
  let docs = [{}, {}, {}];

  // 报错
  docs.forEach(function (doc) {
    await db.post(doc);
  });
}
```

上面代码会报错，因为await用在普通函数之中了。但是，如果将forEach方法的参数改成async函数，
```js
function dbFuc(db) { //这里不需要 async
  let docs = [{}, {}, {}];

  // 可能得到错误结果
  docs.forEach(async function (doc) {
    await db.post(doc);
  });
}
```

上面代码可能不会正常工作，原因是这时三个db.post操作将是并发执行，也就是同时执行，而不是继发执行。正确的写法是采用for循环。
```js
async function dbFuc(db) {
  let docs = [{}, {}, {}];

  for (let doc of docs) {
    await db.post(doc);
  }
}
```

如果确实希望多个请求并发执行，可以使用Promise.all方法。当三个请求都会resolved时，下面两种写法效果相同。
```js
async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = await Promise.all(promises);
  console.log(results);
}

// 或者使用下面的写法

async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = [];
  for (let promise of promises) {
    results.push(await promise);
  }
  console.log(results);
}
```

## <a id="error-handle"></a> 错误处理
如果await后面的异步操作出错，那么等同于async函数返回的 Promise 对象被reject。

所以防止出错的方法，也是将其放在try...catch代码块之中。

如果有多个await命令，可以统一放在try...catch结构中。

也可以使用try...catch结构，实现多次重复尝试。例如在遍历中。


