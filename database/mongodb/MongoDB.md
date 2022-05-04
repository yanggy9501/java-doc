# MongoDB 

- 文档地址: https://docs.mongodb.com/manual/

## 1，简介

### 1.1 说明

- 官方
  
  MongoDB是一个`文档数据库`，旨在方便应用开发和扩展。
  
  <img src="assets/image-20211214132120713.png" alt="image-20211214132120713" style="zoom: 33%;" />

- 百度百科

MongoDB是一个`基于分布式文件存储的数据库`。由C++语言编写。旨在`为WEB应用提供可扩展高性能数据存储解决方案`。

MongoDB是一个`介于关系数据库和非关系数据库`之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。他支持的数据结构非常松散，是类似json的bson格式，因此可以存储比较复杂的数据类型 。Mongo最大的特点是他支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，`几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引`。

总结: mongoDB 是一个非关系型文档数据库

### 1.2 特点

-  面向集合存储，易存储对象类型的数据
-  支持查询, 以及动态查询
-  支持RUBY，PYTHON，JAVA，C++，PHP，C#等多种语言
-  文件存储格式为BSON（一种JSON的扩展）
-  支持复制和故障恢复和分片
-  支持多文档事务
-  索引 聚合 关联....

### 1.3 应用场景

- 游戏应用：使用云数据库MongoDB作为游戏服务器的数据库存储用户信息。用户的游戏装备、积分等直接以内嵌文档的形式存储，方便进行查询与更新。
- 物流应用：使用云数据库MongoDB存储订单信息，订单状态在运送过程中会不断更新，以云数据库MongoDB内嵌数组的形式来存储，一次查询就能将订单所有的变更读取出来，方便快捷且一目了然。
- 社交应用：使用云数据库MongoDB存储用户信息以及用户发表的朋友圈信息，通过地理位置索引实现附近的人、地点等功能。并且，云数据库MongoDB非常适合用来存储聊天记录，因为它提供了非常丰富的查询，并在写入和读取方面都相对较快。
- 视频直播：使用云数据库MongoDB存储用户信息、礼物信息等。
- 大数据应用：使用云数据库MongoDB作为大数据的云存储系统，随时进行数据提取分析，掌握行业动态。

### 1.4 数据库产品选择

## 2，安装

### 1，传统方式

#### 1.下载 MongoDB

```markdown
- https://www.mongodb.com/try/download/community
```

<img src="./assets/image-20220408214207120.png" alt="image-20220408214207120" style="zoom: 50%;" />

#### 2.解压安装

```markdown
# 2.将下载安装包上传到 linux 系统
- tar -zxvf mongodb-xxx.tgz 
```

```markdown
# 3.查看安装目录
- ls
	`bin`目录   用来存放启动mongoDB的服务以及客户端链接的脚本文件等
	使用：./mongod help 查看该命令的帮助
	其他命令也如此
```

<img src="assets/image-20211214104027532.png" alt="image-20211214104027532" style="zoom: 50%;" />

<img src="assets/image-20211214104107363.png" alt="image-20211214104107363" style="zoom:50%;" />

#### 3.启动服务

**命令参数启动**

```sh
./mongod --port=27017 --dbpath=../data --logpath=../logs/mongo.log --bind_ip=0.0.0.0 --fork
```

```markdown
# 启动 MongoDB 服务
- ./mongod --port=27017 --dbpath=../data/db --logpath=../logs/mongo.log --bind_ip=0.0.0.0 --fork
	`--port`    	指定服务监听端口号 默认为 27017
	`--dbpath`  	指定 mongodb 数据存放目录 启动要求目录必须存在
	`--logpath` 	指定 mongodb 日志文件存放位置
	`--logappend`	指定日志追加方式
	`--bind_ip`		指定监听地址，默认只监听localhost网卡
	`--fork`		后台启动
	`--auth`			开启认证模式

```

![image-20211214105137607](assets/image-20211214105137607.png)

`注意: 由于指定日志文件因此启动时日志输出到日志中终端不显示任何日志`

**配置文件启动**

```sh
./mongod -f /usr/local/server/mongodb-5.0.4/conf/mongodb.conf
```

编辑./conf/mongodb.conf文件（yaml格式），如下：

```yaml
systemLog:
  # MongoDB发送所有日志输出的目标指定为文件
  destination: file
  # mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: "/usr/local/server/mongodb-5.0.4/logs"
  # 当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
storage:
  # mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod。
  dbPath: "/usr/local/server/mongodb-5.0.4/data/db"
  journal:
    # 启用或禁用持久性日志以确保数据文件保持有效和可恢复。
    enabled: true
processManagement:
  # 启用后台运行，mongos或mongod进程的守护进程模式。
  fork: true
net:
  # 服务实例绑定的IP（云是局域网ip，不是公网ip，eth0的），默认是localhost, 0.0.0.0表示所有
  bindIp: 0.0.0.0
  # 绑定的端口，默认是27017
  port: 27017
```

#### 4.连接mongodb

```sh
# 5.客户端连接
./mongo --port=27017
```

![image-20211214105628652](assets/image-20211214105628652.png)

#### 5. 关闭mongodb服务

**方式一**

```sh
./mongod --port=27017 --dbpath=/usr/local/server/mongodb-5.0.4/data/db --shutdown
```

**方式二**

已经进入mongo shell，需要切换库admin

```sh
use admin
db.shutdownServer()
exit
```

### 2，Docker方式

```sh
# 1.拉取 mongodb 镜像
docker pull mongo:5.0.5
```

```sh
# 2.运行 mongo 镜像
docker run --name mongo-server -p 29017:27017 \
-e MONGO_INITDB_ROOT_USERNAME=root \
-e MONGO_INITDB_ROOT_PASSWORD=123456 \
-d mongo:5.0.5 --wiredTigerCacheSizeGB 1
```

![image-20211214110307469](assets/image-20211214110307469.png)

```sh
# 3.进入 mongo 容器
docker exec -it mongo-server  bash
# 进入Mongo shell
mongo -u root -p 123456
```

![image-20211214110427395](assets/image-20211214110427395.png)

## 3，核心概念

### 3.1 mongodb关系

| SQL概念           | MongoDB概念       |
| ----------------- | ----------------- |
| 数据库<database>  | 数据库<database>  |
| 表<table>         | 集合<collection>  |
| 行<row>           | 文档<document>    |
| 列<colume>        | 字段<field>       |
| 索引<index>       | 索引<index>       |
| 主键<primary key> | _id（字段）       |
| 视图<view>        | 视图<view>        |
| 表连接<join>      | 聚会操作<$lookup> |

**库<DataBase>**

​		`mongodb中的库就类似于传统关系型数据库中库的概念，用来通过不同库隔离不同应用数据`。mongodb中可以建立多个数据库。每一个库都有自己的集合和权限，不同的数据库也放置在不同的文件中，admin数据库的用户拥有所有库的权限。数据库存储在启动指定的data目录中。

**集合<Collection>**

​		`集合就是 MongoDB 文档集，类似于关系数据库管理系统中的表的概念`

​		集合存在于数据库中，一个库中可以创建多个集合。每个集合`没有固定的结构`，创建集合时不需要创建其结构，这意味着你在对集合可以插入不同格式和类型的数据，但通常情况下我们插入集合的数据都会有一定的关联性。

**文档**

​		文档集合中一条条记录，是一组键值对(即 BSON)，不是通常我们所理解 的PDF, word文档。

一个简单的文档例子如下：

```json
{"site":"www.xxx.xin", "name":"kato"}
```

**_id**

​	\_id是mongodb中文档的主键，插入数据时若不指定主键，MongoDB将自动添加一个ObjectId对象的\_id。

```js
// 查询时使用以下
{"_id": ObjectId("62519d2e9fa00726e9fc8db4")}
```

### 3.2 Mongo shell

mongo是mongodb的交互式JavaScript界面，它位为系统管理员提供强大的界面，并位开发人员提供测试数据和操作的方法。

```sh
./mongo --port=27017
# --port 指定端口
# --host 连接的主机地址
./mongo -uroot -p123456 --authenticationDatabase=admin
```

**mongo shell常用命令**

这些命令中都有部分专属mongo shell所以在navicat或者datagrip等客服端使用时会报异常。

| 命令                       | 说明                           |
| -------------------------- | ------------------------------ |
| show dbs \| databases      | 显示数据库列表                 |
| use 数据库名               | 切换或创建数据库，不存在则创建 |
| db.dropDatabase()          | 删除数据库                     |
| show collections \| tables | 显示当前数据库的集合列表       |
| db.集合.stats()            | 查看集合详情                   |
| db.集合.drop()             | 删除集合                       |
| show users                 | 显示当前数据库的用户列表       |
| show roles                 | 显示当前数据库的角色列表       |
| show profile               | 显示最近发生的操作             |
| load(“js文件”)             | 执行js脚本文件                 |
| exit \| quit()             | 退出当前shell                  |
| help                       | 查看帮助                       |
| db.help()                  | 查询当前数据库支持的方法       |
| db.集合.help()             | 显示集合的帮助信息             |
| db.version                 | 数据库版本                     |

### 3.3 安全认证

>   mongodb可以不设置安全认证，但是不安全，为了安全起见有必要设置安全认证，否则恶意用户知道IP即可登录mongodb，恶意篡改数据库数据。
>
>   **ps: **mongodb必须以授权模式启动，参数--auth

**鉴权模式启动**

```sh
./mongod -f /usr/local/server/mongodb-5.0.4/conf/mongodb.conf --auth
```

#### 3.3.1 创建管理账户

```markdown
# 设置管理员用户名，密码需要切换到admin数据库
use admin
# 创建管理员
db.createUser({"user": "root", "pwd": "123456", roles:["root"]})
# 查看所有用户
show users
# 删除用户
db.dropUser("用户名")
```

**常用权限**

| 权限名               | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| read                 | 允许用户读取指定数据库                                       |
| readWrite            | 允许用户读写指定数据库                                       |
| dbAdmin              | 允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile |
| dbOwner              | 允许用户在指定数据库中执行任意操作，增、删、改、查等         |
| userAdmin            | 允许用户向system.users集合写入，可以在指定数据库里创建、删除和管理用户 |
| clusterAdmin         | 只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限 |
| readAnyDatabase      | 只在admin数据库中可用，赋予用户所有数据库的读权限            |
| readWriteAnyDatabase | 只在admin数据库中可用，赋予用户所有数据库的读写权限          |
| userAdminAnyDatabase | 只在admin数据库中可用，赋予用户所有数据库的userAdmin权限     |
| dbAdminAnyDatabase   | 只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限       |
| root                 | 只在admin数据库中可用。超级账号，超级权限                    |

