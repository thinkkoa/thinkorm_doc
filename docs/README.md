# ThinkORM

[![npm version](https://badge.fury.io/js/thinkorm.svg)](https://badge.fury.io/js/thinkorm)
[![Build Status](https://travis-ci.org/thinkkoa/thinkorm.svg?branch=master)](https://travis-ci.org/thinkkoa/thinkorm)
[![Dependency Status](https://david-dm.org/thinkkoa/thinkorm.svg)](https://david-dm.org/thinkkoa/thinkorm)

* thinkorm@4.0 Use Typescript's decorator!!

A flexible, lightweight and powerful Object-Relational Mapper for Node.js.

ThinkORM是一个可扩展轻量级的功能丰富的ORM，运行在Node.js环境，已经支持Typescript。

ThinkORM试图用一种抽象的DSL语言，尽量保持各种数据库书写语法一致，用户专注于数据操作逻辑而非具体的数据存储类型，达到快速开发和移植的目的。

## 特性

1. 支持Typescript (4.x版本)

2. 基于Knex.js实现,支持 MySQL, PostgreSQL, MariaDB, SQLite, MS SQL Server, Oracle等常见数据库. 

3. 抽象的面向对象式SQL操作语言,保持各种数据库书写语法一致,方便开发和项目迁移

4. 支持schema定义数据结构,支持严格的类型检查;支持数据结构迁移到数据库,通过migrate方法调用

5. 支持hasone,hasmany,manytomany关联查询

6. 支持left,right,inner join查询,支持count,sum,group查询

7. 支持连接池配置.支持数据链接检测以及自动重联，数据库服务的宕机修复后无需重启应用

8. 支持事务操作,包括同模型、跨模型、并行事务

9. 支持数据自动验证以及自定义规则验证,且规则可扩展

10. 支持前置、后置逻辑处理

# 开始

## 安装

```bash
npm install thinkorm --save
```

## 使用

used thinkorm@4.x

```js
//class User.js
const {BaseModel, helper, Entity, PrimaryColumn, Column, IsNotEmpty } = require('thinkorm');

@Entity()
class User extends BaseModel {
    @PrimaryColumn()
    id: number;

    @IsNotEmpty({ message: "姓名不能为空" })
    @Column(0, '', true)
    name: string;
}

//CURD
const userModel = new User(config);
// add
let result = await userModel.add({"name": "张三"});

// delete
result = await userModel.where({id: 1}).delete();

// update
result = await userModel.where({id: 2}).update({"name": "李四"});

// select 
result = await userModel.where({id: 3}).find(); //limit 1
result = await userModel.where({"name": {"<>": ""}}).select(); //query name is not null

```

# 对象关系映射

## 实体

通过@Entity注解，将模型类映射为实体表。支持自定义表名

```
//User.ts 

//对应数据库中的user表, 如果该参数未填写,默认值为类名
@Entity("User")
Class User extend BaseModel {
    ...
}
```

## 字段

通过@Column注解，将模型类成员属性映射为实体表字段

```

//User.ts 

@Entity("User")
Class User extend BaseModel {
    @PrimaryColumn() //主键
    id: number

    @Column(20, "") //表字段名name, 类型varchar(20)
    name: string; 

    @TimestampColumn() //时间戳类型字段
    create_time: number;
}
```

## 链式操作


例如：

```js
//useModel为实例化后的模型
useModel.where({name: 'aa'}).find(); //查询user表name值为aa的数据

```

## 关联关系

参见 [关联关系](#关联查询)

# 查询语法

## field(string | string[])

设置在查询中选择的列

```
// select 'aaa', 'bbb', 'ccc' from user limit 1;
userModel.field(['aaa', 'bbb', 'ccc']).find();

```

## alias(string)
设置所查询表的别名

```
// select u.id from user as u limit 1;
userModelalias('u').find();
```

## where(object)
设置查询条件，入参为object类型

### and

```js
userModel.where({ name: 'walter', state: 'new mexico' }).find();

userModel.where({ age: { '>=': 30 , '<=': 60}}).find();

```

### or

```js
// select * from think_user where (name = 'walter') or (occupation = 'teacher')
userModel.where({
    or : [
        { name: 'walter' },
        { occupation: 'teacher' }
    ]
}).find();

//select * from think_user where (id = 1 and name = walter) or (occupation ='teacher')
userModel.where({
    or : [
        { name: 'walter' , id: 1},
        { occupation: 'teacher' }
    ]
}).find();

```

### in

```js
userModel.where({
    name : ['Walter', 'Skyler']
}).find();
```

### not in

```js
userModel.where({
    name: { 'notin' : ['Walter', 'Skyler'] }
}).find();

userModel.where({
    notin: { 'name' : ['Walter', 'Skyler'] , 'id': [1, 3]}
}).find();
```

### is null

```js
userModel.where({
    name: null }
}).find();

```
### is not null

```js
userModel.where({
    not: {name: null} }
}).find();

userModel.where({
    name: {"!=": null} }
}).find();


userModel.where({
    name: {"<>": null} }
}).find();

```


### less than

```js
userModel.where({ age: { '<': 30 }}).find();
```

### less than or equal

```js
userModel.where({ age: { '<=': 30 }}).find();
```

### greater than

```js
userModel.where({ age: { '>': 30 }}).find();
```

### greater than or equal

```js
userModel.where({ age: { '>=': 30 }}).find();
```

### not equal

```js
userModel.where({ age: { '<>': 30 }}).find();

userModel.where({ age: { '!=': 30 }}).find();
```

### not

```js
userModel.where({ age: { 'not': 30 }}).find();

userModel.where({ not: { 'age': 30, 'name': 'aa' }}).find();

```

### like

```js
userModel.where({ name: { 'like': '%walter' }}).find();
userModel.where({ name: { 'like': 'walter%' }}).find();
userModel.where({ name: { 'like': '%walter%' }}).find();
```

## limit(skip: number, limit: number)
设置查询分拣的结果数量

```
// select * from user limit 20, 10;
userModel.limit(20, 10).select();
```

## order(values: object)
设置字段排序

```
//select * from user order by id desc limit 1;
userModel.order({"id": "desc"}).find();
```

## distinct(values: string[])
设置去重的字段

```
//select distinct(name) from user where name = 'aa';
userModel.distinct(["name"]).where({"name": "aa"}).select();
```
## group(values: string | string[])
设置分组查询的字段名

```
//select name from user where name = 'aa' group by age;
userModel.field(["name"]).where({"name": "aa"}).group("age").select();
```
## having(values: object)
用于分组查询的having子句. 仅可以配合group使用

```
//select name from user where name = 'aa' group by age having age > 10;
userModel.field(["name"]).where({"name": "aa"}).having({"age": {">":10}}).group("age").select();
```

## join(values: any[])
join查询
```js
//将join表字段写到field方法内，join表条件写入where
userModel
.field(['id','name','Demo.id','Demo.name'])
.join([{from: 'Demo', on: {demoid: 'id'}, type: 'inner'}])
.where({id: 1, 'Demo.name': 'test'})
.find()

//将join表字段声明在join方法的field属性内
userModel
.join([{from: 'Demo', alias: 'demo', on: {demoid: 'id'}, field: ['id', 'name'], type: 'inner'}])
.where({id: 1, 'demo.name': 'test'})
.find()
```
join方法传入的是一个数组，每一个数组元素均表示join一个表。

* from : 需要的join的模型名

* alias : 需要的join的模型查询别名

* on : join的on条件

* field : join表筛选的字段

* type : join的类型，目前支持 inner,left,right三种


## add(data: object | any[], options?: object)
新增数据

* data 新增的字段对象键值对象
* options 扩展项 

```
// insert into `user` (`name`) values ('qqqesddfsdqqq')
userModel.add({"name": "qqqesddfsdqqq"});

// insert into `user` (`name`) values ('qqqesddfsdqqq'), ('qqqesddfsdqqq')
userModel.add([{"name": "qqqesddfsdqqq"}, {"name": "qqqesddfsdqqq"}]);

```

## delete(options?: object)
删除数据

* options 扩展项 

```
// delete from `user` where `id` = 10 
userModel.where({ id: 10 }).delete();
```

## update(data: object, options?: object)
更新数据

* data 更新的字段键值对象
* options 扩展项 

```
//update `user` set `name` = 'aa' where `id` = 10
userModel.where({ id: 10 }).update({ name: 'aa' });
```

## increment(field: string, step = 1, data = {}, options?: object)
字段自增

* field 需要自增的字段名
* step 步进
* data 需要同步执行更新的其他字段数据
* options 扩展项 

```
//update `user` set `num` = `num` + 1 where `id` in (1, 2)
userModel.where({ id: [1, 2] }).increment('num', 1);
```

## decrement(field: string, step = 1, data = {}, options?: object)
字段自减

* field 需要自减的字段名
* step 步进
* data 需要同步执行更新的其他字段数据
* options 扩展项 

```
//update `user` set `num` = `num` - 1 where `id` in (1, 2)
userModel.where({ id: [1, 2] }).decrement('num', 1);
```

## count(field: string, options = {})
根据条件检索行数

* field 需要获取行数的字段名
* options 扩展项 

```
// select count(`num`) from `user` where `id` in (1, 2)
userModel.where({ id: [1, 2] }).count("num");
```


## sum(field: string, options = {})
根据条件计算字段求和

* field 需要求和的字段名
* options 扩展项 

```
// select sum(`num`) from `user` where `id` in (1, 2)
userModel.where({ id: [1, 2] }).sum("num");
```

## find(options?: object)
查询单条

* options 扩展项 

## select(options?: object)
查询多条数据

* options 扩展项 

## countSelect(options?: object)
分页查询

* options 扩展项 

## sql(options = {}, data?: object | any[]) 
生成sql语句，不同数据库会有所区别

## query(sqlStr: string, params = [])
原生语句查询

* sqlStr 原生sql语句，不同数据库有区别
* params bind模式字段名

```
userModel.query("select * from user where name='?' and age > ?", ["test", 18])
```

## transaction(fn: Function)
执行事务

* fn 事务包含的执行内容回调函数

## forUpdate(tsx: Object)
事务查询锁,必须在transaction回调函数内部使用

* tsx 事务回调函数入参.事务操作句柄

## migrate(sqlStr: string)
表结构同步到数据库


# 关联查询

## 一对一、一对多、多对多关联查询

使用类成员属性relation来声明关联关系.(后续拓展使用注解实现)

例如user.js类中申明的关联关系：

```js

//User.ts 

@Entity("User")
Class User extend BaseModel {
    ...

    //使用注解
    @Relations(Profile, "HASONE", "id", "user_id")
    profile: any;


    //或者定义 this.relation属性: 
    this.relation = {
        Profile: {
            type: 'hasone', //关联方式
            model: ProfileModel, //子表模型
            //field: ['test', 'id'],//关联表字段
            fkey: 'profile', //主表外键 (子表主键)
            rkey: 'id' //子表主键
        },
        Pet: {
            type: 'hasmany',
            model: PetModel, //子表模型
            //field: ['types','user', 'id'],
            fkey: '', //hasmany关联此值没用
            rkey: 'user'//子表外键 (主表主键)
        },
        Group: {
            type: 'manytomany',
            model: GroupModel, //子表模型
            //field: ['name', 'type', 'id'],
            fkey: 'userid', //map外键(主表主键)
            rkey: 'groupid', //map外键(子表主键)
            map: UserGroupModel//map模型
        }
    }
}
```
上述定义表示：

* 主表user同profile是一对一的关系

* 主表user同pet是一对多的关系

* 主表user同group是多地多关联关系

# 数据验证

# 结构迁移