# pymysql

## 基本使用

pymysql是一款第三方的Python模块，它可以帮助我们链接MySQL：

```
$ pip3 install pymysql
```

以下是它的基本使用：

```
import pymysql

# 链接信息
link_information = {
    "host": "192.168.0.120",
    "user": "Jack",
    "password": "123",
    "database": "db1",
    "charset": "utf8mb4",
    "cursorclass": pymysql.cursors.DictCursor,  # 结果以字典形式呈现
     "autocommit": False,  # 关闭自动提交
}

# 获取链接
conn = pymysql.connect(**link_information)

# 获取游标
cursor = conn.cursor()

# 定义语句, ?为占位符
sql_statement = "SELECT * FROM studentsInfo WHERE id IN (%s, %s)"
# 执行语句，返回受影响的行数，传入tuple格式化占位符，防止sql注入
affected_row = cursor.execute(sql_statement, (1, 2))
# 结果信息
result = cursor.fetchall()

# 打印结果
print(affected_row)
# 2
print(result)
# [
#     {'id': 1, 'name': 'Jack', 'gender': 'male', 'age': 17, 'fk_student_id': 1, 'fk_class_id': 2},
#     {'id': 2, 'name': 'Tom', 'gender': 'male', 'age': 18, 'fk_student_id': 2, 'fk_class_id': 1}
# ]

# 关闭游标和链接
cursor.close()
conn.close()
```



## 结果获取

pymysql中对于结果集而言，是存在一个游标的概念的，它相当于当前记录的指针，可以上下移动。

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/202112071722672.gif" alt="cursor" style="zoom: 50%;" />

当pymysql执行完成SQL语句后，可用下面3个方法从由表中获取记录结果：

```
- cursor.fetchone()
	返回第一条记录，游标向下移动一行，默认返回一个Tuple
	若指定了cursorclass为pymysql.cursors.DictCursor，则返回一个Dict

- cursor.fetchmany(2)
	获取接下来的两条记录，游标向下移动两行，默认返回一个List[Tuple...]
	若指定了cursorclass为pymysql.cursors.DictCursor，则返回一个List[Dict...]

- cursor.fetchall()
	获取全部记录，游标移动到末尾，默认返回一个List[Tuple...]
	若指定了cursorclass为pymysql.cursors.DictCursor，则返回一个List[Dict...]
```

也可以通过下面的方法改变游标的位置：

```
- cursor.scroll(3, mode='absolute')
	游标以绝对位置向下移动3条记录

- cursor.scroll(3, mode='relative')
	游标以相对位置向上移动3条记录
	
# 如果移动值为负数，则代表向上移动
```





## SQL注入

在提交某些需要格式化的SQL语句时，一定要使用cursor.execute()中的方法第二参数args格式化SQL语句，而不是利用str.format()或者f-string进行格式化。

这会避免SQL注入带来的安全隐患。

另外，在使用cursor.execute()格式化SQL语句时，不能格式化table-name、database-name，否则会抛出语法错误，这是因为execute()在格式化SQL语句时会自动加上`号。



## 事务提交

在执行UPDATE、INSERT、DELETE等TCL事务控制语句时，必须使用conn.commit()或conn.rollback()方法提交或撤销事务语句：

```
sql_statement = "UPDATE studentsInfo SET age = age + 1 WHERE id = %s"
affected_row = cursor.execute(sql_statement, (1, ))
conn.commit()
# conn.rollback()
```

或者你也可以在实例化conn对象时为它传入autocommit=true的参数：

```
link_information = {
    "host": "192.168.0.120",
    "user": "Jack",
    "password": "123",
    "database": "db1",
    "charset": "utf8mb4",
    "cursorclass": pymysql.cursors.DictCursor,  # 结果以字典形式呈现
    "autocommit": True,
}

# 获取链接
conn = pymysql.connect(**link_information)
```





## 提交多条

使用cursor.executemany()方法可以一次性提交多条SQL语句，这在INSERT时非常常用：

```
import pymysql

# 链接信息
link_information = {
    "host": "192.168.0.120",
    "user": "Jack",
    "password": "123",
    "database": "db1",
    "charset": "utf8mb4",
    "cursorclass": pymysql.cursors.DictCursor,  # 结果以字典形式呈现
    "autocommit": False,
}

# 获取链接
conn = pymysql.connect(**link_information)

# 获取游标
cursor = conn.cursor()

# 定义语句, ?为占位符
sql_statement = "INSERT INTO userInfo(name, gender, age) VALUES(%s, %s, %s)"
user_info = (
    ("Jack", 1, 18),
    ("Mary", 0, 17),
    ("Ken", 1, 21)
)
# 将会自动执行3次
affected_row = cursor.executemany(sql_statement, user_info)
conn.commit()
# 结果信息
result = cursor.fetchall()
# 最后一条行号

# 打印结果
print(affected_row)
# 2

print(result)
# ()

# 插入完成后，最后一条记录所在的行号
print(cursor.lastrowid)
# 3

# 关闭游标和链接
cursor.close()
conn.close()
```





# dbutils

## 链接池

使用dbutils的链接池来与MySQL服务端建立链接，可以有效避免频繁的socket链接请求，在多线程场景下适用。

因为pymysql在多线程下必须与MySQL服务器沟通多次，每一次创建1条链接，而dbutils可以一次创建多条链接供所有线程使用，所以它的网络I/O开销会比较小。

下载dbutils模块：

```
$ pip3 install dbutils
```



## 基本使用

如下所示：

```
import pymysql
from dbutils.pooled_db import PooledDB

link_information = {
    "host": "192.168.0.120",
    "user": "Jack",
    "password": "123",
    "database": "db1",
    "charset": "utf8mb4",
    "cursorclass": pymysql.cursors.DictCursor,  # 结果以字典形式呈现
    "autocommit": False,
    # dbutils依赖于pymysql模块发起链接
    "creator": pymysql,
    # 初始化时，链接池中至少创建的空闲的链接，0表示不创建
    "mincached": 10,
    # 链接池中最多共享的链接数量，0和None表示全部共享
    "maxshared": 0,
    # 一个链接最多被重复使用的次数，None表示无限制
    "maxusage": 0,
    # 链接池中最多闲置的链接，0和None不限制
    "maxcached": 0,
    # 连接池允许的最大连接数，0和None表示不限制连接数
    "maxconnections": 70,
    # 连接池中如果没有可用连接后，是否阻塞等待。True，等待；False，不等待然后报错
    "blocking": True,
    # 开始会话前执行的命令列表
    "setsession": [],
}
# 确保链接池只会初始化一次
pool = PooledDB(**link_information)

if __name__ == "__main__":
    conn = pool.connection()
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM userInfo;")
    print(cursor.fetchall())
    cursor.close()
    conn.close()
```