#### 3.3.2 认证登录

**查看认证帮助**

```sh
./mongo -help
```

![image-20220409092401288](assets/image-20220409092401288.png)

**使用管理员登录**

```shell
./mongo -u root -p 123456 --authenticationDatabase=admin或其他数据库名
```

**ps**：每个数据库可以有自己的用户去管理，无需暴露管理员用户

****

**重新赋予用户操作权限**

```js
db.grantRolesToUser( "用户名" , [ 
     { role: "clusterAdmin", db: "admin" } ,
     { role: "userAdminAnyDatabase", db: "admin"},
     { role: "userAdminAnyDatabase", db: "admin"},
     { role: "readWriteAnyDatabase", db: "admin"} 
 ])
```

**删除用户**

```js
 db.dropUser("用户") 
//删除当前数据库所有用户
db.dropAllUser()              
```



## 4，基本操作

### 4.1 数据库操作

- **查看所有库**

  ```sql
  show databases
  # 或
  show dbs
  ```

  ![image-20211214114636572](assets/image-20211214114636572.png)

  `默认系统数据库`

  - **`admin`**： 从权限的角度来看，这是"root"数据库。要是将一个用户添加到这个数据库，这个用户自动继承所有数据库的权限。一些特定的服务器端命令也只能从这个数据库运行，比如列出所有的数据库或者关闭服务器。
  - **`local`**: 这个数据永远不会被复制，可以用来存储限于本地单台服务器的任意集合
  - **`config`**: 当Mongo用于分片设置时，config数据库在内部使用，用于保存分片的相关信息。

- **切换或创建数据库**

  ```sql
  # 当数据库存在则切换数据库，不存在则创建数据库
  use 数据库名
  ```

  <img src="assets/image-20211214115055562.png" alt="image-20211214115055562" style="zoom: 50%;" />

  `注意: use 代表创建并使用,当库中没有数据时默认不显示这个库`

  ![image-20211214115241365](assets/image-20211214115241365.png)

- **删除数据库**

  `默认删除当前选中的库`

  ```sql
  db.dropDatabase()
  ```

  ![image-20211214115339513](assets/image-20211214115339513.png)

- **查看当前所在库**

  ```sql
  db
  ```

  ![image-20211217094919116](assets/image-20211217094919116.png)

### 4.2 集合操作

- **查看库中所有集合**

  ```sql
  show collections; | show tables;
  ```

  ![image-20211214120226543](assets/image-20211214120226543.png)

- **创建集合**

  ```sql
  # 创建集合方式一, 这种方式不指定集合的初始结构
  db.createCollection('集合名称', [options])
  # 方式二，插入创建并指定初始集合结构
  db.集合名称.insert(
      { 
          "age" : 29, 
          "likes" : [ "看动漫", "读书xx", "美女" ], 
          "name" : "yg" 
      }
  )
  ```
  
  ![image-20211214120316227](assets/image-20211214120316227.png)
  
  `options可以是如下参数：`
  
  | 字段   | 类型 | 描述                                                         |
  | :----- | :--- | :----------------------------------------------------------- |
  | capped | 布尔 | （可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。 **当该值为 true 时，必须指定 size 参数，日志可以使用。** |
  | size   | 数值 | （可选）为固定集合指定一个最大值，即字节数。 **如果 capped 为 true，也需要指定该字段。** |
  | max    | 数值 | （可选）指定固定集合中包含文档的最大数量。                   |

​		`注意:当集合不存在时,向集合中插入文档也会自动创建该集合。`

- **删除集合**

  ```sql
  db.集合名称.drop();
  ```

  ![image-20211214120404239](assets/image-20211214120404239.png)

### 4.3 文档操作

#### 4.3.1 插入文档

3.2 版本之后新增了db.collection.insertOne()和db.collection.insertMany()

Java客服端只有insert方法

**注意：** mongodb是一个文档型数据库，保存数据bson数据，因此value可以是一个普通类型，也可以是数组或者是一个bson对象。

##### 单条文档：insertOne 

>   insertOne 支持writeConcern（`集群`）.
>
>   writeConcern 决定一个写操作落到多少个节点才算成功（集群）。
>
>   writeConcern取值包括：
>
>   *   0：发起写操作，不关心是否成功
>   *   1-n：写操作需要被复制到指定节点数才算成功
>   *   majority：写操作需要被复制到大多数节点上才算成功
>
>   默认是1

```sql
db.collection.insertOne(
     <document bson文档>, // {"name": "asunaxx", "age": 21, "sex": "女"}
	{
    	writeConcern: <writeConcern bson对象> // {"writeConcern": 1}
    }
)
// 复杂一点的案例
db.user.insertOne({
    _id: "1",
    name: "yanggy",
    age: 27,
    likes: ["编程", "游戏", "听歌"],
    car: [
        {
            car_name: "lals",
            price: 9999999,
            color: "black"
        },
        {
            car_name: "mbh",
            price: 888888,
            color: "red"
        },
    ],
    love: {
        name: 'kato-2cy',
        age: 21
    }
})
```

##### 单条|多条：insert

该命令插入数据时，若数据主键已经存在，则会抛出主键重复异常，不保存当前数据。

支持单条和多条插入

```sh
db.collection.insert(
     <document bson文档>
)

db.collection.insert(
     [<document1>, <document2>,....]
)
```

##### ~~单条文档：save~~

如果_id 主键存在则更新数据，如果不存在就插入数据（不推荐使用，更新使用update）。

##### 多条文档：insertMany

```sql
db.collection.insertMany(
   [ <document 1> , <document 2>, ... ],
   {
 		writeConcern: 1, // 写入策略，默认为 1
      	ordered: true // 指定是否按顺序写入，默认 true。
   }
)

db.collection.insert(
	[ <document 1> , <document 2>, ... ]
);
```

- 脚本方式

  ```javascript
  for(let i=0;i<100;i++){
      db.users.insert({"_id": i, "name":"name_" + i, "age": 23});
  }
  ```

`注意:在 mongodb 中每个文档都会有一个_id作为唯一标识，_id默认会自动生成如果手动指定将使用手动指定的值作为_id 的值。`

### 4.4 查询文档

>   文档查询是一个最重要的操作，放在查询章节介绍。

### 4.5 更新文档

>    ​		可以使用update方法对指定的数据集进行更新，要更新的内容很多，如field的名字，field的value为某值，或者value的值增减，value可能是数组或对象更新又有所不同，因此更新操作有更新动作和更新内容两部分。
>
>    **update命令配置选项很多，为了简化使用还可以使用一些快捷命令**：
>
>    *   update：更新一条或多条文档，通过multi进行控制，默认一条
>
>    *   updateOne：更新匹配的第一条文档，相当update的multi=false
>    *   updateMany：更新所有匹配的文档，相当update的multi=true
>    *   replaceOne：替换单个文档，不存在时不创建，相当update的upsert=false

#### 命令格式

```sql
db.集合名称.update(
   <query  更新条件>,
   <update 更新动作及更新内容>,
   <options更新选项>
);
// 案例
db.restaurant.updateOne(
    {_id: ObjectId("625b8bdb0dcfe40dd8cc68b4")},
    {
        $set: {
            restaurantName: "lsp专卖店",
            location: {
                type: "Point",
                coordinates: [-73.97, 40.77]
            }
        }
    }
)
```

#### 参数说明

- **query** : update的查询条件，即哪些文档集可以被更新。

- **update** : update更新操作及其更新内容，两者结合使用。

- **options**: 描述更新的选项

    *   **upsert** : `可选`，如果不存在update的记录，是否插入objNew, true为插入，默认是false，不插入。

    - **multi** : `可选`，默认是false，只更新找到的第一条记录，如果这个参数为true, 就把按条件查出来多条记录全部更新。
    - **writeConcern** :`可选` ，写关注决定一个写操作落到多少个节点才算成功（集群，复制集有管）。



#### 更新操作符

| 操作符    | 格式                                        | 描述                                          |
| --------- | ------------------------------------------- | --------------------------------------------- |
| $set      | {$set: {field: value}}                      | 指定一个field更新其value，若key不存在则创建   |
| $unset    | {$unset: {field: 1}}                        | 删除该文档的该field                           |
| $inc      | {$inc: {field: value}}                      | 对数值类型进行增减                            |
| $rename   | {$rename: {old_field_name: new_field_name}} | 修改field名称                                 |
| $push     | {$push: {field: value}}                     | 将value追加到数组中，若数组不存在则进行初始化 |
| $pushAll  | {$pushAll: {field: value_array}}            | 将多个值追加到数组中                          |
| $addToSet | {$addToSet: {field: value}}                 | 添加元素到数组中，就有排重功能                |
| $pop      | {$pop: {field: 1}}                          | 删除数组的第一个或最后一个元素                |
|           |                                             |                                               |

**~~findAndModify命令~~**

>   findAndMOdify命令兼容了查询和修改指定单个文档的功能，该命令会返回符合查询条件的文档数据（默认返回旧值），并完成对文档的修改。

```js
db.集合名称.findAndModify({
    <query 匹配条件>,
    <update 更新操作和更新内容>
    <option 可选选项>
})
```

*   option
    *   new：是否返回新的值，不写就是false，返回旧值

### 4.6 删除文档

#### remove删除

使用remove命令删除文档

```sql
db.集合名称.remove(
   <query>,
   <options 更新选项>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```

**参数说明：**

- **query** :`可选`, 删除的文档的条件，不选删除所有。
- **options: **删除文档选项
    - **justOne** : `可选`如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档。
    - **writeConcern** :`可选`，决定一个写操作落到多少个节点才算成功（集群）。

****

#### delete删除

>   官方推荐使用deleteOne和deleteMany方法delete删除文档

**语法格式**

```js
db.集合名称.delete(条件)
// 删除符合条件的第一条文档，相当remove的justOne=true
db.集合名称.deleteOne(条件)
// 删除符合条件的多条文档
db.集合名称.deleteMany(条件)
// 删除所有
db.集合名称.deleteMany()
```

