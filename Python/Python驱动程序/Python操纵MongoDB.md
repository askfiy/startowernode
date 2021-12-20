# pymongo

pymongo是MongoDB官方推出的一款驱动程序，适用于Python语言。

```
$ pip3 install pymongo
```

官方文档：[点我跳转](https://pypi.org/project/pymongo/)

# 链接MongoDB

通过pymongo模块链接MongoDB的示例代码：

```
from pymongo import MongoClient

host = "192.168.0.120"
port = "27017"
user = "root"
password = "123"
auth_source = "admin"

client = MongoClient(
    f"mongodb://{user}:{password}@{host}:{port}/?authSource={auth_source}")

with client:
    db = client["db1"]
    print(db.name)
    # db1
```



# 库的操作

## 相关API

库层面操作的相关API如下所示：

```
- list_database_names
   查看所有库的名字
   
- list_databases
   查看所有库
   
- get_database
   获取某个库
   
- get_default_database
   获取默认库
   
- drop_database
   删除某个库
```

## 查看库

查看当前MongoDB实例上的所有库的名字：

```
with client:
    print(client.list_database_names())
    
# ['admin', 'config', 'db1', 'local']
```

查看当前MongoDB实例上的所有库及其基础状态：

```
with client:
    db_lst = client.list_databases()
    for db in db_lst:
        print(db)

# ['admin', 'config', 'db1', 'local']
# {'name': 'admin', 'sizeOnDisk': 135168.0, 'empty': False}
# {'name': 'config', 'sizeOnDisk': 12288.0, 'empty': False}
# {'name': 'db1', 'sizeOnDisk': 73728.0, 'empty': False}
# {'name': 'local', 'sizeOnDisk': 221184.0, 'empty': False}
```



## 获取库

获取库有3种方法：

```
client.库名
client["库名"]
client.get_database("库名")
```

个人比较喜欢第2种方式，如下所示：

```
with client:
    admin = client.admin
    test = client["test"]
    local = client.get_database("local")
    print(admin.name)
    print(test.name)
    print(local.name)
    
# admin
# test
# local
```



## 创建库

创建库的语法：

```
client.新库名
client["新库名"]
```

示例如下：

```
with client:
    db1 = client.db1
    db2 = client["db2"]
    print(db1.name)
    print(db2.name)

# db1
# db2
```



## 删除库

删除库可以使用drop_database()方法，如下所示：

```
with client:
    client.drop_database("db2")
```

删除后并不会有任何返回结果，因为在MongoDB中即使删除一个不存在的库也不会抛出异常。





# 集合操作

## 相关API

集合层面操作的相关API如下所示：

```
- list_collection_names
   查看所有集合的名字
   
- list_collections
   查看所有集合
   
- get_collection
   获取某个集合

- create_collection
   创建某个集合

- drop_collection
   删除某个集合
```



## 查看集合

查看当前库中所有集合的名字：

```
with client:
    admin = client.get_database("admin")
    print(admin.list_collection_names())

# ['system.users', 'system.version']
```

查看当前库中所有集合及其基础状态：

```
with client:
    admin = client.get_database("admin")
    collection_lst = admin.list_collections()
    for collection in collection_lst:
        print(collection)

# {'name': 'system.users', 'type': 'collection', 'options': {}, 'info': {'readOnly': False, 'uuid': Binary(b'j\x04\xe5\rd\x0eM\xb4\xa0T\xa0L\x90>b\xe7', 4)}, 'idIndex': {'v': 2, 'key': {'_id': 1}, 'name': '_id_'}}
# {'name': 'system.version', 'type': 'collection', 'options': {}, 'info': {'readOnly': False, 'uuid': Binary(b'\xdc\xf5qj\xed!I\xf1\xbe\x07N\xaf\xcf\xc1\xab\xe9', 4)}, 'idIndex': {'v': 2, 'key': {'_id': 1}, 'name': '_id_'}}
```



## 获取集合

获取集合有3种方法：

```
库对象.集合名
库对象["集合名"]
库对象.get_collection("集合名")
```

个人比较喜欢第2种方式，如下所示：

```
with client:
    admin = client.get_database("admin")
    user = admin["user"]
    version = admin.get_collection("version")
    print(user.name)
    print(version.name)

# user
# version
```



## 创建集合

创建集合的语法：

```
库对象.新集合名
库对象["新库名"]
库对象.create_collection("新集合名")
```

示例如下：

```
with client:
    db1 = client.get_database("db1")
    user = db1.create_collection("user")
    print(user.name)

# user
```



## 删除集合

删除集合可以使用drop_collection()方法，如下所示：

```
with client:
    db1 = client.get_database("db1")
    db1.drop_collection("user")
```

删除后并不会有任何返回结果，因为在MongoDB中即使删除一个不存在的集合也不会抛出异常。



# 文档操作

## 相关API

文档操作的相关API如下所示：

```
- insert_one
   插入单行记录

- insert_many
   插入多行记录
   
- update_one
   更新单行记录

- update_many
   更新多行记录
   
- delete_one
   删除单行记录
   
- delete_many
   删除多行记录
```





## 插入文档

更新记录可以使用insert_one()或者insert_many()，他们的区别在于：

- insert_one()：仅插入第一个文档
- insert_many()：可插入多个文档

如果要插入时间类型，则需要使用datetime模块，下面使用insert_many()进行示例：

```
from pymongo import MongoClient
import datetime

host = "192.168.0.120"
port = "27017"
user = "root"
password = "123"
auth_source = "admin"

client = MongoClient(
    f"mongodb://{user}:{password}@{host}:{port}/?authSource={auth_source}")

with client:
    db1 = client.get_database("db1")
    user_info = db1.get_collection("userInfo")
    user_info.insert_many(
        [
            {
                "name": "Jack",
                "age": 18,
                "gender": 1,
                "create_time": datetime.datetime(2016, 8, 15),
                "grades": [
                    {"type": "mysql", "score": 92},
                    {"type": "redis", "score": 78},
                    {"type": "mongodb", "score": 32},
                ]
            },
            {
                "name": "Tom",
                "age": 20,
                "gender": 1,
                "create_time": datetime.datetime(2013, 4, 5),
                "grades": [
                    {"type": "mysql", "score": 88},
                    {"type": "redis", "score": 68},
                    {"type": "mongodb", "score": 54},
                ]
            },
            {
                "name": "Mary",
                "age": 18,
                "gender": 0,
                "create_time": datetime.datetime(2012, 2, 3),
                "grades": [
                    {"type": "mysql", "score": 91},
                    {"type": "redis", "score": 89},
                    {"type": "mongodb", "score": 72},
                ]
            }

        ]
    )

```



## 更新文档

更新文档可以使用update_one()或者update_many()，他们的区别在于：

- update_one()：仅更新匹配到的第一个文档
- update_many()：可更新匹配到的多个文档

下面使用update_many()进行示例：

```
with client:
    db1 = client.get_database("db1")
    user_info = db1.get_collection("userInfo")
    result = user_info.update_many(
        {"name": "Jack"}, {"$inc": {"age": +1}})
    # 成功匹配的文档数量
    print(result.matched_count)
    # 成功修改的文档数量
    print(result.modified_count)
```



## 删除文档

删除文档可以使用delete_one()或者delete_many()，他们的区别在于：

- delete_one()：仅删除掉匹配到的第一个文档
- delete_many()：可删除掉匹配到的多个文档

下面使用delete_many()进行示例：

```
with client:
    db1 = client.get_database("db1")
    user_info = db1.get_collection("userInfo")
    result = user_info.delete_many(
        {"name": "Jack"})
    # 成功删除的文档数量
    print(result.deleted_count)
```



# 文档查询

## 相关API

文档查询的相关API如下所示：

```
- find
  查询符合条件的所有文档，如不指定查询条件则返回全部文档
  
- find_one
  只查询一条文档

- aggregate
  聚合查询
  
- count_documents
  查询文档数量
  
- distinct
  获取所有文档中的某一个key
```



## 文档查询

普通的文档查询可使用find()以及find_one()进行，他们的区别在于：

- find_one()：仅返回匹配到的第一个文档
- find()：返回全部匹配到的文档

如下所示：

```
result = user_info.find({"grades.0.score": {"$gt": 25}}, {
                            "name": 1, "_id": 0})
```

查询完成后的result对象可调用的一些方法或者属性如下：

```
- cursor_id
- distinct
- explain
- hint
- limit
- max
- min
- skip
- sort
- where
```

示例演示，查询所有mysql成绩大于25的学生：

```
with client:
    db1 = client.get_database("db1")
    user_info = db1.get_collection("userInfo")
    result = user_info.find({"grades.0.score": {"$gt": 25}}, {
                            "name": 1, "_id": 0})
    for row in result:
        print(row)

# {'name': 'Tom'}
# {'name': 'Mary'}
```

其实pymongo的查询接口和原生语句都差不多，可以参照MongoDB系列的查询文章。



## 聚合查询

聚合使用aggregate()方法进行，在内部指定管道和步骤，如下所示：

```
with client:
    db1 = client.get_database("db1")
    user_info = db1.get_collection("userInfo")
    # 以性别分组，查询年龄大于18的组
    result = user_info.aggregate(
        [
            # 步骤1
            {
                "$group": {
                    # 指定聚合条件
                    "_id": "$gender",
                    "avg_age": {"$avg": "$age"}
                }
            },
            # 步骤2
            {
                "$match": {
                    "avg_age": {"$gt": 18}
                }
            }
        ]
    )
    for row in result:
        print(row)

# {'_id': 1, 'avg_age': 20.0}
```

具体的聚合查询可参照MongoDB系列中的聚合查询文章。

# 索引相关

## 相关API



```
- create_index
   创建索引

- create_indexes
   创建多个索引
   
- index_information
  查询索引相关信息
  
- list_indexes
  列出所有的索引

- drop_index
   删除索引

- drop_indexes
   删除多个索引
```





## 创建索引

查询索引可使用create_index()或者create_indexes()方法，如下所示：

```
with client:
    db1 = client.get_database("db1")
    user_info = db1.get_collection("userInfo")
    user_info.create_index([("grades.type", pymongo.DESCENDING)],
                              background=True)
```



## 查询索引

查询索引可使用list_indexes()或者index_information()方法，如下所示：

```
with client:
    db1 = client.get_database("db1")
    user_info = db1.get_collection("userInfo")
    print(user_info.index_information())

# {'_id_': {'v': 2, 'key': [('_id', 1)]}, 'grades.type_-1': {'v': 2, 'key': [('grades.type', -1)], 'background': True}}
```



## 删除索引

删除索引可使用drop_index()或者drop_indexs()方法，如下所示：

```
with client:
    db1 = client.get_database("db1")
    user_info = db1.get_collection("userInfo")
    user_info.drop_index(index_or_name="grades.type_-1")
```

