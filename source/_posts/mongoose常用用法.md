---
title: mongoose常用用法
date: 2017-11-02 15:28:37
categories: mongodb
tags: 
- node
- mongodb
- mongoose
---
[参考链接](https://www.villainhr.com/page/2016/05/11/%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAmongoose)

讲的非常好，推荐！
<!-- more -->
# <a id="toc-meet-mongoose"></a> 初识mongoose
## <a id="toc-install-mongoose"></a> install mongoose

安装mongodb可以看一下另一篇文章[mongodb用法初识](https://hddhyq.github.io/2017/10/29/mongodb%E7%94%A8%E6%B3%95%E5%88%9D%E8%AF%86/)
这里我们开启mongodb服务
```sh
sudo service mongod start
```

接下来下载安装mongoose
```sh
npm install mongoose --save
```

## <a id="toc-connect-mongoose"></a> connect mongoose
下载好之后，我们需要连接database.在js文件写入:
```js
'use strict';

const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/test', {
  useMongoClient: true // 新版本需要使用解决旧版本升级问题，可以查阅官网
});
const db = mongoose.connection;

db.on('error', () => {
  console.log('连接失败')
});
con.once('open', () => {
  console.log('连接成功')
})
```
如果不出什么问题，我们的第一次连接会在命令行上显示**连接成功**。

# <a id="toc-understand-mongoose"></a>  Mongoose的基本概念
* **Schema:** 相当于一个数据库的模板. Model可以通过mongoose.model 集成其基本属性内容. 当然也可以选择不继承.
* **Model:** 基本文档数据的父类,通过集成Schema定义的基本方法和属性得到相关的内容.
* **instance:** 这就是实实在在的数据了. 通过 new Model()初始化得到.

![mongoose](http://7xpsmd.com1.z0.glb.clouddn.com/16-4-29/24287224.jpg)

demo:
```js
'use strict';

const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/test', {
   useMongoClient: true
});
const db = mongoose.connection;
db.on('error', console.error.bind(console, '连接数据库失败'));
db.once('open',()=>{
    //定义一个schema
    let Schema = mongoose.Schema({
        category: String,
        name: String
    });
    Schema.methods.eat = function(){
        console.log("I've eatten one "+this.name);
    }
    //继承一个schema
    let Model = mongoose.model("fruit",Schema);
    //生成一个document
    let apple = new Model({
        category:'apple',
        name:'apple'
    });
    //存放数据
    apple.save((err,apple)=>{
        if(err) return console.log(err);
        apple.eat();
        //查找数据
        Model.find({name:'apple'},(err,data)=>{
            console.log(data);
        })
    });
})
```

# <a id="toc-more-mongoose"></a> 深入浅出mongoose
从三个基本概念深入展开
## <a id="toc-Schema"></a> Schema
这实际上是,mongoose中最重要的一个theroy. schema 是用来定义 documents的基本字段和集合的. 在mongoose中,提供了Schema的类。 我们可以通过实例化他, 来实现创建 Schema的效果. 而不需要每次调用 mongoose.Schema()这个丑陋的API.

```js
// from mongoose author
var mongoose = require('mongoose');
var Schema = mongoose.Schema;

var blogSchema = new Schema({
  title:  String,
  author: String,
  body:   String,
  comments: [{ body: String, date: Date }],
  date: { type: Date, default: Date.now },
  hidden: Boolean,
  meta: {
    votes: Number,
    favs:  Number
  }
});
```
Schema 之所以能够定义documents, 是因为他可以限制你输入的字段及其类型. mongoose支持的基本类型有:
* String
* Number
* Date
* Buffer
* Boolean
* Mixed
* ObjectId
* Array

其中, Mixed和ObjectId是mongoose中特有的,ObjectId实际上就是_**id**的一个映射. 同样,mongoose里面有着和所有大众数据库一样的东西. **索引** – indexs

### mongoose 设置索引
这里设置索引分两种,一种设在Schema filed, 另外一种设在 Schema.index 里.
```js
var animalSchema = new Schema({
  name: String,
  type: String,
  tags: { type: [String], index: true } 
});
// 在Schema.index中设置.
animalSchema.index({ name: 1, type: -1 }); 

// 1 表示正序, -1 表示逆序
```
实际上,两者效果是一样的. 看每个人的喜好了. 不过推荐直接在Schema level中设置, 这样分开能够增加可读性. 不过,官方给出了一个建议, 因为在创建字段时, 数据库会自动根据自动排序(ensureIndex)排序. 有可能严重拖慢查询或者创建速度,所以一般而言,我们需要将该option 关闭.
```js
mongoose.connect('mongodb://user:pass@localhost:port/database', { config: { autoIndex: false } });  //真心推荐
// or  
mongoose.createConnection('mongodb://user:pass@localhost:port/database', { config: { autoIndex: false } });  //不推荐
// or
animalSchema.set('autoIndex', false);  //推荐
// or
new Schema({..}, { autoIndex: false }); //懒癌不推荐
```
另外, Schema 另一大特色就是其methods. 我们可以通过定义其methods,访问到实际上的所有内容.

### 定义Schema.methods
使用的方法很简单,就是使用 .methods即可.
```js
// 定义一个schema
var freshSchema = new Schema({ name: String, type: String });

// 添加一个fn. 
animalSchema.methods.findSimilarTypes = function (cb) {
  //这里的this指的是具体document上的this
  return this.model('Animal').find({ type: this.type }, cb);
}
// 实际上,我们可以通过schema绑定上,数据库操作的所有方法.
// 该method实际上是绑定在 实例的 doc
```
定义完methods和property之后, 就到了生成Model的阶段了.
### 实例Model
这里同样很简单,只需要 mongoose.model() 即可.
```js
// 生成,model 类. 实际上就相当于我们的一个collection
var Animal = mongoose.model('Animal', animalSchema);
var dog = new Animal({ type: 'dog' });
```
但是, 这里有个问题. 我们在Schema.methods.fn 上定义的方法,只能在 new Model() 得到的实例中才能访问. 那如果我们想,直接在Model上调用 相关的查询或者删除呢？

### 绑定Model方法
同样很简单,使用 statics 即可.
```js
// 给model添加一个findByName方法
animalSchema.statics.findByName = function (name, cb) {
  // 这里的this 指的就是Model
  return this.find({ name: new RegExp(name, 'i') }, cb);
}

var Animal = mongoose.model('Animal', animalSchema);
Animal.findByName('fido', function (err, animals) {
  console.log(animals);
});
```
Mongoose 还有一个super featrue-- virtual property 该属性是直接设置在Schema上的. 但是,需要注意的是,VR 并不会真正的存放在db中. 他只是一个提取数据的方法.
```js
//schema基本内容
var personSchema = new Schema({
  name: {
    first: String,
    last: String
  }
});

// 生成Model
var Person = mongoose.model('Person', personSchema);

//现在我们有个需求,即,需要将first和last结合输出.
//一种方法是,使用methods来实现
//schema 添加方法
personSchema.methods.getName = function(){
    return this.first+" "+this.last;
}

// 生成一个doc
var bad = new Person({
    name: { first: 'jimmy', last: 'Gay' }
});

//调用
bad.getName();
```

但是,像这样,仅仅这是为了获取一个属性, 实际上完全可以使用虚拟属性来实现.
```js
// schema 添加虚拟属性
personSchema.virtual('fullName').get(function(){
    return this.first+" "+this.last;
})
// 调用
bad.fullName;  // 和上面的方法的结果是完全一致的
```
而且,经过测试, 使用fn实现的返回,比VR 要慢几十倍. 一下是测试结果:
```js
console.time(1);
    bad.getName();
    console.timeEnd(1);
    console.time(2);
    bad.fullName;
    console.timeEnd(2);
    
    //结果为:
    1: 4.323ms; // method
    2: 0.253ms  // VR
```

最后再补充一下,Schema中初始化的相关参数.

**Schema参数** 在 `new Schema([options])` 中,我们需要设置一些相关的参数.
* **safe:** 用来设置安全模式. 实际上,就是定义入库时数据的写入限制. 比如写入时限等.
```js
// 使用安全模式. 表示在写入操作时,如果发生错误,也需要返回信息.
var safe = true;
new Schema({ .. }, { safe: safe });

// 自定义安全模式. w为写入的大小范围. wtimeout设置写入时限. 如果超出10s则返回error
var safe = { w: "majority", wtimeout: 10000 };
new Schema({ .. }, { safe: safe });
```

* **toObject:** 用来表示在提取数据的时候, 把documents 内容转化为Object内容输出. 一般而言只需要设置getters为true即可.
```
schema.set('toObject', { getters: true });
var M = mongoose.model('Person', schema);
var m = new M({ name: 'Max Headroom' });
//实际打印出来的就是一个Object类型
console.log(m); // { _id: 504e0cd7dd992d9be2f20b6f, name: 'Max Headroom is my name' }
```

* **toJSON:** 该是和toObject一样的使用. 通常用来把 documents 转化为Object. 但是, 需要显示使用toJSON()方法,否则,不会起作用。实际上,没什么卵用…
看一下总结图谱: 
![](http://7xpsmd.com1.z0.glb.clouddn.com/16-4-29/98685367.jpg)

看完schema之后,我们需要了解一下 model的内容.

## <a id="toc-Model"></a> Model
实际上,Model才是操作数据库最直接的一块内容. 我们所有的CRUD就是围绕着Model 展开的. ok. 还记得,我们是怎样创建一个model的吗？
```js
//from mongoosejs
var schema = new mongoose.Schema({ name: 'string', size: 'string' });
var Tank = mongoose.model('Tank', schema);
```

这里,我们一定要搞清楚一个东西. 实际上, mongoose.model里面定义的第一个参数,比如’Tank’, 并不是数据库中的, collection. 他只是collection的单数形式, 实际上在db中的collection是’Tanks’.

ok, 我们现在已经有了一个基本的Model. 但并没有什么x用. 接下来, 正式到了 dry goods(干货) 时间.

**model 的子文档操作** 这个就厉害了. 本来mongodb是没有关系的. 但是, mongoose提供了children字段. 让我们能够轻松的在表间建立关系. 现在,我们来创建一个子域:
```js
var childSchema = new Schema({ name: 'string' });

var parentSchema = new Schema({
  children: [childSchema]   //指明sub-doc的schema
});
//在创建中指明doc
var Parent = mongoose.model('Parent', parentSchema);
var parent = new Parent({ children: [{ name: 'Matt' }, { name: 'Sarah' }] })
parent.children[0].name = 'Matthew';
parent.save(callback);
```

现在, 我们就已经创建了3个table. 一个parent 包含了 两个child 另外,如果我们想要查询指定的doc。 则可以使用 id()方法
```js
var doc = parent.children.id(id);
```

子文档的CRUD, 实际上就是数组的操作, 比如push,unshift,remove,pop,shift等
```js
parent.children.push({ name: 'Liesl' });
```

mongoose还给移除提供了另外一个方法–remove:
```js
var doc = parent.children.id(id).remove();
```

如果你忘记添加子文档的话，可以在外围添加, 但是字段必须在Schema中指定
```js
var newdoc = parent.children.create({ name: 'Aaron' });
```

### model的CRUD操作
**model的创建** 关于model的创建,有两种方法, 一种是使用实例创建,另外一种是使用Model类创建.
```js
var Tank = mongoose.model('Tank', yourSchema);

var small = new Tank({ size: 'small' });
//使用实例创建
small.save(function (err) {
  if (err) return handleError(err);
  // saved!
})

//使用Model类创建
Tank.create({ size: 'small' }, function (err, small) {
  if (err) return handleError(err);
  // saved!
})
```

上面已经完美的介绍创建的方法了. 另外,官方给出一个提醒: 由于mongoose, 会自身连接数据库并断开. 如果你手动连接, 则创建model的方式需要改变.

```js
// 自己并没有打开连接:
// 注意,这里只是连接,并没有创建connection
mongoose.connect('mongodb://localhost:27017/test'); 

// 手动创建连接:
var connection = mongoose.createConnection('mongodb://localhost:27017/test');
var Tank = connection.model('Tank', yourSchema); 
```

然后, 下面的API还是一样的. 实际上,我们一般常用的写法为:
```js
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
//设置连接位置
mongoose.connect('mongodb://localhost:27017/test');
var schema = new mongoose.Schema({ name: 'string', size: 'string' });
var Tank = mongoose.model('Tank', schema);
var small = new Tank({ size: 'small' });
//使用实例创建
small.save(function (err) {
  if (err) return handleError(err);
  console.log('创建成功');
})
```

这样,就不用自己去手动管连接的问题了. 如果你,在后面想手动添加字段的话,可以使用.set方法.
```js
// 一个key/valye
doc.set(path, value)
//很多key/value
doc.set({
    path  : value,
    path2 : {
       path  : value
    }
})
```

**model**的**query** model的查找主要提供了以下的API,给我们进行操作. **find**, **findById**, **findOne**, or **where** 在mongodb中, query返回的数据格式一般都是为JSON的. 这点需要注意.

事实上,在mongoose中,query数据 提供了两种方式.

* **callback:** 使用回调函数, 即, query会立即执行,然后返回到回调函数中.
```js
Person.findOne({ 'name.last': 'Ghost' }, 'name occupation', function (err, person) {
  if (err) return handleError(err);
 // get data
})
```

* **query:** 使用查询方法,返回的对象. 该对象是一个Promise, 所以可以使用 chain 进行调用.最后必须使用exec(cb)传入回调进行处理. cb 是一个套路, 第一个参数永远是err. 第二个就是返回的数据。
```js
Person.
  find({
    occupation: /host/,
    'name.last': 'Ghost',
    age: { $gt: 17, $lt: 66 },
    likes: { $in: ['vaporizing', 'talking'] }
  }).
  limit(10).
  sort({ occupation: -1 }).
  select({ name: 1, occupation: 1 }).
  exec(callback);
  
//如果没有查询到,则返回[] (空数组)
// 如果你使用findOne, 没有的话则会返回 null
```

童鞋, 你觉得我会推荐哪种呢？

上面4个API, 3个使用方式都是一样的, 另外一个不同的是where. 他一样是用来进行query. 只是,写法和find系列略有不同

**where简介** where的API为: `Model.where(path, [val])` path实际上就是字段, 第二个参数.val表示可以用来指定,path = val的数据内容, 你也可以不写, 交给后面进行筛选. 看一下对比demo吧:
```js
User.find({age: {$gte: 21, $lte: 65}}, callback);
//等价于:
User.where('age').gte(21).lte(65).exec(callback);
```

从上面的query中,我们可以看到有许多fn, 比如gte,lte,$gte,$lte. 这些是db提供给我们用来查询的快捷函数. 我们可以参考, mongoose给的参考: [query Helper](http://mongoosejs.com/docs/api.html#query-js) fn 这里,我们简要的了解下,基本的快捷函数.

|name|effect|
|----|------|
|select|添加需要显示的字段,需要的字段在字段后加上:1,不需要的加上0;<br/>query.select({ a: 1, b: 0 }); //显示a字段, 隐藏b字段<br/>不能和distinct方法一起使用|
|distinct|用来筛选不重复的值或者字段<br/>distinct(field). //筛选指定不重复字段的数据|
|$lt,$lte,$gt,$gte|分别对应: <,<=,>,>=. 该字段是用在condition中的.如果,你想要链式调用,则需要使用<br/>lt,lte,ge,gte.<br/>eg:<br/> model.find({num:{$gt:12}},cb)<br/>model.where(‘num’).gt(12).exec(cb)|
|$in|查询包含键值的文档,<br/>model.find({name:{$in:[“jimmy”,“sam”]}}) //相当于匹配 jimmy或者sam|
|$nin|返回不匹配查询条件文档,都是指定数组类型<br/>model.find({name:{$nin:[“jimmy”,“sam”]}})|
|$ne|表示不包含指定值<br/>model.find({name:{$ne:“sam”}})|
|$or|表示或查询<br/>model.find({$or:[{ color: ‘red’ }, { status: ‘emergency’ }]})|
|$exits|表示键值是否存在;<br/>model.find({name:{$exits:true}})|
|$all|通常用来匹配数组里面的键值,匹配多个值(同时具有)<br/>$all:[“apple”,“banana”,“peach”]}|
|$size|用来查询数组的长度值<br/>model.find({name:{$size:3}}); 匹配name的数组长度为3|
|$slice|用来获取数组字段的内容:<br/>query.slice(‘comments’, 5)|

ok~ 这上面就是比较常用的快捷函数. 另外还有一些游标集合的处理方法: 常用的就3个, limit,skip,sort.

* **limit:** 用来获取限定长度的内容.
```js
query.limit(20); //只返回前20个内容
```

* **skip:** 返回，跳过指定doc后的值.
```js
query.skip(2);
```

* **sort:** 用来设置根据指定字段排序. 可以设置为1:升序, -1:降序.
```js
query.sort({name:1,age:-1});
```

实际上, 关于query,我们需要了解的也就差不多了.

我们接下来,来看一下remove. **mongoose remove 操作**
```js
Model.find().remove({ name: 'Anne Murray' }).remove(fn);
//或者直接添加回调
Model.find().remove({ name: 'Anne Murray' },cb)
```

另外,我们可以直接在Model上调用. 因为remove也是Schema定义的statics方法. 而且, remove返回一个Promise对象
```js
product.remove().then(function (product) {
   ...
});
//或者直接传入回调
Tank.remove({ size: 'large' }, function (err) {
  if (err) return handleError(err);
  // removed!
});
```

最后,我们再看一下 update. 然后mongoose就基本结束了 **update操作:** 这里,我只说一下API就好. 因为update 比起上面来说,还是比较简单的. `Model.update(conditions, doc, [options], [callback])`

* **conditions:** 就是query. 通过query获取到指定doc
* **doc:** 就是用来替换doc内容的值.
* **options:** 
  * safe (boolean) 是否开启安全模式 (default for true)
  * upsert (boolean) 如果没有匹配到内容,是否自动创建 ( default for false)
  * multi (boolean) 如果有多个doc,匹配到,是否一起更改 ( default for false)
  * strict (boolean) 使用严格模式(default for false)
  * overwrite (boolean) 匹配到指定doc,是否覆盖 (default for false)
  * unValidators (boolean): 表示是否用来启用验证. 实际上,你首先需要写一个验证. 关于如果书写,验证大家可以参考下文, validate篇(default for false)
```js
Model.update({age:18}, { $set: { name: 'jason borne' }}, {multi:true}, function (err, raw) {
  if (err) return handleError(err);
  console.log('raw 就是mongodb返回的更改状态的falg ', raw);
  //比如: { ok: 1, nModified: 2, n: 2 }
});
```
其中的$set是,用来指明更新的字段.另外,mongoose还提供了一个:findByIdAndUpdate(id,doc[,options][,callback]); 方法. 关于mongoose的更新helper 函数. 童鞋们可以参考一下.mongoose官方文档.

## <a id="toc-validation"></a> validation
说完了,mongoose的body之后. 我们接着来看一下,官方给mongoose穿上的漂亮的衣服. 其中一件,比较吸引人的是–validation. 在你save数据之前, 你可以对数据进行一些列的validation. 来防止某天你傻不拉几的把数据完整性给破坏了. mongoose贴心的提供了几个built-in的验证函数.
* **required:** 表示必填字段.
```js
new Schema({
 name: {
    type:String,
    required:[true,"name 是必须的"] //第二个参数是错误提示信息
 }
})
```

* **min,max:** 用来给Number类型的数据设置限制.
```js
 var breakfastSchema = new Schema({
      eggs: {
        type: Number,
        min: [6, 'Too few eggs'],
        max: 12
      }
});
```

* **enum,match,maxlength,minlength:** 这些验证是给string类型的. enum 就是枚举,表示该属性值,只能出席那那些. match是用来匹配正则表达式的. maxlength&minlength 显示字符串的长度.
```js
new Schema({
    drink: {
        type: String,
        enum: ['Coffee', 'Tea']
      },
     food:{
        type: String,
        match:/^a/,
        maxlength:12,
        minlength:6
    }
})
```

mongoose提供的helper fn就是这几种, 如果你想定制化验证. 可以使用custom validation.
```js
new Schema({
 phone: {
        type: String,
        validate: {
          validator: function(data) {
            return /\d{3}-\d{3}-\d{4}/.test(data);
          },
          message: '{VALUE} is not a valid phone number!' //VALUE代表phone存放的值
        },
        required: [true, 'User phone number required']
      }
})
```

另外,还可以额外添加验证.
```js
var toySchema = new Schema({
    color: String,
    name: String
  });

  var validator = function (value) {
    return /blue|green|white|red|orange|periwinkle/i.test(value);
  };
  toySchema.path('color').validate(validator,
    'Color `{VALUE}` not valid', 'Invalid color');
```
现在,我们已经设置了validation. 但是你不启用,一样没有什么卵用. 实际上, 我们也可以把validation当做一个中间件使用. mongoose 提供了两种调用方式. 一种是内置调用, 当你使用.save方法时,他会首先执行一次存储方法.
```js
cat.save(function(error) {
//自动执行,validation
});
```
另外一种是,手动验证–指定validate方法.
```js
//上面已经设置好user的字段内容.
  user.validate(function(error) {
    //error 就是验证不通过返回的错误信息
     assert.equal(error.errors['phone'].message,
        '555.0123 is not a valid phone number!');
    });
});
```
事实上, 在validate时, 错误的返回信息有以下4个字段: kind, path, value, and message;

* **kind:** 用来表示验证设置的第二个参数. 一般不用
* **path:** 就是字段名
* **value:** 你设置的错误内容
* **message:** 提示错误信息 看一个整体demo吧:
```js
var validator = function (value) {
    return /blue|green|white|red|orange|periwinkle/i.test(value);
  };
  Toy.schema.path('color').validate(validator,
    'Color `{VALUE}` not valid', 'Invalid color'); //设置了message && kind

  var toy = new Toy({ color: 'grease'});

  toy.save(function (err) {
    // err is our ValidationError object
    // err.errors.color is a ValidatorError object
    assert.equal(err.errors.color.message, 'Color `grease` not valid'); //返回message
    assert.equal(err.errors.color.kind, 'Invalid color');
    assert.equal(err.errors.color.path, 'color');
    assert.equal(err.errors.color.value, 'grease');
    assert.equal(err.name, 'ValidationError');
    //访问color 也可以直接上 errors["color"]进行访问.
  });
```

在Model.update那一节有个参数–runValidators. 还没有详细说. 这里, 展开一下. 实际上, validate一般只会应用在save上, 如果你想在update使用的话, 需要额外的trick，而runValidators就是这个trick.
```js
var opts = { runValidators: true };
    Test.update({}, update, opts, function(error) {  //额外开启runValidators的验证
      // There will never be a validation error here
    });
```
我们来看一下基本总结吧: 
![](http://7xpsmd.com1.z0.glb.clouddn.com/16-4-30/64475521.jpg)

## <a id="toc-population"></a> population
originally, mongodb 本来就是一门非关系型数据库。 但有时候,我们又需要联合其他的table进行数据查找。 这时候, 一般的做法就是实现两次查询，效率我就呵呵了.
此时, mongoose 说了一句: 麻麻, 我已经都把脏活帮你做好了. 感动~ 有木有~ 这就是mongoose提供的 population. 用来连接多表数据查询. 一般而言, 我们只要提供某一个collection的_id , 就可以实现完美的联合查询. population 用到的关键字是: ref 用来指明外联的数据库的名字. 一般,我们需要在schema中就定义好.
```js
var mongoose = require('mongoose')
  , Schema = mongoose.Schema
  
var personSchema = Schema({
  _id     : Number,
  name    : String,
  age     : Number,
  stories : [{ type: Schema.Types.ObjectId, ref: 'Story' }]
});

var storySchema = Schema({
  _creator : { type: Schema.Types.ObjectId, ref: 'Person' },
  title    : String
});
```
这里就指明了, 外联数据表的应用关系 personSchema <stories> By _id => Story storySchema <_creator> By _id => Person 实际上, 就是通过_id的相互索引即可. 这里需要说明的是, _id 应该是某个具体model的id.

我们来看一下, 接下来应该如何利用population实现外联查询.
```js
const sam = new Person({
    name: 'sam',
    _id: 1,
    age: 18,
    stories: []
});
sam.save((err,sam)=>{
    if(err) return err;
    let story = new Story({
        _creator:sam._id,
        title:"喜剧之王"
    })
})

Story.findOne({title:"喜剧之王"}).populate('_creator').exec((err,story)=>{
    if(err)console.log(err);
    console.log(story._creator.name);
})
//使用populate来指定,外联查询的字段, 而且该值必须是_id才行
```
现在es6时代的来临, generator , async/await 盛行. 也带来另外一种书写方式–middleware. 在mongoose也有这个hook, 给我们使用. (说实话, 有点像AOP)

## <a id="toc-mongoose-middleware"></a> mongoose && middleware
mongoose里的中间件,有两个, 一个是pre, 一个是post.
* **pre:** 在指定方法执行之前绑定。 中间件的状态分为 parallel和series.
* **post:** 相当于事件监听的绑定

这里需要说明一下, 中间件一般仅仅只能限于在几个方法中使用. (但感觉就已经是全部了)
* doc 方法上: init,validate,save,remove;
* model方法上: count,find,findOne,findOneAndRemove,findOneAndUpdate,update

### pre
我们来看一下,pre中间件是如何绑定的.
```js
// series执行, 串行
var schema = new Schema(..);
schema.pre('save', function(next) {
 // exe some operations
 this.model.
  next();  //  这里的next()相当于间执行权给下一个pre
});
```

在你调用 model.save方法时, 他会自动执行pre. 如果你想并行执行中间件, 可以设置为:
```js
schema.pre('save', true, function(next, done) {
  // 并行执行下一个中间件
  next();
});
```

### post
相当于绑定啦~ post会在指定事件后触发
```js
schema.post('save', function(doc) {
 //在save完成后 触发.
  console.log('%s has been saved', doc._id);
});
```

当save方法调用时, 便会触发post绑定的save事件. 如果你绑定了多个post。 则需要指定一下中间件顺序.
```js
schema.post('save', function(doc, next) {
  setTimeout(function() {
    console.log('post1');
    next();
  }, 10);
});

schema.post('save', function(doc, next) {
  console.log('post2');
  next();
});
```

实际上,post触发的时间为:
```js
var schema = new Schema(..);
schema.post('save', function (doc) {
  console.log('this fired after a document was saved');
});

var Model = mongoose.model('Model', schema);

var m = new Model(..);
m.save(function (err) {
  console.log('this fires after the `post` hook');
});
```

另外,在post和find中, 是不能直接修改doc上的属性的. 即,像下面一样的,没有效果
```js
articleSchema.post('find',function(docs){
    docs[1].date = 1
})
docs[1].date 的值还是不变
```
不过可以使用虚拟属性,进行操作.