**ps**：如果希望删除整个集合，则使用drop命令会更加高效

#### ~~返回被删除文档~~

>   如果希望在删除的时候返回被删除的文档，则可以使用findOneAndDelete，还可以安装指定的属性删除找的第一个文档

```js
db.集合名称.findOneAndDelete({条件}, {sort: {排序条件}})
```

## 5，文档查询

​		**MongoDB 查询文档使用 find和findOne方法。find() 方法以非结构化的方式来显示所有文档。findOne返回匹配条件的第一条文档**

​		mongodb是一个嵌套文档型数据库（对象里面有对象），因此，对于比较复杂的查询mongodb是不擅长的。

### 5.1 基础语法

```sql
db.集合名称.find(query, projection)
```

- **query** ：可选，使用查询操作符指定`查询条件`
- **projection** ：可选，使用投影操作符指定返回的Filed（默认省略）。

如果你需要以易读的方式来读取数据，可以使用 pretty() 方法，语法格式如下：

```sql
db.集合名称.find().pretty()
```

`注意: pretty() 方法以格式化的方式来显示所有文档。`

如果你熟悉常规的 SQL 数据，通过下表可以更好的理解 MongoDB 的条件语句查询：

![image-20211214123631600](assets/image-20211214123631600.png)

#### 1. 等值查询

```js
// 多个条件用逗号分隔，多个条件相当于and
db.集合名称.find({
    {
    	<filed1>: <value1>,
    	<filed2>: <value2>,
    	...
	}
})
```

db.集合名称.find(query, projection)方法中query参数指定某Filed为某个值即可

```js
// 查询emp集合中name=kato的所有文档
db.emp.find({
    {"name": "kato"}
})
// 查询emp集合中name=asuna1并且age=21的所有文档
db.emp.find(
    {"age": 21, "name": "asuna1"}
)
```

**ps：** 同处于一个对象{}层级的条件都是and条件

#### 2. 范围查询

>   范围查询和等值查询基本类似，但是Filed的条件value有所改变，value不在是一个值还是一个对象，
>
>   mongodb是用json对象作为一个查询条件，一个普通的json是k-v结构的是一种等值的结构，范围查询时value中的key需要mongodb提供特殊key作为范围查询的标识。
>
>   **条件标识：**
>
>   1.   小于(less than)：$lt
>   2.   小于等于(less than equal)：$lte
>   3.   等于：就是等值查询，查看上面等值查询
>   4.   大于（greater than）：$gt
>   5.   大于等于（greater than equal）：$gte
>   6.   不等于（no equal）：$ne

```js
db.集合名称.find({
    {<filed1>: {条件标识: 值}, <filed2>: {条件标识: 值},...}
})
```

```js
db.emp.find(
    {"age": {"$gte": 19}}
)
```

#### 3. 模糊查询|正则查询

`类似 SQL 中为 'where name like '%name%''，mongodb使用正则作为模糊查询`

**完整方式**

```js
db.集合名称.find(
    {
    	<filed1>: {$regex: "正则字符串"} 
    }
)
```

**简写方式**

```sql
db.集合名称.find(
    {
    	<filed1>: /正则/, 
    	<filed2>: /正则/,...
    }
)
```

`注意:在 mongoDB 中使用正则表达式可以是实现近似模糊查询功能`

```js
db.emp.find({
    "name": /a*1/
})
```

### 5.2 多条件查询

| SQL             | Mongdb                                 |
| --------------- | -------------------------------------- |
| a = 1 AND b = 1 | {a: 1, b: 1}或{$and: [{a: 1}, {b: 1}]} |
| a = 1 OR b = 1  | {$or: [{a: 1}, {b: 1}]}                |
| a IS NULL       | {a: {$exists: false}}                  |
| a IN (1, 2, 3)  | {a: {$in: [1, 2, 3]}}                  |

#### 1. AND

and查询要求多个条件同时满足，`类似于 WHERE 语句：WHERE key1=value1 AND key2=value2`。

**1. 对于普通的and查询使用下面：**

```sql
db.集合名称.find(
    {
    	key1:value1,
    	key2:value2,
    	...
    }
).pretty()
```

**2. 对于会参杂其他的查询使用 $and**

因为是多个条件所以value是一个数组，数组里面是一个个的条件对象，这些条件对象是and关系。

```js
$and:[
    {key1:value1},
    {key2:value2},
    ...
]
```

```java
db.emp.findOne(
    {
        "$and": [
            {"name": /as*/},
            {"age": 21}
        ]
    }
)
```

#### 2. OR

MongoDB OR 条件语句使用了关键字 **$or**, 和and查询类似，数组里面条件对象的关系是or关系，语法格式如下：

```sql
db.集合名称.find(
   {
      $or: [
          {key1: value1},
          {key2:value2}
      ]
   }
)
```

`类似于 WHERE 语句：WHERE key1=value1 or key2=value2`

#### 3. AND 和 OR 联合

`类似SQL语句为：'where age >50 AND (name = 'name1' OR name = 'MongoDB')'`

主要控制and和or的范围

```sql
db.集合名称.find(
    {
        "age": {$gt:50}, 
    	$or: [
            {"name": "name1"},
            {"name": "MongoDB"}
        ]
    }
).pretty();
```

**注意**： and和or查询时and和or关系指定是数组[]里面的对象的关系，与外面无关。

#### 4. in

 对应MySQL中 `a IN (1,2,3)`的IN 语句。

**语法格式**

```sh
{field: {$in: [v1, v2, ....]}}
```

### 5.3 数组查询

mongodb是一个嵌套文档型数据库，value可以是一个数组对象，当value是数组对象时就有数组查询，即某个值在数组中。

```sql
-- 测试数据
db.集合名称.insert(
    { 
        "age" : 29, 
        "likes" : [ "看动漫", "读书xx", "美女" ], 
        "name" : "yg" 
    }
)
-- 执行数组查询
db.users.find(
    {likes: "看电视"}
)
-- $size 按照数组长度查询
db.users.find(
    {
    	likes: {$size: 3}
    }
)
```

<img src="assets/image-20220409225044778.png" alt="image-20220409225044778" style="zoom: 80%;" />

### 5.4 排序

调用sort方法指定排序方式:

*   1 升序  
*   -1 降序

```sql
db.集合名称.find()
	.sort(
        {
        	<filed1>: 排序方式如：1,
        	<filed2>: 排序方式如：1,
        	...
        }
    )
```

`类似 SQL 语句为: 'order by name,age'`

数据量大的时候，应该避免使用skip/limit形式的分页。替代方案：**使用查询条件+唯一排序条件。*

如：

第一页：db.posts.find({}).sort({_id: 1}).limit(20); 

第二页：db.posts.find({_id: {$gt: <第一页最后一个_id>}}).sort({_id: 1}).limit(20); 

第三页：db.posts.find({_id: {$gt: <第二页最后一个_id>}}).sort({_id: 1}).limit(20);

### 5.5 分页

先调用skip方法跳过前面start条数据，而调用limit方法则返回之后rows条数据

```sql
db.集合名称.find()
	.skip(start)
	.limit(rows);
```

`类似于 SQL 语句为: 'limit start,rows'`

```js
db.emp.find()
    .skip(1)
    .limit(2);
```

### 5.6 总条数

```sql
db.集合名称.count();
db.集合名称.find(条件).count();
```

`类似于 SQL 语句为: 'select count(id) from ....'`

### 5.7 去重

```sql
db.集合名称.distinct('字段')
```

`类似于 SQL 语句为: 'select distinct name from ....'`

### 5.8 值是否存在

类似于MySQL的 `a is null`，mondodb中也存在类似：

```sh
{a: {$exists: false}}
```

## 6，$type

**说明**：

$type操作符是基于BSON类型来检索集合中匹配的数据类型，并返回结果。

MongoDB 中可以使用的类型如下表所示：

![image-20211214125324193](assets/image-20211214125324193.png)

**使用**

```sql
> db.col.insert({
    title: 'PHP 教程', 
    description: 'PHP 是一种创建动态交互性站点的强有力的服务器端脚本语言。',
    by: 'name',
    url: 'http://www.baizhiedu.xin',
    tags: ['php'],
    likes: 200
});

> db.col.insert({
    title: 'Java 教程', 
    description: 'Java 是由Sun Microsystems公司于1995年5月推出的高级程序设计语言。',
    by: 'name',
    url: 'http://www.baizhiedu.xin',
    tags: ['java'],
    likes: 550
});

> db.col.insert({
    title: 'MongoDB 教程', 
    description: 'MongoDB 是一个 Nosql 数据库',
    by: 'name',
    url: 'http://www.baizhiedu.xin',
    tags: ['mongodb'],
    likes: 100
});

> db.col.insert({
    title: 2233, 
    description: '2233 是一个 B站的',
    by: 'name',
    url: 'http://www.baizhiedu.xin',
    tags: ['2233'],
    likes: 100
});
```

- 如果想获取 "col" 集合中 title 为 String 的数据，你可以使用以下命令：

```sql
db.col.find({"title" : {$type : 2}}).pretty();
或
db.col.find({"title" : {$type : 'string'}}).pretty();
```

- 如果想获取 "col" 集合中 tags 为 Array 的数据，你可以使用以下命令：

```sql
dge
或
db.col.find({"tags" : {$type : 'array'}}).pretty();
```

## 7，索引

https://docs.mongodb.com/manual/indexes/

​		索引通常能够极大的提高查询的效率，如果没有索引，MongoDB在读取数据时必须扫描集合中的每个文件并选取那些符合查询条件的记录。这种扫描全集合的查询效率是非常低的，特别在处理大量的数据时，查询可以要花费几十秒甚至几分钟，这对网站的性能是非常致命的。索引是帮助查询并且有序的特殊的数据结构，索引存储在一个易于遍历读取的数据集合中，索引是对数据库表中一列或多列的值进行排序的一种结构。

**索引的分类**：

-   按照索引包含的字段数量，可以分为单键索引和组合索引（或复合索引）。
-   按照索引字段的类型，可以分为主键索引和非主键索引。
-   按照索引节点与物理记录的对应方式来分，可以分为聚簇索引和非聚簇索引，其中聚簇索引是指索引节点上直接包含了数据记录，而后者则仅仅包含一个指向数据记录的指针。
-   按照索引的特性不同，又可以分为唯一索引、稀疏索引、文本索引、地理空间索引等

