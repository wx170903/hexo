---
title: mongodb用法初识
date: 2017-10-29 14:18:14
categories: mongodb
tags: 
- node
- mongodb
---
# <a id="toc-mongodb"></a> mongoDB
<!-- more -->
## <a id="toc-what-mongodb"></a> 什么是mongodb

[中文官网](http://www.mongodb.org.cn/)首页是这么介绍的:mongoDB基于分布式文件储存的数据库高性能、可拓展、易部署、易使用，存储数据非常方便

### [NoSQL简介](http://www.mongodb.org.cn/tutorial/2.html)

NoSQL(NoSQL = Not Only SQL )，意即"不仅仅是SQL"。
在现代的计算系统上每天网络上都会产生庞大的数据量。

这些数据有很大一部分是由关系数据库管理系统（RDMBSs）来处理。 

1970年 E.F.Codd's提出的关系模型的论文 "A relational model of data for large shared data banks"，这使得数据建模和应用程序编程更加简单。

通过应用实践证明，关系模型是非常适合于客户服务器编程，远远超出预期的利益，今天它是结构化数据存储在网络和商务应用的主导技术。

NoSQL 是一项全新的数据库革命性运动，早期就有人提出，发展至2009年趋势越发高涨。NoSQL的拥护者们提倡运用非关系型的数据存储，相对于铺天盖地的关系型数据库运用，这一概念无疑是一种全新的思维的注入。

### [MongoDB 简介](http://www.mongodb.org.cn/tutorial/3.html)
#### 什么是MongoDB ?
MongoDB 是由C++语言编写的，是一个基于分布式文件存储的开源数据库系统。

在高负载的情况下，添加更多的节点，可以保证服务器性能。

MongoDB 旨在为WEB应用提供可扩展的高性能数据存储解决方案。
MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。

MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。

```json
{
  name: "hdd",
  age: 22,
}
```
#### 主要特点
* MongoDB的提供了一个面向文档存储，操作起来比较简单和容易。
* 你可以在MongoDB记录中设置任何属性的索引 (如：FirstName="Sameer",Address="8 Gandhi Road")来实现更快的排序。
* 你可以通过本地或者网络创建数据镜像，这使得MongoDB有更强的扩展性。
* 如果负载的增加（需要更多的存储空间和更强的处理能力） ，它可以分布在计算机网络中的其他节点上这就是所谓的分片。
* Mongo支持丰富的查询表达式。查询指令使用JSON形式的标记，可轻易查询文档中内嵌的对象及数组。
* MongoDb 使用update()命令可以实现替换完成的文档（数据）或者一些指定的数据字段 。
* Mongodb中的Map/reduce主要是用来对数据进行批量处理和聚合操作。
* Map和Reduce。Map函数调用emit(key,value)遍历集合中所有的记录，将key与value传给Reduce函数进行处理。
* Map函数和Reduce函数是使用Javascript编写的，并可以通过db.runCommand或mapreduce命令来执行MapReduce操作。
* GridFS是MongoDB中的一个内置功能，可以用于存放大量小文件。
* MongoDB允许在服务端执行脚本，可以用Javascript编写某个函数，直接在服务端执行，也可以把函数的定义存储在服务端，下次直接调用即可。
* MongoDB支持各种编程语言:RUBY，PYTHON，JAVA，C++，PHP，C#等多种语言。
* MongoDB安装简单。

## <a id="toc-install-mongoDB"></a> [安装mongoDB](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/)
### Linux平台Ubuntu安装MongoDB社区版

#### 第一步：导入包管理系统使用的公钥

Ubuntu软件包管理工具（即dpkg和apt）通过要求分销商使用GPG密钥对软件包进行签名来确保软件包的一致性和真实性。发出以下命令导入 MongoDB公共GPG密钥：

```sh
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
```

#### 第二步：为MongoDB创建一个列表文件

/etc/apt/sources.list.d/mongodb-org-3.4.list使用适合您的Ubuntu版本的命令创建列表文件：

Ubuntu
```sh
echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```

#### 第三步：重新加载本地包数据库

发出以下命令重新加载本地软件包数据库：

```sh
sudo apt-get update
```

#### 第四步：安装MongoDB包

**安装最新的稳定版本的MongoDB**

发出以下命令：

```sh
sudo apt-get install -y mongodb-org
```

## <a id="toc-run-mongodb"></a> 运行MongoDB社区版

#### 第一步：启动MongoDB

发出以下命令开始mongod：

```sh
sudo service mongod start
```

#### 第二步：验证MongoDB已经成功启动


验证MongoDB已经成功启动

mongod通过检查日志文件的内容进行/var/log/mongodb/mongod.log 读取，验证进程是否成功启动

```log
[initandlisten] waiting for connections on port <port>
```

其中<port>被配置为在该端口/etc/mongod.conf，27017默认情况下。

#### 第三步：停止MongoDB

根据需要，您可以mongod通过发出以下命令来停止进程：

```sh
sudo service mongod stop
```
#### 第四步：重新启动MongoDB

发出以下命令重新启动mongod：

```sh
sudo service mongod restart
```

#### 第五步：开始使用MongoDB
为了帮助您开始使用MongoDB，MongoDB提供了各种驱动程序版本中的[入门指南](https://docs.mongodb.com/manual/tutorial/getting-started/#getting-started)。有关可用版本，请参阅 [入门](https://docs.mongodb.com/manual/tutorial/getting-started/#getting-started)。

在生产环境中部署MongoDB之前，请考虑 [生产备注文档](https://docs.mongodb.com/manual/administration/production-notes/)。

稍后，要停止MongoDB，请`Control+C`在mongod实例运行的终端上 按。

## <a id="toc-node-mongodb"></a> [NodeJS中MongoDB驱动](http://www.mongodb.org.cn/drivers/5.html)

虽然说在NodeJS下连接MongoDB用Mongoose的较多，但作为其基础的mongodb库了解一下还是很有必要的。

### 安装
一如既往的通过npm安装，命令
```sh
npm install mongodb
```
### 连接数据库
通过MongoClient.connect连接数据库，在回调中会返回db对象以供之后使用。
```js
var MongoClient = require('mongodb').MongoClient;
var url = 'mongodb://localhost:27017/dbname';
MongoClient.connect(url, function(err, db) {
	if(err){
		console.error(err);
		return;
	}else{
		console.log("Connected correctly to server");
		db.close();
	}
});
```

### 获得Collection
调用db对象的collection获得collection

```js
var collection = db.collection('collectionName');
```

### 添加记录
调用collection的insert|insertMany方法添加记录。

```js
collection.insert[|insertMangy]({name:"myName",age:"myAge"},function(err,result){
	if(err){
		console.error(err);
	}else{
		console.log("insert result:");
		console.log(result);
	}
})
```

### 更新记录
调用collection的updateOne方法更新单个记录。

```js
collection.updateOne({ a : 2 }, { $set: { b : 1 } }, function(err, result) {
	if(err){
		console.error(err);
	}else{
		console.log("update result:");
		console.log(result);
	}
});
```

### 删除记录
调用collection的deleteOne方法更新单个记录。
```js
collection.deleteOne({ a : 3 }, function(err, result) {
    if(err){
        console.error(err);
    }else{
        console.log("delete result:");
        console.log(result);
    }
  });
```
### 查询记录
调用collection的find方法查找记录,find方法的参数为查找条件。
```js
collection.find({}).toArray(function(err, docs) {
    if(err){
        console.error(err);
    }else{
        console.log("find result:");
        console.log(result);
    }
  });
```

仅仅写了基础的CRUD,详情参考 http://mongodb.github.io/node-mongodb-native/2.0














