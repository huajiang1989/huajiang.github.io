MongoDB的介绍

**
在说MongoDB数据插入操作之前，我们先来简单了解下它的数据逻辑结构。
MongoDB的逻辑结构是一种层次结构，主要由：文档(document)、集合(collection)、数据库(database)这三部分组成的。
文档(document)：由键/值对构成，像{a:1}；{s:”abc”}等，它是MongoDB核心单元，MongoDB的文档（document），相当于关系数据库中的一行记录。
集合（Collection）：多个文档组成一个集合（collection），相当于关系数据库的表。
数据库（database）：多个集合（collection），逻辑上组织在一起，就是数据库（database）。

一个MongoDB实例支持多个数据库（database）。

安装及引用

安装

npm install mongoose

引用mongoose

var mongoose = require(“mongoose”);

使用mongoose链接数据库

var db = mongoose(“mongodb://user:pass@localhost:port/database”);

示例

var mongoose = require(“mongoose”);
var db = mongoose.connect(“mongodb://127.0.0.1:27017/test”);
db.connection.on(“error”, function (error) {
console.log(“数据库连接失败：” + error);
});
db.connection.on(“open”, function () {
console.log(“——数据库连接成功！——”);
});

MongoDB基础

Schema ： 一种以文件形式存储的数据库模型骨架，不具备数据库的操作能力

Model ： 由Schema发布生成的模型，具有抽象属性和行为的数据库操作对

Entity ： 由Model创建的实体，他的操作也会影响数据库

Schema
Schema —— 一种以文件形式存储的数据库模型骨架，无法直接通往数据库端，也就是说它不具备对数据库的操作能力，仅仅只是数据库模型在程序片段中的一种表现，可以说是数据属性模型(传统意义的表结构)，又或着是“集合”的模型骨架。
定义

var mongoose = require("mongoose")
var TestSchema = new mongoose.Schema({
    name : {type:String},
    age : {type:Number,default:0},
    time : {type:Date,default:Date.now},
    emial : {type:String,default:''}
});

基本属性类型有：字符串、日期型、数值型、布尔型(Boolean)、null、数组、内嵌文档等

Model
Model —— 由Schema构造生成的模型，除了Schema定义的数据库骨架以外，还具有数据库操作的行为，类似于管理数据库属性、行为的类。

var db = mongoose.connect("mongodb://127.0.0.1:27017/test");
// 通过Schema创建Model
var TestModel = db.model("test1", TestSchema);

数据库中的集合名称,当我们对其添加数据时如果test1已经存在，则会保存到其目录下，如果未存在，则会创建test1集合，然后在保存数据。

Entity
Entity —— 由Model创建的实体，使用save方法保存数据，Model和Entity都有能影响数据库的操作，但Model比Entity更具操作性。

var TestEntity = new TestModel({
    name : "Lenka",
    age : 36,
    email : "lenka@qq.com"
});
console.log(TestEntity.name); // Lenka
console.log(TestEntity.age); // 36

**

示例1

**
数据库的连接，Schemal的创建，模型的创建，实体的创建，通过实体保存数据库信息。

var mongoose = require("mongoose");
var db = mongoose.connect("mongodb://127.0.0.1:27017/test");
var TestSchema = new mongoose.Schema({
    name : {type:String},
    age : {type:Number,default:0},
    email : {type:String},
    time : {type:Date,default:Date.now}
});
var TestModel = db.model("test1",TestSchema); //'test'相当于collection
var TestEntity = new TestModel({
    name:'helloworld',
    age:28,
    emial:'helloworld@qq.com'
});
TestEntity.save(function(err,doc){
    if(err){
        console.log("error :" + err);
    } else {
        console.log(doc);
    }
});

这里写图片描述

可以看到数据库新增记录成功

示例2

通过entity、model来完成数据库的增删查改。
// mongoose 链接

var mongoose = require('mongoose');
var db       = mongoose.createConnection('mongodb://127.0.0.1:27017/test');
1
2
1
2
Schema 结构

var mongooseSchema = new mongoose.Schema({
    username : {type : String, default : '匿名用户'},
    title    : {type : String},
    content  : {type : String},
    time     : {type : Date, default: Date.now},
    age      : {type : Number}
});

// 添加 mongoose 实例方法
mongooseSchema.methods.findbyusername = function(username, callback) {
    return this.model('mongoose').find({username: username}, callback);
}
// 添加 mongoose 静态方法，静态方法在Model层就能使用
mongooseSchema.statics.findbytitle = function(title, callback) {
    return this.model('mongoose').find({title: title}, callback);
}

相当于Schemal是一个构造函数，在这里可以添加方法findbyuseraname()和findbytitle

model结构

var mongooseModel = db.model('mongoose', mongooseSchema);
1
1
我理解为在test数据库下面创建了mongoose collection的连接

Entity结构

//增加操作
var doc = {username : 'emtity_demo_username', title : 'emtity_demo_title', content : 'emtity_demo_content'};
var mongooseEntity = new mongooseModel(doc);
mongooseEntity.save(function(error) {
    if(error) {
        console.log(error);
    } else {
        console.log('saved OK!');
    }
    // 关闭数据库链接
    db.close();
});

//查询操作
var mongooseEntity = new mongooseModel({});
mongooseEntity.findbyusername('model_demo_username', function(error, result){
    if(error) {
        console.log(error);
    } else {
        console.log(result);
    }
    //关闭数据库链接
    db.close();
});

基于model的增加、修改与查询

// 增加记录 基于model操作
var doc = {username : 'model_demo_username', title : 'model_demo_title', content : 'model_demo_content'};
mongooseModel.create(doc, function(error){
    if(error) {
        console.log(error);
    } else {
        console.log('save ok');
    }
    // 关闭数据库链接
    db.close();
});

//修改记录
//mongooseModel.update(conditions, update, options, callback);
var conditions = {username : 'model_demo_username'};
var update     = {$set : {age : 27, title : 'model_demo_title_update'}};
var options    = {upsert : true};
mongooseModel.update(conditions, update, options, function(error){
    if(error) {
        console.log(error);
    } else {
        console.log('update ok!');
    }
    //关闭数据库链接
    db.close();
});

// mongoose find
var criteria = {title : 'emtity_demo_title'}; // 查询条件
var fields   = {title : 1, content : 1, time : 1}; // 待返回的字段
var options  = {};
mongooseModel.find(criteria, fields, options, function(error, result){
    if(error) {
        console.log(error);
    } else {
        console.log(result);
    }
    //关闭数据库链接
    db.close();
});

// 删除记录
var conditions = {username: 'emtity_demo_username'};
mongooseModel.remove(conditions, function(error){
    if(error) {
        console.log(error);
    } else {
        console.log('delete ok!');
    }

    //关闭数据库链接
    db.close();
});

基于实例方法的查询

前面说到findbyusername()在schemal层定义，相当于实例化，然后继承findbyusername()方法

var mongooseEntity = new mongooseModel({});
mongooseEntity.findbyusername('model_demo_username', function(error, result){
    if(error) {
        console.log(error);
    } else {
        console.log(result);
    }
    //关闭数据库链接
    db.close();
});

基于静态方法的查询

静态方法可以直接在模型层调用

// 基于静态方法的查询
mongooseModel.findbytitle('emtity_demo_title', function(error, result){
    if(error) {
        console.log(error);
    } else {
        console.log(result);
    }
    //关闭数据库链接
    db.close();
});