### 7.1 原理

![image-20211220093934250](assets/image-20211220093934250.png)

​		从根本上说，MongoDB中的索引与其他数据库系统中的索引类似。MongoDB在集合层面上定义了索引，并支持对MongoDB集合中的任何字段或文档的子字段进行索引。

### 7.2 索引操作

#### 1. 创建索引

```sql
db.集合名称.createIndex(keys, options)
// 案例
db.book.createIndex({"title":1, "description":-1})
```

`说明: 语法中 Key 值为你要创建的索引字段，1 为指定按升序创建索引，如果你想按降序来创建索引指定为 -1 即可。`

**options 可选参数列表如下**：

| Parameter            | Type          | Description                                                  |
| :------------------- | :------------ | :----------------------------------------------------------- |
| `background`         | Boolean       | 建索引过程会阻塞其它数据库操作，background可指定以后台方式创建索引，即增加 "background" 可选参数。 "background" 默认值为**false**。 |
| `unique`             | Boolean       | 建立的索引是否唯一。指定为true创建唯一索引。默认值为**false**. |
| `name`               | string        | 索引的名称。如果未指定，MongoDB的通过连接索引的字段名和排序顺序生成一个索引名称。 |
| `sparse`             | Boolean       | 对文档中不存在的字段数据不启用索引（**稀疏索引**），这个参数需要特别注意，如果设置为true的话，在索引字段中不会查询出不包含对应字段的文档。默认值为 **false** |
| `expireAfterSeconds` | integer       | 指定一个以秒为单位的数值，完成 TTL设定，设定集合的生存时间（**TTL索引**）。 |
| `v`                  | index version | 索引的版本号。默认的索引版本取决于mongod创建索引时运行的版本。 |
| weights              | document      | 索引权重值，数值在 1 到 99,999 之间，表示该索引相对于其他索引字段的得分权重。 |
| default_language     | string        | 对于文本索引，该参数决定了停用词及词干和词器的规则的列表。 默认为英语 |
| language_override    | string        | 对于文本索引，该参数指定了包含在文档中的字段名，语言覆盖默认的language，默认值为 language. |

**案例**

```sh
// 创建索引，后台执行
db.values.createIndex(
    {
        open:1,
        close: 1
    },
    {
    	background: true
    }
)
// 创建唯一索引
db.values.createIndex(
    {
        title:1,
    },
    {
    	unique: true
    }
)
```

#### 2、查看集合索引

```js
// 查看索引信息
db.集合名称.getIndexes()
// 查看索引键
db.集合名称.getIndexKeys()
```

#### 3、查看集合索引大小

```js
// 查看索引占用空间
// is_detail：可选参数,传入除0或false外的任意数据，都会显示该聚合中每个索引的大小及总大小。
// 如果传入0或false则只显示该集合中所有索引的总大小
db.集合名称.totalIndexSize()
```

#### 4、删除集合所有索引

```sql
db.集合名称.dropIndexes()
```

#### 5、删除集合指定索引

```sql
db.集合名称.dropIndex("索引名称")
```

### 7.3 索引类型

索引类型的介绍

****

#### 7.3.1 单键索引

​		在某个字段上建立索引，如mongodb在id上建立了唯一的单键索引，所以经常会使用id来进行查询；在使用字段上进行精准匹配、排序以及范围查询都会使用到索引（单键索引即索引字段只有一个）。

​		如果是数组或内嵌文档又该如何呢？

**对普通属性**：如下对title建立单键索引

```js
db.books.createIndex({title: 1})
```

**对内嵌文档**

```js
// author: {name: "xxx", age: 19}
db.books.createIndex(
	{
        "author.name": 1
    }
)
```

**多键索引**

​		在数组的属性上建立索引。针对这个数组的任意值的查询都会定位到这个文档，即多个索引入口或者键值应用同一个文档。

​		多键索引很容易与复合索引产生混淆，复合索引是多个字段的组合，而多键索引则仅仅是在一个字段上出现了多键（multi key）。而实质上，多键索引也可以出现在复合字段上

​		**注意**：mongodb并不支持一个复合索引中同时也出现多个数组字段。

```js
db.集合.createIndex({和单键索引一致})
```

![image-20220417112533214](assets/image-20220417112533214.png)

#### 7.3.2 复合索引

​		复合索引是多个字段组合而成的索引，其性质合单字段索引类似。但不同的是，复合索引中字段是有顺序的，字段的升序降序对查询性能都有直接的影响，异常在设计复合索引的时候要考虑不同的场景。

```sql
db.collection.createIndex({key,....})
// 如：
db.books.createIndex({"title":1,"description":-1})
```

​	`注意: mongoDB 中复合索引和传统关系型数据库一致都是最左前缀原则`![image-20211220122343526](assets/image-20211220122343526.png)

**查询案例**

![image-20211220122531182](assets/image-20211220122531182.png)

#### 7.3.3 地理空间索引

​		mongodb为地理位置检索提供了非常方便的功能，地理空间索引(2dsphereindex)就是专门用于实现位置检索的一种特殊索引。

**语法格式**：创建索引时value值不再是1或者-1，而是字符串“2dsphere”

```js
db.集合.createIndex({field: "2dsphere"})
```

**案例**：mongodb如何实现查询附件商家？

```java
db.restaurant.insert({
    restaurantId: 0,
    restaurantName: "lsp专卖店",
    location: {
        type: "Point",
        coordinates: [-73.97, 40.77]
    }
})
```

**创建地理空间索引**

```js
db.restaurant.createIndex({location: "2dsphere"})
```

**查询附件10000米商家信息**

```js
db.restaurant.find(
	{
        location: {
            $near: {
                $geometry: {
                    type: "Point",
                    coordinates: [-73.88, 40.78]
                },
                $maxDistance: 10000
            }
        }
    }
)
```

*   \$near：查询操作符，用实现附件地位位置查询，返回数组结果或按距离排序
*   \$geometry：操作符用于指定一个GeoJson格式的地理位置空间对象，type=Point表示地位位置坐标，coordinates则是地理位置的经纬度
*    \$maxDistance: 限定查询的最多距离，单位米

#### 7.3.4 ~~全文索引~~

​		mongodb支持全文检索功能，可通过建立文本随意来实现建议的分词检索。

**创建分词索引**

```js
db.集合.createIndex({field1: "text", ...})
```

\$text操作符可以在又text index的集合上执行文本检索。\$text将会使用空格合标点符号作为分词符，并且对检索字符串中的所有分词进行一个逻辑是国内的or操作。

**注意**：mongodb的文本索引功能是否受限，不推荐使用。

#### 7.3.5 Hash 索引

>   不同于传统的B+树索引，Hash索引是在索引字段上进行精确匹配的索引，但是，其不支持范围查询，也不支持多键hash，Hash索引上的入口是均匀分布的，在分配集合中非常有用。

```js
db.集合.createIndex({
    field: 'hashed'
})
```

#### 7.3.6 通配符索引

mongodb的文档模式是动态变化的（不知道有哪些属性等），而通配符索引可以建立在一些不可预知的字段上，以此实现查询的加速。mongodb引入通配符来支持对未知或任意字段的查询

**准备数据**

```js
db.products.insert([
    {
        "product_name": "spy coat",
        "product_attributs": {
            "material": ["tweed", "wool", "leather"],
            "size": {
                "length": 72,
                "units": "inches"
            }
        }
    },
    {
        "product_name": "spy pen",
        "product_attributs": {
            "color": ["blue", "black"],
            "secret_feature": {
                "name": "laser",
                "power": "10000",
                "units": "watts"
            }
        } 
    },
    {
       "product_name": "spy book"
    }
])
```

**创建通配符索引**

```js
db.products.createIndex({
    "prodcut_attributes.$**": 1
})
```

**注意事项**：

*   通配符索引不兼容的索引类型或属性，如：复合索引，地理位置，.....
*   通配符索引是稀疏索引，不索引空字段。因此，通配符索引不能支持查询字段不存在的文档

### 7.4 

### 索引属性

​		其是针对创建索引的第二个参数而言的。

#### 7.4.1 唯一属性

​		通过建立唯一属性，可以保证集合中文档的指定字段拥有唯一值。

```js
# 创建唯一索引
db.books.createIndex(
    {"titlel": 1},
    {"unique": true}
)
# 复合索引支持唯一性约束, 多个字段的组合是唯一的
db.books.createIndex(
    {"titlel": 1, "type": 1},
    {"unique": true}
)
# 多建索引支持唯一性约束，数组中的一项是唯一的
db.books.createIndex(
    {"tags": 1},
    {"unique": true}
)
```

*   唯一性索引对应文档中缺失的字段，会使用null值代替，因此不允许存在多个文档缺失索引字段的情况
*   对于分片的集合，唯一性约束必须匹配分片规则。换句话说，为了保证全局的唯一性，分片键必须作为唯一性的前缀字段

#### 7.4.2 部分索引

​	 	部分索引仅针对满足指定过滤器表达式的文档进行索引。通过在一个集合中为文档的一个子集合建立索引，部分索引具有更低的存储需求和更低的索引创建和维护成本。

​		部分使用提供了稀疏索引功能的超集，应该优先于稀疏索引。

**创建索引**

```sh
db.restaurants.createIndex(
	{cuisione: 1, name: 1},
	{partialFilterExpression: {tating: {$gt: 5}}}
)
```

partialFilterExpression 选项接收受控指定过滤条件的文档

*   等值表达式，如：field：value 或者$eq
*   $exist: true
*   $gt，.....
*   $type
*   顶层的$and

注意：如果同时指定了partialFilterExpression和唯一约束，那么唯一约束只适用于满足筛选器表达式的文档。

#### 7.4.3 稀疏索引

​		稀疏索引确保索引只包含具有索引字段的文档的条目，索引将跳过没有索引字段的文档。如果稀疏索引会导致查询和排序操作的结果集不完整，MongoDB将不会使用该索引，除非hint()明确指定索引。

**特性**：只对存在字段的文档进行索引（包括字段值为null的文档）

**创建索引**

```js
db.address.createIndex(
	{"xxx": 1},
    {sparse: true}
)
```

**注意**：如果稀疏索引会导致查询和排序操作的结果不完整，mongodb将不会使用索引，除非hint（）明确指定索引。

#### 7.4.4 TTL索引

​		在一般的应用系统中，并非所有的数据都需要永久存储。例如一些系统事件、用户消息等，这些数据随着时间的推移，其重要程度逐渐降低。更重要的是，存储这些大量的历史数据需要花费较高的成本，因此项目中通常会对过期且不再使用的数据进行老化处理。

通常的做法如下：

方案一：为每个数据记录一个时间戳，应用侧开启一个定时器，按时间戳定期删除过期的数据。

方案二：数据按日期进行分表，同一天的数据归档到同一张表，同样使用定时器删除过期的表。

对于数据老化，MongoDB提供了一种更加便捷的做法：TTL（Time To Live）索引。TTL索引需要声明在一个日期类型的字段中，TTL 索引是特殊的单字段索引，MongoDB 可以使用它在一定时间或特定时钟时间后自动从集合中删除文档。

```js
// 创建 TTL 索引，TTL 值为3600秒
db.eventlog.createIndex( { "lastModifiedDate": 1 }, { expireAfterSeconds: 3600 } )
```

日志存储：

-   日期分表
-   固定集合
-   TTL索引

插入：  writeConcern:{w:0}

## 8，聚合

MongoDB 中聚合(aggregate)主要用于处理数据(诸如统计平均值，求和等)，并返回计算后的数据结果。聚合操作的结果值来自于多个文档，可以对分组数据执行各种操作以返回当结果。聚合操作包含：

1.   单一作用聚会（很少用）
2.   聚合管道
3.   ~~MapReduce（高版本可能会抛弃，功能太局限）~~

**常见聚合表达式**：

![image-20211214131110733](assets/image-20211214131110733.png)

### 8.1 单一作用聚合

>   mongodb提供 
>
>   *   db.集合.estimatedDocumentCount()：忽略查询条件，返回集合或视图所有文档的计数
>   *   db.集合.count()：返回与find集合或视图的查询匹配的文档计数。等同于db.集合.find(query).count()
>   *   db.集合.distinct()：单个字段去重，基本不用
>
>   这类单一作用的聚合函数。这些操作都聚合来自当集合的文档（但缺少像聚合管道和MapReduce的灵活性）。

**案例**： 

```sh
db.books.estimatedDocumentCount()
db.books.count({favCount:{$gt: 50}})
db.books.distinct({favCount:{$gt: 50}})
```

### 8.2 集合管道

>   mongodb聚合框架（aggregate framework）是一个计算框架，它可以：
>
>   *   作用在一个或几个集合上
>   *   对集合的数据进行的一系列运算
>   *   将这些数据转化为期望的形式
>
>   从效果而言，其相当于mysql中的group by、left/right join、as等。

#### 8.2.1 管道（pipeline）和阶段（stage）

整个聚合运算过程称之为管道，它是由多个阶段组成的。

![image-20220415211006422](assets/image-20220415211006422.png)

#### 8.2.2 聚合管道语法

```sh
pipeline=[$stage1, $stage2, ...]
db.集合.aggregate(pipeline, {option})
```

**pipeline**: 一组数据聚合操作。除了$out, \$Merger阶段之外，每个阶段都可以在管道中出现多次。

**option**：可选参数。包含：查询计划，是否使用临时文件，游标，最大操作时间，读写策略等。

**案例图**![image-20220415212659920](assets/image-20220415212659920.png)

#### 8.2.3 常用管道聚合操作

| 阶段            | 描述     | 等价sql         |
| --------------- | -------- | --------------- |
| $match          | 筛选条件 | where           |
| $project        | 投影     | as              |
| $lookup         | 左外连接 | left outer join |
| $out            | 排序     | order by        |
| $group          | 分组     | group by        |
| $skip/\$limit   | 分页     | limit           |
| $unwind         | 展开数组 |                 |
| $graphLookup    | 图搜索   |                 |
| $facet/\$bucket | 分面搜索 |                 |
|                 |          |                 |

##### $projcet 投影

投影操作，可以将原始字段投影成指定名称，如将集合中name投影成username。

$projcet 可以灵活控制输出文档的格式，也可以剔除不需要的字段。

**语法格式**

```java
db.集合.aggregate(
    [
        {
            $project: {新field名: "$旧filed名"},
            ...
        }
    ]
)
```

```sh
db.user.aggregate(
    [
        {
            $project: {username: "$name"}
        }
    ]
)
// 结果
[
  {
    "_id": 0,
    "username": "kato0"
  },
  {
    "_id": 1,
    "username": "alice1"
  }
]
```

##### $match 筛选

$match用于对文档的筛选，之后可以在得到的文档子集合上做聚合，\$match可以使用除了地理空间之外的所有常规操作符，在实际应用中尽可能的将\$match放在管道的前面。好处：

1.   快速过滤不需要的文档，以减少管道的工作量
2.   如果在投影和分组之前执行\$match，查询可以使用索引

**ps**: 可以使用query查询条件

```sh
db.集合.aggregate(
    [
        {
            $match: query条件
        }
    ]
)
// 案例
db.user.aggregate(
    [
        {
            $match: {name: {$regex: "kato*"}}
        }
    ]
)
```

##### $count 统计

统计某个字段的个数

```sh
db.user.aggregate(
    [
        {
            $match: {name: {$regex: "kato*"}}
        },
        {
            $count: "name"
        }
    ]
)
```

##### $group 分组

 按照指定的表达式对文档进行分组，并将每个不同分支（分组）的文档输出到下一个阶段。输出文档包含一个\_id字段，该字段按key包含不同的组。输出文档还可以包含计算字段，该字段保存由$group的\_id字段分组的以下accumulator表达式的值。\$group 不会输出具体的文档而是统计信息。

```js
// field是文档字段，accumulator是运算符，expression是分组表达式（即告诉怎么分组），如果使用文档字段则$+字段名
{$group: {_id: <expression>, <field>:{<accumulator1>:<expression1>}, ...}}
```

*   \_id：该字段必须填，但是，可以指定id值位null来为整个输入计算累计值（null时整个做一组）
*   其他有字段：可选，并使用<accumulator>运算符进行计算

**accumulator 运算符**:

| 运算符       | 描述                                          | 类比sql    |
| ------------ | --------------------------------------------- | ---------- |
| \$avg        | 计算平均值                                    | avg        |
| \$first      | 返回每组第一个文档，若排序，按照排序第一个    | limit 0，1 |
| \$last       | 返回没组最后一个文档，若排序，按照排序        |            |
| \$max        | 根据分组，获取集合中所有文档对应的最大值      | max        |
| \$min        | 根据分组，获取集合中所有文档对应的最小值      | min        |
| \$push       | 将指定的表达式的值添加到一个数组中            |            |
| $addToset    | 将指定的表达式的值添加到一个数组中(去重+乱序) |            |
| \$stdDevPop  | 返回输入值的总体标准偏差                      |            |
| \$stdDevSamp | 返回输入值的样本标准偏差                      |            |

**ps**：\$group阶段的内存限制为100M，可以设置allowDiskUse选项设置位true以启用\$group操作以写入临时文件。

**book的数量，收藏总数和平均值**

```sh
// 返回时：key就是返回的字段名，如下的：count，pop，avg
db.books.aggregate([
	{
		$group: {
            _id: null,
            count: {$sum: 1},
            pop: { $sum: "$facCount"},
            avg: {$avg: "$favCount"}
		}
	}
])
```

**统计每个作者的book收藏总数**

```sh
db.books.aggregate([
	{
		$group: {
            _id: "$author.name",
            pop: { $sum: "$facCount"}
		}
	}
])
```

**统计每个作者的每本book的收藏数**：有两次分组或者说根据两者字段分组

```sh
db.books.aggregate([
	{
		$group: {
            _id: {
            	name: "$author.name",
            	title: "$title"
            },
            pop: { $sum: "$facCount"}
		}
	}
])
```

##### \$unwind

可以将数组拆分位单个文档

```sh
{
	$unwind: {
		// 主要指定字段路径，在字段名称前加上$ 符号并用引号括起来
		path: <field path>,
		// 可选，一个新字段的名称用于存放元素的数组索引。该名称不能一般$ 开头
		includeArrayIndex: <String>,
		// 可选，default：flase，若位true，如果路径位空，缺少或者位空数组，则unwind输出文档
		preserveNullAndEmptyArrays: <boolean>
	}
}
```

**姓名位xxx的作者的book的tag数组拆分为多个文档**

```sh
db.books.aggregate([
	{$match: {"author.name": "xxx"}},
	// 书名的tag标签有多个，将其展开
	{$unwind: "$tag"}
])
```

**每个作者book的tag合集**

```sh
db.books.aggregate([
	{$unwind: "$tag"},
	{$group: {
		_id: "$author.name"
		types: {$addToSet: "$tag"}
	}}
])
```

##### $limit

限制传递到管道中下一阶段的文档数

```sh
db.集合.aggregate([
	{$limit: n}
])
```

**注意**：与\$sort集合使用，mongodb做了优化，无论\$sort在前还是在后limit都是排好序的前n个。

##### \$skip

跳过前n个文档，后面的文档传递到下一个阶段。

```sh
db.集合.aggregate([
	{$skip: n}
])
```

##### \$sort

排序

##### \$lookup

>   主要用来实现表关联查询，相当关系型数据库中多表关联查询。每个输入待处理文档，结果\$lookup阶段的处理，输出的新文档中会包含一个新生产的数组（可更加需要命名新key）。数组列存放的数据是来自被join集合的适配文档，如果没有，则集合为空（[]).

**语法结构**

```sh
db.集合.aggregate([
	$lookup: {
		from : "<collection to join>",
		localField: "<field from the input document>",
		foreignField: "<field from the document of the from collection>",
		as: "<output array field>"
	}
])
```

**属性说明**

| 属性         | 作用                                                         |
| ------------ | ------------------------------------------------------------ |
| from         | 被join的集合                                                 |
| localField   | 源集合中mathc值，如果输入的集合中，某个文档没有localField这个Field字段，在处理的过程中会默认为是有：<localField：null>的键值对。 |
| foreignField | 待join集合的match值，如果被join的集合中，某个文档没有foreignField这个Field，在处理的过程中会默认为传闻还有foreignField：null的键值对。 |
| as           | 为输出文档的新增值名。如果输入的集合以及存值该值，则覆盖掉   |

**ps**：null = null为true

**关联查询**

```sh
// 查询顾客及其订单
db.customer.aggragate([
	{
		$lookup: {
			form: "order", // join表
			localField: "customerId",// 主表字段
			foreignField: "customerId", // join表字段
			as: "customerOrder"// 主表多出字段用于存放join的数据
		}
	}
])
// 订单详细(关联顾客，订单详情表)
db.order.aggragate([
	{
		$lookup: {
			form: "customer", // join表
			localField: "customerId",// 主表字段
			foreignField: "customerId", // join表字段
			as: "customer"// 主表多出字段用于存放join的数据
		}
	},
	{
		$lookup: {
			form: "orderItem", // join表
			localField: "orderId",// 主表字段
			foreignField: "orderId", // join表字段
			as: "orderItem"// 主表多出字段用于存放join的数据
		},
	}
])
```

### 8.3 MapReduce

>   MapReduce能实现的功能，都可用聚合管道来完成，并且高版本可能废弃。

## 9, 整合spring boot

**说明: 这里主要以 springboot 应用为基础应用进行整合开发。**

Spring Data : `Spring 数据框架 JPA 、Redis、Elasticsearch、AMQP、MongoDB`

`JdbcTemplate`

`RedisTemplate`

`ElasticTempalte`

`AmqpTemplate`

`MongoTemplate`

SpringBoot  Spring Data  MongoDB

<font color="red">MongoTemplate上的操作和在Mongo shell上的操作基本一样（函数名是相同的），除此之外还可以使用MongoRepository，这两个都是用相同的依赖</font>

*   MongoTemplate
*   MongoRepository

### 9.1,  环境准备

**1. 依赖**

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

**2. 编写配置**

mongodb的自动配置类MongoAutoConfiguration

**yaml：~~yaml配置密码时有问题~~**

password属性注入失败，在MongoProperties中password定义为char数组，注入失败，如果使用uri能正常连接。

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://root:123456@192.168.134.128:27017/test?authSource=admin
      # 使用uri等价下面所有配置
      #database: test
      #host: 192.168.134.128
      #port: 27017
      # 开启安全认证时输入
      #username: root
      #password: 123456
      #authentication-database: admin
```

**properties：配置密码可以使用**

```properties
# mongodb 默认没有开启任何安全协议
# mongodb(协议)://[用户名:密码@]121.5.167.13(主机):27017(端口)/test(库名)
# spring.data.mongodb.uri=mongodb://localhost:27017/test

spring.data.mongodb.database=test
spring.data.mongodb.host=192.168.134.128
spring.data.mongodb.port=27017
spring.data.mongodb.username=root
spring.data.mongodb.password=123456
spring.data.mongodb.authentication-database=admin
```

### 9.2, 集合操作

>   一般集合操作很少在Java进程操作，就像java很少回去操作mysql的表一样。

#### 1. 判断集合存在

mongoTemplate#collectionExists

```java
// 判断集合是否存在
boolean exists = mongoTemplate.collectionExists("集合名字");
boolean exists = mongoTemplate.collectionExists(ORM映射类.class);
```

#### 2. 创建集合

mongoTemplate#createCollection

```java
@Test
public void testCreateCollection(){
    // 判断集合是否存在
    boolean exists = mongoTemplate.collectionExists("emp1");
    if (exists) {
        // 删除集合 db.集合.drop()
        mongoTemplate.dropCollection("emp1");
    }
    // 参数: 创建集合名称或ORM映射类.class
    // db.createCollection("集合名称"，option参数)，同样重装了第二个参数option
    mongoTemplate.createCollection("emp1");
}
```

`注意:创建集合不能存在,存在报错`

#### 3. 删除集合

 mongoTemplate#dropCollection

```java
@Test
public void testDeleteCollection() {
  // db.集合.drop()
  mongoTemplate.dropCollection("emp1");
}
```

****

### 9.3 文档操作

>   一般Java对象会和mongodb的文档一一对应，需要了解mongodb的文档（集合）如何与Java对应。mybatis-plus通过注解对应mysql的表，同样的mongodb通过注解对应Java对象。

#### 9.3.1 相关注解

- `@Document`
    - 修饰范围: 用在实体类上
    - 作用: 用来映射这个类的一个对象为 mongo 中一条文档数据
    - 属性：(`value 、collection` )用来指定操作的集合名称
- `@Id`
    - 修饰范围: 用在成员变量、方法上
    - 作用：用来将成员变量的值映射为文档的_id 的值
- `@Field`
    - 修饰范围: 用在成员变量、方法上
    - 作用: 用来将成员变量以及值映射为文档中一个key、value对
    - 属性: ( `name, value `)用来指定在文档中 key 的名称，默认为成员变量名
- `@Transient`
    - 修饰范围: 用在成员变量、方法上
    - 作用: 用来指定改成员变量，不参与文档的序列化

**创建实体类**

```java
@Document(value = "emp") // 对应emp集合
public class Employee {
    @Id
    private Long id;

    @Field("name")
    private String username;

    @Field
    private int age;

    @Field
    private Double salary;

    @Field
    private Date birthday;
}
```

#### 9.3.2 查询文档

>   mongo shell查询文档：db.集合名称.find | findOne
>
>   criteria是标准查询的接口，可以引用静态criteria.where把多个条件组合在一起，就可以将多个方法标志和查询连接起来，方便查询。

![image-20211221201808467](assets/image-20211221201808467.png)

##### 查询所有

1.   mongoTemplate#findAll
2.   mongoTemplate#find

```java
@Test
public void testFindAll() {
    // db.集合.findMany()
    List<Employee> all = mongoTemplate.findAll(Employee.class);
    System.out.println(all);
}

// 条件查询：无条件
@Test
public void testFindAll() {
     // db.集合.find()
    List<Employee> all = mongoTemplate.find(new Query(), Employee.class);
    System.out.println(all);
}
```

##### 主键查询

通过文档_id查询

mongoTemplate#findById

```java
@Test
public void testFindById() {
    // db.集合.findOne({"_id", 0}) 或 db.集合.find({"_id", 0})
    Employee emp = mongoTemplate.findById(0, Employee.class);
    System.out.println(emp);
}
```

##### 查询一个

>   查询结果只返回匹配的第一个，其就是条件查询，条件查询往下有详细内容。
>
>   db.集合.findOne({"_id", 0})

```java
@Test
    public void testFindOne() {
        Employee emp = mongoTemplate.findOne(new Query(), Employee.class);
        System.out.println(emp);
    }
```

##### 条件查询

>   由mongo shell条件查询，我们知道条件查询需要一系列的json条件参数，在Java端要构建类似于mongo shell端json条件参数。Java端使用`Query`类进行封装条件。
>
>   ```java
>   public Query() {
>   }
>   
>   public Query(CriteriaDefinition criteriaDefinition) {
>       this.addCriteria(criteriaDefinition);
>   }
>   ```
>
>   由构造函数知道其需要一个CriteriaDefinition实例（Criteria是1其实现类），criteria是标准查询的接口，可以引用静态criteria.where把多个条件组合在一起。因此，使用条件查询先创建Criteria对象。
>
>   `Criteria提供了静态方法直接调用，或者new一个Criteria对象`
>
>   **指定Field**：条件查询之指定是哪个field的条件
>
>   *   where(field)：用于条件查询的开始，如：{k1: v1, K2: v2}的k1
>   *   and((field))：指定另外一个field的条件并且和前面成&关系，这里的and是json{k1: v1, K2: v2}中k1和k2的关系，就是k2，不是`$and`

###### 等值查询：{k1: v1, k2: v2}

```java
@Test
public void testFind1() {
    // 等值查询：name=luora1 and age=20
    Criteria criteria = Criteria.where("name").is("luora1").and("age").is(20);
    Employee emp = mongoTemplate.findOne(new Query(criteria), Employee.class);
    System.out.println(emp);
}
```

###### 范围查询

使用where指定字段，如果有多个字段要指定使用and。

**注意**：范围查询的字段指定指定一次，否则报错。

```java
@Test
public void testRangeQuery() {
    // 19 <= age <=25 and salary >= 13000
    Criteria criteria = Criteria.where("age").gte(19).lte(25).and("salary").gte(13000);
    List<Employee> employees = mongoTemplate.find(new Query(criteria), Employee.class);
    System.out.println(employees);
}
```

###### 模糊查询|正则查询

>   在java端正则查询时，正则字符串无需“//“

```java
@Test
public void testRegexQuery() {
    // 正则查询
    Criteria criteria = Criteria.where("name").regex("katou*");
    List<Employee> employees = mongoTemplate.find(new Query(criteria), Employee.class);
    System.out.println(employees);
}
```

##### 多条件查询

>   这里的多条件查询说 and，or的复杂查询（$and, \$or)
>
>   *   Criteria#orOperator ：接收可变参数Criteria，条件间是and关系
>   *   Criteria#orOperator ：接收可变参数Criteria，条件间是or关系

```java
@Test
public void testAndOrQuery1() {
    Criteria criteria = new Criteria();
    // 可以直接用等值查询
    criteria.andOperator(Criteria.where("name").is("alice2"), Criteria.where("age").is(21));
    Employee employee = mongoTemplate.findOne(new Query(criteria), Employee.class);
    System.out.println(employee);
}

@Test
public void testAndOrQuery1() {
    Criteria criteria = new Criteria();
    // name=alice2 or age=25
    criteria.orOperator(Criteria.where("name").is("alice2"), Criteria.where("age").is(25));
    List<Employee> employee2 = mongoTemplate.find(new Query(criteria), Employee.class);
    System.out.println(employee2);
}

@Test
public void testAndOrQuery2() {
    Criteria criteria = new Criteria();
    // salary > 13013 or (name=kato* and age < 30)
    criteria.orOperator(Criteria.where("salary").gte(13013),
                        new Criteria().andOperator(
                            Criteria.where("name").regex("kato*"),
                            Criteria.where("age").lt(30))
                       );
    // (name=kato* and age < 30)可以使用Criteria.where("name").regex("kato*").and("age").lt(30)
    List<Employee> employee2 = mongoTemplate.find(new Query(criteria), Employee.class);
    System.out.println(employee2);
}
```

##### 排序

>   db.集合.find().sort({
>           	<filed1>: 排序方式如：1,
>           	<filed2>: 排序方式如：1,
>           	...
>           })
>
>   在Java中也类似，在查询的时候调用sort方法
>
>   Query#with：指定对查询结果做什么额外操作如：排序，分页等
>
>   Sort#by：指定通过什么排序
>
>   Sort#Order#desc：指定排序方式和字段

```java
//sort 排序
Query query = new Query();
query.with(Sort.by(Sort.Order.desc("age")));//desc 降序  asc 升序
template.find(query, User.class);
```

##### 分页

```java
//skip limit 分页
Query queryPage = new Query();
queryPage.with(Sort.by(Sort.Order.desc("age")))
    .skip(0) //起始条数
    .limit(4); //每页显示记录数
template.find(queryPage, User.class);
```

##### 其他

```java
//count 总条数
template.count(new Query(), User.class);

//distinct 去重
//参数 1:查询条件 参数 2: 去重字段  参数 3: 操作集合  参数 4: 返回类型
template.findDistinct(new Query(), "name", User.class, String.class);
```

#### 9.3.3 Json查询

>   上述的查询需要创建对象，然后创建条件，操作起来比较麻烦。有没有一种比较简单的方法向mongo shell一样的方法来操作mongodb呢？
>
>   只需要将Query对象替换成BasicQuery对象即可，这样就可以传一个json字符串了。
>
>   query条件就可以使用json字符串来表示和使用mogo shell完全一样。
>
>   <font color="red">期待出现一个以配置的方式代替拼接json，类似于mybatis，其中变量使用占位符</font>

```java
@Test
public void testJsonQuery() {
    String s = "{name: 'kato·megumi0', age: 19}";
    Query query = new BasicQuery(s);
    List<Employee> employee2 = mongoTemplate.find(query, Employee.class);
    System.out.println(employee2);
}
```

**ps**：目前推荐json使用map来进行封装

#### 9.3.4 更新文档

>   db.集合名称.update(
>      <query 更新条件>,
>      <update更新动作及更新内容>,
>      <options更新选项>
>   )
>
>   下面都是update的衍生方法：
>
>   *   updateOne：更新匹配的第一条文档，相当update的multi=false
>   *   updateMany：更新所有匹配的文档，相当update的multi=true
>   *   replaceOne：替换单个文档，不存在时不创建，相当update的upsert=false
>
>   在mongo shell文档更新涉及到更新操作符和操作数据两项。
>
>   ****
>
>   在java中提供的更新api：
>
>   *   updateFirst：只更新满足条件的第一条记录，其就是db.集合.updateOne
>   *   updataMulti：更新所有满足条件的记录，db.集合.updateMany
>   *   upset：不符合条件的记录则插入，就是db.集合.replaceOne
>
>   ****
>
>   更新操作需要：
>
>   1.   Query，Criteria：匹配条件
>   2.   Update：更新操作及其数据



```java
@Test
public void testUpdateFirst() {
    // 更新name=kato·megumi0的salary位52000
    Query query = new Query(Criteria.where("name").is("kato·megumi0"));
    Update update = new Update();
    update.set("salary", 52000);
    UpdateResult updateResult = mongoTemplate.updateFirst(query, update, Employee.class);
    System.out.println(updateResult.getModifiedCount());
}

@Test
public void testUpdateMulti() {
    // 更新name=kato·megumi0的salary位52000
    Query query = new Query(Criteria.where("name").regex("kato·megumi*"));
    Update update = new Update();
    update.set("salary", 52000);
    UpdateResult updateResult = mongoTemplate.updateMulti(query, update, Employee.class);
    System.out.println(updateResult.getModifiedCount());
}
```

****

#### 9.3.5 添加文档

>   mongo客服端：db.集合名称.insert、db.集合名称.insertOne、db.集合名称.insertMany.
>
>   *   java中调用insert方法，insert是一个重载方法可以插入单条或多条文档。
>   *   save方法（不推荐使用）
>
>   insert方法插入文档，里面包含了新增后id的值。`如果集合不存在会自动创建集合`。还可以给集合中多加一个_class的属性（默认会添加），存储新增时document对应java中类的全限定路径。这么做为了查询时能把document转化为对应的java类对象。

```java
@Test
public void testInsertDoc() {
    Employee emp1 = new Employee(1L, "kato", 21, 13000.0, new Date());
    Employee empldb = mongoTemplate.insert(emp1);
    System.out.println(empldb);
}
```

- **insert:** 插入重复数据时：`insert`报`DuplicateKeyException`提示主键重复；`save`对已存在的数据进行更新。
- **save:** 批处理操作时：`insert`可以一次性插入整个数据，效率较高；`save`需遍历整个数据，一次插入或更新，效率较低。



##### 9.3.6 删除文档

>   删除文档也需要条件，所有必然用到Query，和Criteria类

```java
@Test
public void testDelete(){
  // 删除所有，不如用db.dropCollection()
  mongoTemplate.remove(new Query(), User.class);
  // 条件删除
  mongoTemplate.remove(
    Query.query(Criteria.where("name").is("xxx")), Employee.class);
}
```

### 9.4 MongoRepository

>   MongoRepository是spring data操作数据库的规范，使用这套规划也可以方便的操作mongodb，这种方式和操作mysql的mybatis-plus类似，都是dao模型，其提供多种操作mongodb的方法（框架自己实现接口）。
>
>   但是MongoRepository实现了的只是最基本的增删改查的功能，要想增加额外的查询方法，可以按照以下规则定义接口的方法。
>
>   1.   格式为“findBy+字段名+方法后缀”。
>
>   MongoRepository这个方便对实体类映射进行简单的`增删改查`以及`分页`，`查询支持@Query注解，增删改不支持注解`，只有自带的方法不支持多表(Lookup)+聚合(aggregate)+分布式计算(MapReduce)
>   这里只演示查询就行.

**1. 持久化类**

```java
@Document(collection = "grade")//对应表名
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Grade
{
    @Id//主键
    private String id;
    private Integer grade_id;
    private String grade_name;
    private List<Student> student_list;
}
```

```java
@Document(collection = "student")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Student
{
    @Id//主键
    private String id;
    @Field("stu_name") //对应列名
    private String stu_name;
    private Integer age;
    private Integer grade_id;
    private Grade grade;
}
```



**2. dao接口**

```java
/**
 * MongoRepository需要指定两个泛型：
 * 泛型1：orm对应实例
 * 泛型2：实体类对应主键类型
 * 不需要加spring注解
 */
public interface StudentRepository extends MongoRepository<Student, String> {
    @Query("{'stu_name':?0}}")
    List<Student> findStudentsByStu_name(String stu_name);

    @Query("{'stu_name':{$regex:'?0'}}")
    List<Student> findStudentsByStu_nameBetween(String stu_name);

    @Query("{'stu_name':{$regex:'^?0'}}")
    List<Student> findStudentsByStu_nameStartingWith(String stu_name);

    @Query("{$and:[{'stu_name':?0},{'age':?1}]}")
    List<Student> findStudentsByStu_nameaAndAge(String stu_name,Integer age);
}
```

```java
public interface GradeRepository extends MongoRepository<Grade, String> {
 
}
```

**3.Test 插入**

```java
@Test
public void select() {
    // 查询所有 *
    studentRepository.findAll().forEach(System.out::println);
    // 查询所有并按照年龄排序
    Sort sort = new Sort(Sort.Direction.ASC, "age");
    studentRepository.findAll(sort).forEach(System.out::println);
    // 分页 每页2条 查询第二页
    PageRequest pageRequest = new PageRequest(1, 2);//page 0第一页 1第二页
    studentRepository.findAll(pageRequest).forEach(System.out::println);
    // 根据名称查询 stu_name='六豹子'
    studentRepository.findStudentsByStu_name("六豹子").forEach(System.out::println);
    // 模糊查询 stu_name like '%豹%'
    studentRepository.findStudentsByStu_nameBetween("豹").forEach(System.out::println);
    // 模糊查询以什么开头 stu_name like '豹%'
    studentRepository.findStudentsByStu_nameStartingWith("豹").forEach(System.out::println);
    // 查询 stu_name=六豹子 and age=13
    studentRepository.findStudentsByStu_nameaAndAge("六豹子",13).forEach(System.out::println);
}
```







## 10，副本集

### 10.1 说明

https://docs.mongodb.com/manual/replication/

MongoDB 副本集（Replication Set）是有自动故障恢复功能的主从集群，有一个Primary节点和一个或多个Secondary节点组成。副本集没有固定的`主节点`, 当`主节点`发生故障时整个集群会`选举一个主节点`为系统提供服务以保证系统的高可用。

复制集提供冗余和高可用性，是所有生产部署的基础。

![](assets/replica-set-read-write-operations-primary.bakedsvg-20211214132813414.svg)

​	   自动故障转移机制: 当主节点未与集合的其他成员通信超过配置的选举超时时间（默认为 10 秒）时，合格的辅助节点将调用选举以将自己提名为新的主节点。集群尝试完成新主节点的选举并恢复正常操作。

在实现高可用的同时，复制集实现了其他几个附加作用:

-   数据分发: 将数据从一个区域复制到另一个区域，减少另一个区域的读延迟
-   读写分离: 不同类型的压力分别在不同的节点上执行
-   异地容灾: 在数据中心故障时候快速切换到异地

![](assets/replica-set-trigger-election.bakedsvg.svg)

### 10.2 搭建副本集

- **安装 3 个mongodb**

- 修改配置文件

  配置replication.replSetName复制集名称，表名是mongodb的复制集。
  
  分发到各个节点，每个复制集的名称都是相同的。
  
  ```shell
  # ...
  replication:
    # 复制集名称
    replSetName: re0
  ```

-  **启动服务，配置复制集**

  连接任意节点，开始配置复制集，启动服务时通过配置文件只是知道了该节点是属于哪个复制集，但是其他节点在哪还未知道，需要做以下配置。

  **方式一**：

  ```js
  # mongo --port 28017
  # 初始化复制集
  > rs.initiate() 
  # 将其余成员添加到复制集
  > rs.add("hadoop102:27017") 
  > rs.add("hadoop103:27017")
  ```

  **方式二**：

  ```json
  > var config = { 
  		_id:"myreplace", 
  		members:[
  		{_id:0,host:"hadoop101:27017"},
  		{_id:1,host:"hadoop102:27017"},
  		{_id:2,host:"hadoop103:27017"}]
  }
  > rs.initiate(config);
  ```

-  **设置客户端临时可以访问**

  ```sql
  > rs.secondaryOk();
  ```

*   **复制集状态查询**

    ```js
    rs.status()
    ```

    ![](assets/41295)

>   members 中一列体现了所有复制集成员的状态，主要如下：
>
>   health：成员是否健康，通过心跳进行检测。
>
>   state/stateStr：成员的状态，PRIMARY表示主节点，而SECONDARY则表示备节点，如果节点出现故障，则可能出现一些其他的状态，例如RECOVERY。
>
>   uptime：成员的启动时间。
>
>   optime/optimeDate：成员最后一条同步oplog的时间。
>
>   optimeDurable/optimeDurableDate：成员最后一条同步oplog的时间。
>
>   pingMs：成员与当前节点的ping时延。
>
>   syncingTo：成员的同步来源。

*   **查看当前节点角色**

    ```js
    db.isMaster()
    ```

### 10.3 **复制集命令**

| **命令**                           | **描述**                                                 |
| ---------------------------------- | -------------------------------------------------------- |
| rs.add()                           | 为复制集新增节点                                         |
| rs.addArb()                        | 为复制集新增一个 arbiter                                 |
| rs.conf()                          | 返回复制集配置信息                                       |
| rs.freeze()                        | 防止当前节点在一段时间内选举成为主节点                   |
| rs.help()                          | 返回 replica set 的命令帮助                              |
| rs.initiate()                      | 初始化一个新的复制集                                     |
| rs.printReplicationInfo()          | 以主节点的视角返回复制的状态报告                         |
| rs.printSecondaryReplicationInfo() | 以从节点的视角返回复制状态报告                           |
| rs.reconfig()                      | 通过重新应用复制集配置来为复制集更新配置                 |
| rs.remove()                        | 从复制集中移除一个节点                                   |
| rs.secondaryOk()                   | 为当前的连接设置 从节点可读                              |
| rs.status()                        | 返回复制集状态信息。                                     |
| rs.stepDown()                      | 让当前的 primary 变为从节点并触发 election               |
| rs.syncFrom()                      | 设置复制集节点从哪个节点处同步数据，将会覆盖默认选取逻辑 |

### 10.4 集群之安全认证

*   **创建用户**

    ```js
    use admin
    #创建用户
    db.createUser({
        user: "root",
        pwd: "123456",
         roles: [ 
             { role: "clusterAdmin", db: "admin" } ,
             { role: "userAdminAnyDatabase", db: "admin"},
             { role: "userAdminAnyDatabase", db: "admin"},
             { role: "readWriteAnyDatabase", db: "admin"}]
    })
    ```

*   **创建 KeyFile 文件**

    >   keyFile文件的作用： 集群之间的安全认证，增加安全认证机制KeyFile（开启keyFile认证就默认开启了auth认证了）。
    >
    >   **注意：创建keyFile前，需要先停掉复制集中所有主从节点的mongod服务，然后再创建，否则有可能出现服务启动不了的情况。**

    ```js
    # mongo.key 采用随机算法生成，用作节点内部通信的密钥文件。
    openssl rand -base64 756 > /data/mongo.key
    # 权限必须是600
     chmod 600 /data/mongo.key  
    ```

*   **启动 mongod**

    ```js
    ./mongod -f ../conf/mongodb.conf --keyFile ../data/mongo.key
    ```

### 10.5 复制集连接

#### 方式1：直接连接 Primary 节点

>   直接连接 Primary 节点，正常情况下可读写 MongoDB，但主节点故障切换后，无法正常访问。

![](assets/40973)

#### 方式2：URL 高可用连接

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://root:123456@hadoop101:27017,hadoop102:27017,hadoop103:27017/test?authSource=admin&replicaSet=rs0
```



## 11，分片集群

### 说明

​	https://docs.mongodb.com/manual/sharding/

​	`分片(sharding)`是指`将数据拆分,将其分散存在不同机器的过程`，有时也用`分区(partitioning)`来表示这个概念,将数据分散在不同的机器上，不需要功能强大的大型计算机就能存储更多的数据，处理更大的负载。

​	分片目的是通过分片能够增加更多机器来应对不断的增加负载和数据，还不影响应用运行。

​	MongoDB支持`自动分片`,可以摆脱手动分片的管理困扰，集群自动切分数据做负载均衡。MongoDB分片的基本思想就是将集合拆分成多个块，这些快分散在若干个片里，每个片只负责总数据的一部分，应用程序不必知道哪些片对应哪些数据，甚至不需要知道数据拆分了，所以在分片之前会运行一个路由进程，mongos进程，这个路由器知道所有的数据存放位置，应用只需要直接与mongos交互即可。mongos自动将请求转到相应的片上获取数据，从应用角度看分不分片没有什么区别。

### 架构

![image-20211214150523282](assets/image-20211214150523282.png)

- **Shard:**	用于存储实际的数据块，实际生产环境中一个shard server角色可由几台机器组个一个replica set承担，防止主机单点故障

- **Config Server**:mongod实例，存储了整个 ClusterMetadata。

- **Query Routers**: 前端路由，客户端由此接入，且让整个集群看上去像单一数据库，前端应用可以透明使用。
- **Shard Key**: 片键，设置分片时需要在集合中选一个键,用该键的值作为拆分数据的依据,这个片键称之为(shard key)，片键的选取很重要,片键的选取决定了数据散列是否均匀。

### 搭建

```markdown
# 1.集群规划
- Shard Server 1：27017
- Shard Repl   1：27018

- Shard Server 2：27019
- Shard Repl   2：27020

- Shard Server 3：27021
- Shard Repl   3：27022

- Config Server ：27023
- Config Server ：27024
- Config Server ：27025

- Route Process ：27026

# 2.进入安装的 bin 目录创建数据目录
- mkdir -p ../cluster/shard/s0
- mkdir -p ../cluster/shard/s0-repl

- mkdir -p ../cluster/shard/s1
- mkdir -p ../cluster/shard/s1-repl

- mkdir -p ../cluster/shard/s2
- mkdir -p ../cluster/shard/s2-repl

- mkdir -p ../cluster/shard/config1
- mkdir -p ../cluster/shard/config2
- mkdir -p ../cluster/shard/config3

# 3.启动4个 shard服务

# 启动 s0、r0
> ./mongod --port 27017 --dbpath ../cluster/shard/s0 --bind_ip 0.0.0.0 --shardsvr --replSet r0/121.5.167.13:27018
> ./mongod --port 27018 --dbpath ../cluster/shard/s0-repl --bind_ip 0.0.0.0 --shardsvr --replSet r0/121.5.167.13:27017
-- 1.登录任意节点
-- 2. use admin
-- 3. 执行
		config = { _id:"r0", members:[
      {_id:0,host:"121.5.167.13:27017"},
      {_id:1,host:"121.5.167.13:27018"},
    	]
    }
		rs.initiate(config);//初始化

# 启动 s1、r1
> ./mongod --port 27019 --dbpath ../cluster/shard/s1 --bind_ip 0.0.0.0 --shardsvr  --replSet r1/121.5.167.13:27020
> ./mongod --port 27020 --dbpath ../cluster/shard/s1-repl --bind_ip 0.0.0.0 --shardsvr --replSet r1/121.5.167.13:27019
-- 1.登录任意节点
-- 2. use admin
-- 3. 执行
		config = { _id:"r1", members:[
      {_id:0,host:"121.5.167.13:27019"},
      {_id:1,host:"121.5.167.13:27020"},
    	]
    }
		rs.initiate(config);//初始化

# 启动 s2、r2
> ./mongod --port 27021 --dbpath ../cluster/shard/s2 --bind_ip 0.0.0.0 --shardsvr --replSet r2/121.5.167.13:27022
> ./mongod --port 27022 --dbpath ../cluster/shard/s2-repl --bind_ip 0.0.0.0 --shardsvr --replSet r2/121.5.167.13:27021
-- 1.登录任意节点
-- 2. use admin
-- 3. 执行
		config = { _id:"r2", members:[
      {_id:0,host:"121.5.167.13:27021"},
      {_id:1,host:"121.5.167.13:27022"},
    	]
    }
		rs.initiate(config);//初始化

# 4.启动3个config服务

> ./mongod --port 27023 --dbpath ../cluster/shard/config1 --bind_ip 0.0.0.0 --replSet  config/[121.5.167.13:27024,121.5.167.13:27025] --configsvr

> ./mongod --port 27024 --dbpath ../cluster/shard/config2 --bind_ip 0.0.0.0 --replSet  config/[121.5.167.13:27023,121.5.167.13:27025] --configsvr

> ./mongod --port 27025 --dbpath ../cluster/shard/config3 --bind_ip 0.0.0.0 --replSet  config/[121.5.167.13:27023,121.5.167.13:27024] --configsvr

# 5.初始化 config server 副本集
- `登录任意节点 congfig server`
> 1.use admin 
> 2.在admin中执行
  config = { 
      _id:"config", 
      configsvr: true,
      members:[
          {_id:0,host:"121.5.167.13:27023"},
          {_id:1,host:"121.5.167.13:27024"},
          {_id:2,host:"121.5.167.13:27025"}
        ]
  }
> 3.rs.initiate(config); //初始化副本集配置 

# 6.启动 mongos 路由服务

> ./mongos --port 27026 --configdb config/121.5.167.13:27023,121.5.167.13:27024,121.5.167.13:27025 --bind_ip 0.0.0.0 

# 7.登录 mongos 服务
> 1.登录 mongo --port 27026
> 2.use admin
> 3.添加分片信息
	db.runCommand({ addshard:"r0/121.5.167.13:27017,121.5.167.13:27018",
	"allowLocal":true });
	db.runCommand({ addshard:"r1/121.5.167.13:27019,121.5.167.13:27020",
	"allowLocal":true });
	db.runCommand({ addshard:"r2/121.5.167.13:27021,121.5.167.13:27022",
	"allowLocal":true });
> 4.指定分片的数据库
	db.runCommand({ enablesharding:"baizhi" });

> 5.设置库的片键信息
	db.runCommand({ shardcollection: "baizhi.users", key: { _id:1}});
	db.runCommand({ shardcollection: "baizhi.emps", key: { _id: "hashed"}})
```

 
