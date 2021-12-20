# SQLAlchemy

SQLAlchemy是Python中一款非常优秀的ORM框架，它可以与任意的第三方web框架相结合，如flask、tornado、django、fastapi等。

SQLALchemy相较于Django ORM来说更贴近原生的SQL语句，因此学习难度较低。

SQLALchemy由以下5个部分组成：

- Engine：框架引擎
- Connection Pooling：数据库链接池
- Dialect：数据库DB API种类
- Schema/Types：数据库架构类型
- SQL Exprression Language：SQL表达式语言

图示如下：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/202112072056784.png" alt="img" style="zoom:40%;" />

运行流程：

- 首先用户输入的操作会交由ORM对象
- 接下来ORM对象会将用户操作提交给SQLALchemy Core
- 其次该操作会由Schema/Types以及SQL Expression Language转换为SQL语句
- 然后Egine会匹配用户已经配置好的egine，并从链接池中去取出一个链接
- 最终该链接会通过Dialect调用DBAPI，将SQL语句转交给DBAPI去执行

下载SQLALchemy：

```
$ pip3 install sqlalchemy
```

值得注意的是，SQLAlchemy必须依赖其他操纵数据库的模块才能进行使用，也就是上面提到的DBAPI。

SQLAlchemy配合DBAPI使用时，链接字符串也有所不同，如下所示：

```
MySQL-Python
    mysql+mysqldb://<user>:<password>@<host>[:<port>]/<dbname>

pymysql
    mysql+pymysql://<username>:<password>@<host>/<dbname>[?<options>]

MySQL-Connector
    mysql+mysqlconnector://<user>:<password>@<host>[:<port>]/<dbname>

cx_Oracle
    oracle+cx_oracle://user:pass@host:port/dbname[?key=value&key=value...]
```





# 创建单表

SQLAlchemy不允许修改表结构，如果需要修改表结构则必须删除旧表，再创建新表，或者执行原生的SQL语句ALERT TABLE进行修改。

这意味着在使用非原生SQL语句修改表结构时，表中已有的所有记录将会丢失，所以我们最好一次性的设计好整个表结构避免后期修改：

```
# models.py
import datetime
from sqlalchemy.orm import sessionmaker
from sqlalchemy.orm import scoped_session

from sqlalchemy import (
    create_engine,
    Column,
    Integer,
    String,
    Enum,
    DECIMAL,
    DateTime,
    Boolean,
    UniqueConstraint,
    Index
)
from sqlalchemy.ext.declarative import declarative_base

# 基础类
Base = declarative_base()

# 创建引擎
engine = create_engine(
    "mysql+pymysql://tom:123@192.168.0.120:3306/db1?charset=utf8mb4",
    # "mysql+pymysql://tom@127.0.0.1:3306/db1?charset=utf8mb4", # 无密码时
    # 超过链接池大小外最多创建的链接
    max_overflow=0,
    # 链接池大小
    pool_size=5,
    # 链接池中没有可用链接则最多等待的秒数，超过该秒数后报错
    pool_timeout=10,
    # 多久之后对链接池中的链接进行一次回收
    pool_recycle=1,
    # 查看原生语句（未格式化）
    echo=True
)

# 绑定引擎
Session = sessionmaker(bind=engine)
# 创建数据库链接池，直接使用session即可为当前线程拿出一个链接对象conn
# 内部会采用threading.local进行隔离
session = scoped_session(Session)


class UserInfo(Base):
    """ 必须继承Base """
    # 数据库中存储的表名
    __tablename__ = "userInfo"
    # 对于必须插入的字段，采用nullable=False进行约束，它相当于NOT NULL
    id = Column(Integer, primary_key=True, autoincrement=True, comment="主键")
    name = Column(String(32), index=True, nullable=False, comment="姓名")
    age = Column(Integer, nullable=False, comment="年龄")
    phone = Column(DECIMAL(6), nullable=False, unique=True, comment="手机号")
    address = Column(String(64), nullable=False, comment="地址")
    # 对于非必须插入的字段，不用采取nullable=False进行约束
    gender = Column(Enum("male", "female"), default="male", comment="性别")
    create_time = Column(
        DateTime, default=datetime.datetime.now, comment="创建时间")
    last_update_time = Column(
        DateTime, onupdate=datetime.datetime.now, comment="最后更新时间")
    delete_status = Column(Boolean(), default=False,
                           comment="是否删除")

    __table__args__ = (
        UniqueConstraint("name", "age", "phone"),  # 联合唯一约束
        Index("name", "addr", unique=True),       # 联合唯一索引
    )

    def __str__(self):
        return f"object : <id:{self.id} name:{self.name}>"


if __name__ == "__main__":
    # 删除表
    Base.metadata.drop_all(engine)
    # 创建表
    Base.metadata.create_all(engine)
```





# 记录操作

## 新增记录

新增单条记录：

```
# 获取链接池、ORM表对象
import models


user_instance = models.UserInfo(
    name="Jack",
    age=18,
    phone=330621,
    address="Beijing",
    gender="male"
)

models.session.add(user_instance)

# 提交
models.session.commit()

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```



## 批量新增

批量新增能减少TCP链接次数，提升插入性能：

```
# 获取链接池、ORM表对象
import models


user_instance1 = models.UserInfo(
    name="Tom",
    age=19,
    phone=330624,
    address="Shanghai",
    gender="male"
)

user_instance2 = models.UserInfo(
    name="Mary",
    age=20,
    phone=330623,
    address="Chongqing",
    gender="female"
)


models.session.add_all(
    (
        user_instance1,
        user_instance2
    )
)

# 提交
models.session.commit()
# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```



## 修改记录

修改某些记录：

```
# 获取链接池、ORM表对象
import models

# 修改的信息：
#  - Jack -> Jack + son
# 在SQLAlchemy中，四则运算符号只能用于数值类型
# 如果是字符串类型需要在原本的基础值上做改变，必须设置
#  - age -> age + 1
# synchronize_session=False

models.session.query(models.UserInfo)\
    .filter_by(name="Jack")\
    .update(
        {
            "name": models.UserInfo.name + "son",
            "age": models.UserInfo.age + 1
        },
        synchronize_session=False
)
# 本次修改具有字符串字段在原值基础上做更改的操作，所以必须添加
# synchronize_session=False
# 如果只修改年龄，则不用添加

# 提交
models.session.commit()
# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```



## 删除记录

删除记录用的比较少，了解即可，一般都是像上面那样增加一个delete_status的字段，如果为1则代表删除：

```
# 获取链接池、ORM表对象
import models

models.session.query(models.UserInfo).filter_by(name="Mary").delete()

# 提交
models.session.commit()
# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```



# 单表查询

## 基本查询

查所有记录、所有字段，all()方法将返回一个列表，内部包裹着每一行的记录对象：

```
# 获取链接池、ORM表对象
import models

result = models.session.query(models.UserInfo)\
    .all()

print(result)
# [<models.UserInfo object at 0x7f4d3d606fd0>, <models.UserInfo object at 0x7f4d3d606f70>]

for row in result:
    print(row)
# object : <id:1 name:Jackson>
# object : <id:2 name:Tom>

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```

查所有记录、某些字段（注意，下面返回的元组实际上是一个命名元组，可以直接通过.操作符进行操作）：

```
# 获取链接池、ORM表对象
import models

result = models.session.query(
    models.UserInfo.id,
    models.UserInfo.name,
    models.UserInfo.age
).all()

print(result)
# [(1, 'Jackson', 19), (2, 'Tom', 19)]

for row in result:
    print(row)
# (1, 'Jackson', 19)
# (2, 'Tom', 19)

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```

只拿第一条记录，first()方法将返回单条记录对象（注意，下面返回的元组实际上是一个命名元组，可以直接通过.操作符进行操作）：

```
# 获取链接池、ORM表对象
import models

result = models.session.query(
    models.UserInfo.id,
    models.UserInfo.name,
    models.UserInfo.age
).first()

print(result)
# (1, 'Jackson', 19)

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```



## AS别名

通过字段的label()方法，我们可以为它取一个别名：

```
# 获取链接池、ORM表对象
import models

result = models.session.query(
    models.UserInfo.name.label("s_name"),
    models.UserInfo.age.label("s_age")
).all()

for row in result:
    print(row.s_name)
    print(row.s_age)


# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```





## 条件查询

一个条件的过滤：

```
# 获取链接池、ORM表对象
import models

result = models.session.query(
    models.UserInfo,
).filter(
    models.UserInfo.name == "Jackson"
).all()

# 上面是Python语句形式的过滤条件，由filter方法调用
# 亦可以使用ORM的形式进行过滤，通过filter_by方法调用
# 如下所示
# .filter_by(name="Jackson").all()
# 个人更推荐使用filter过滤，它看起来更直观，更简单，可以支持 == != > < >= <=等常见符号

# 过滤成功的结果数量
print(len(result))
# 1

# 过滤成功的结果
print(result)
# [<models.UserInfo object at 0x7f11391ea2b0>]

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```

AND查询：

```
# 获取链接池、ORM表对象
import models
# 导入AND
from sqlalchemy import and_

result = models.session.query(
    models.UserInfo,
).filter(
    and_(
        models.UserInfo.name == "Jackson",
        models.UserInfo.gender == "male"
    )
).all()

# 过滤成功的结果数量
print(len(result))
# 1

# 过滤成功的结果
print(result)
# [<models.UserInfo object at 0x7f11391ea2b0>]

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```

OR查询：

```
# 获取链接池、ORM表对象
import models
# 导入OR
from sqlalchemy import or_

result = models.session.query(
    models.UserInfo,
).filter(
    or_(
        models.UserInfo.name == "Jackson",
        models.UserInfo.gender == "male"
    )
).all()

# 过滤成功的结果数量
print(len(result))
# 1

# 过滤成功的结果
print(result)
# [<models.UserInfo object at 0x7f11391ea2b0>]

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```

NOT查询：

```
# 获取链接池、ORM表对象
import models
# 导入NOT
from sqlalchemy import not_

result = models.session.query(
    models.UserInfo,
).filter(
    not_(
        models.UserInfo.name == "Jackson",
    )
).all()

# 过滤成功的结果数量
print(len(result))
# 1

# 过滤成功的结果
print(result)
# [<models.UserInfo object at 0x7f11391ea2b0>]

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```



## 范围查询

BETWEEN查询：

```
# 获取链接池、ORM表对象
import models

result = models.session.query(
    models.UserInfo,
).filter(
    models.UserInfo.age.between(15, 21)
).all()

# 过滤成功的结果数量
print(len(result))
# 1

# 过滤成功的结果
print(result)
# [<models.UserInfo object at 0x7f11391ea2b0>]

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```





## 包含查询

IN查询：

```
# 获取链接池、ORM表对象
import models

result = models.session.query(
    models.UserInfo,
).filter(
    models.UserInfo.age.in_((18, 19, 20))
).all()

# 过滤成功的结果数量
print(len(result))
# 2

# 过滤成功的结果
print(result)
# [<models.UserInfo object at 0x7fdeeaa774f0>, <models.UserInfo object at 0x7fdeeaa77490>]

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```

NOT IN，只需要加上~即可：

```
# 获取链接池、ORM表对象
import models

result = models.session.query(
    models.UserInfo,
).filter(
    ~models.UserInfo.age.in_((18, 19, 20))
).all()

# 过滤成功的结果数量
print(len(result))
# 0

# 过滤成功的结果
print(result)
# []

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```





## 模糊匹配

LIKE查询：

```
# 获取链接池、ORM表对象
import models

result = models.session.query(
    models.UserInfo,
).filter(
    models.UserInfo.name.like("Jack%")
).all()

# 过滤成功的结果数量
print(len(result))
# 1

# 过滤成功的结果
print(result)
# [<models.UserInfo object at 0x7fee1614f4f0>]

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```



## 分页查询

对结果all()返回的列表进行一次切片即可：

```
# 获取链接池、ORM表对象
import models

result = models.session.query(
    models.UserInfo,
).all()[0:1]

# 过滤成功的结果数量
print(len(result))
# 1

# 过滤成功的结果
print(result)
# [<models.UserInfo object at 0x7fee1614f4f0>]

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```





## 排序查询

ASC升序、DESC降序，需要指定排序规则：

```
# 获取链接池、ORM表对象
import models

result = models.session.query(
    models.UserInfo,
).filter(
    models.UserInfo.age > 12
).order_by(
    models.UserInfo.age.desc()
).all()

# 过滤成功的结果数量
print(len(result))
# 2

# 过滤成功的结果
print(result)
# [<models.UserInfo object at 0x7f90eccd26d0>, <models.UserInfo object at 0x7f90eccd2670>]

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```



## 聚合分组

聚合分组与having过滤：

```
# 获取链接池、ORM表对象
import models
# 导入聚合函数
from sqlalchemy import func

result = models.session.query(
    func.sum(models.UserInfo.age)
).group_by(
    models.UserInfo.gender
).having(
    func.sum(models.UserInfo.id > 1)
).all()

# 过滤成功的结果数量
print(len(result))
# 1

# 过滤成功的结果
print(result)
# [(Decimal('38'),)]

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```





# 多表查询

## 多表创建

五表关系：

![image-20211024165837689](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20211024165837689.png)

建表语句：

```
# models.py
from sqlalchemy.orm import sessionmaker
from sqlalchemy.orm import scoped_session
from sqlalchemy.orm import relationship

from sqlalchemy import (
    create_engine,
    Column,
    Integer,
    Date,
    String,
    Enum,
    ForeignKey,
    UniqueConstraint,
)
from sqlalchemy.ext.declarative import declarative_base

# 基础类
Base = declarative_base()

# 创建引擎
engine = create_engine(
    "mysql+pymysql://tom:123@192.168.0.120:3306/db1?charset=utf8mb4",
    # "mysql+pymysql://tom@127.0.0.1:3306/db1?charset=utf8mb4", # 无密码时
    # 超过链接池大小外最多创建的链接
    max_overflow=0,
    # 链接池大小
    pool_size=5,
    # 链接池中没有可用链接则最多等待的秒数，超过该秒数后报错
    pool_timeout=10,
    # 多久之后对链接池中的链接进行一次回收
    pool_recycle=1,
    # 查看原生语句
    # echo=True
)

# 绑定引擎
Session = sessionmaker(bind=engine)
# 创建数据库链接池，直接使用session即可为当前线程拿出一个链接对象
# 内部会采用threading.local进行隔离
session = scoped_session(Session)


class StudentsNumberInfo(Base):
    """学号表"""
    __tablename__ = "studentsNumberInfo"
    id = Column(Integer, primary_key=True, autoincrement=True, comment="主键")
    number = Column(Integer, nullable=False, unique=True, comment="学生编号")
    admission = Column(Date, nullable=False, comment="入学时间")
    graduation = Column(Date, nullable=False, comment="毕业时间")


class TeachersInfo(Base):
    """教师表"""
    __tablename__ = "teachersInfo"
    id = Column(Integer, primary_key=True, autoincrement=True, comment="主键")
    number = Column(Integer, nullable=False, unique=True, comment="教师编号")
    name = Column(String(64), nullable=False, comment="教师姓名")
    gender = Column(Enum("male", "female"), nullable=False, comment="教师性别")
    age = Column(Integer, nullable=False, comment="教师年龄")


class ClassesInfo(Base):
    """班级表"""
    __tablename__ = "classesInfo"
    id = Column(Integer, primary_key=True, autoincrement=True, comment="主键")
    number = Column(Integer, nullable=False, unique=True, comment="班级编号")
    name = Column(String(64), nullable=False, unique=True, comment="班级名称")
    # 一对一关系必须为连接表的连接字段创建UNIQUE的约束，这样才能是一对一，否则是一对多
    fk_teacher_id = Column(
        Integer,
        ForeignKey(
            "teachersInfo.id",
            ondelete="CASCADE",
            onupdate="CASCADE",
        ),
        nullable=False,
        unique=True,
        comment="班级负责人"
    )
    # 下面这2个均属于逻辑字段，适用于正反向查询。在使用ORM的时候，我们不必每次都进行JOIN查询，而恰好正反向的查询使用频率会更高
    # 这种逻辑字段不会在物理层面上创建，它只适用于查询，本身不占据任何数据库的空间
    # sqlalchemy的正反向概念与Django有所不同，Django是外键字段在那边，那边就作为正
    # 而sqlalchemy是relationship字段在那边，那边就作为正
    # 比如班级表拥有 relationship 字段，而老师表不曾拥有
    # 那么用班级表的这个relationship字段查老师时，就称为正向查询
    # 反之，如果用老师来查班级，就称为反向查询
    # 另外对于这个逻辑字段而言，根据不同的表关系，创建的位置也不一样：
    #  - 1 TO 1：建立在任意一方均可，查询频率高的一方最好
    #  - 1 TO M：建立在M的一方
    #  - M TO M：中间表中建立2个逻辑字段，这样任意一方都可以先反向，再正向拿到另一方
    #  - 遵循一个原则，ForeignKey建立在那个表上，那个表上就建立relationship
    #  - 有几个ForeignKey，就建立几个relationship
    # 总而言之，使用ORM与原生SQL最直观的区别就是正反向查询能带来更高的代码编写效率，也更加简单
    # 甚至我们可以不用外键约束，只创建这种逻辑字段，让表与表之间的耦合度更低，但是这样要避免脏数据的产生

    # 班级负责人，这里是一对一关系，一个班级只有一个负责人
    leader_teacher = relationship(
        # 正向查询时所链接的表，当使用 classesInfo.leader_teacher 时，它将自动指向fk的那一条记录
        "TeachersInfo",
        # 反向查询时所链接的表，当使用 teachersInfo.leader_class 时，它将自动指向该老师所管理的班级
        backref="leader_class",
    )


class ClassesAndTeachersRelationship(Base):
    """任教老师与班级的关系表"""
    __tablename__ = "classesAndTeachersRelationship"
    id = Column(Integer, primary_key=True, autoincrement=True, comment="主键")
    # 中间表中注意不要设置单列的UNIQUE约束，否则就会变为一对一
    fk_teacher_id = Column(
        Integer,
        ForeignKey(
            "teachersInfo.id",
            ondelete="CASCADE",
            onupdate="CASCADE",
        ),
        nullable=False,
        comment="教师记录"
    )

    fk_class_id = Column(
        Integer,
        ForeignKey(
            "classesInfo.id",
            ondelete="CASCADE",
            onupdate="CASCADE",
        ),
        nullable=False,
        comment="班级记录"
    )
    # 多对多关系的中间表必须使用联合唯一约束，防止出现重复数据
    __table_args__ = (
        UniqueConstraint("fk_teacher_id", "fk_class_id"),
    )

    # 逻辑字段
    # 给班级用的，查看所有任教老师
    mid_to_teacher = relationship(
        "TeachersInfo",
        backref="mid",
    )

    # 给老师用的，查看所有任教班级
    mid_to_class = relationship(
        "ClassesInfo",
        backref="mid"
    )


class StudentsInfo(Base):
    """学生信息表"""
    __tablename__ = "studentsInfo"
    id = Column(Integer, primary_key=True, autoincrement=True, comment="主键")
    name = Column(String(64), nullable=False, comment="学生姓名")
    gender = Column(Enum("male", "female"), nullable=False, comment="学生性别")
    age = Column(Integer, nullable=False, comment="学生年龄")
    # 外键约束
    # 一对一关系必须为连接表的连接字段创建UNIQUE的约束，这样才能是一对一，否则是一对多
    fk_student_id = Column(
        Integer,
        ForeignKey(
            "studentsNumberInfo.id",
            ondelete="CASCADE",
            onupdate="CASCADE"
        ),
        nullable=False,
        comment="学生编号"
    )
    # 相比于一对一，连接表的连接字段不用UNIQUE约束即为多对一关系
    fk_class_id = Column(
        Integer,
        ForeignKey(
            "classesInfo.id",
            ondelete="CASCADE",
            onupdate="CASCADE"
        ),
        comment="班级编号"
    )
    # 逻辑字段
    # 所在班级, 这里是一对多关系，一个班级中可以有多名学生
    from_class = relationship(
        "ClassesInfo",
        backref="have_student",
    )
    # 学生学号，这里是一对一关系，一个学生只能拥有一个学号
    number_info = relationship(
        "StudentsNumberInfo",
        backref="student_info",
    )


if __name__ == "__main__":
    # 删除表
    Base.metadata.drop_all(engine)
    # 创建表
    Base.metadata.create_all(engine)
```



插入数据：

```
# 获取链接池、ORM表对象
import models
import datetime


models.session.add_all(
    (
        # 插入学号表数据
        models.StudentsNumberInfo(
            number=160201,
            admission=datetime.datetime(2016, 9, 1).date(),
            graduation=datetime.datetime(2021, 6, 15).date()
        ),
        models.StudentsNumberInfo(
            number=160101,
            admission=datetime.datetime(2016, 9, 1).date(),
            graduation=datetime.datetime(2021, 6, 15).date()
        ),
        models.StudentsNumberInfo(
            number=160301,
            admission=datetime.datetime(2016, 9, 1).date(),
            graduation=datetime.datetime(2021, 6, 15).date()
        ),
        models.StudentsNumberInfo(
            number=160102,
            admission=datetime.datetime(2016, 9, 1).date(),
            graduation=datetime.datetime(2021, 6, 15).date()
        ),
        models.StudentsNumberInfo(
            number=160302,
            admission=datetime.datetime(2016, 9, 1).date(),
            graduation=datetime.datetime(2021, 6, 15).date()
        ),
        models.StudentsNumberInfo(
            number=160202,
            admission=datetime.datetime(2016, 9, 1).date(),
            graduation=datetime.datetime(2021, 6, 15).date()
        ),
        # 插入教师表数据
        models.TeachersInfo(
            number=3341, name="David", gender="male", age=32,
        ),
        models.TeachersInfo(
            number=3342, name="Jason", gender="male", age=30,
        ),
        models.TeachersInfo(
            number=3343, name="Lisa", gender="female", age=28,
        ),
        # 插入班级表数据
        models.ClassesInfo(
            number=1601, name="one year one class", fk_teacher_id=1
        ),
        models.ClassesInfo(
            number=1602, name="one year two class", fk_teacher_id=2
        ),
        models.ClassesInfo(
            number=1603, name="one year three class", fk_teacher_id=3
        ),
        # 插入中间表数据
        models.ClassesAndTeachersRelationship(
            fk_class_id=1, fk_teacher_id=1
        ),
        models.ClassesAndTeachersRelationship(
            fk_class_id=2, fk_teacher_id=1
        ),
        models.ClassesAndTeachersRelationship(
            fk_class_id=3, fk_teacher_id=1
        ),
        models.ClassesAndTeachersRelationship(
            fk_class_id=1, fk_teacher_id=2
        ),
        models.ClassesAndTeachersRelationship(
            fk_class_id=3, fk_teacher_id=3
        ),
        # 插入学生表数据
        models.StudentsInfo(
            name="Jack", gender="male", age=17, fk_student_id=1, fk_class_id=2
        ),
        models.StudentsInfo(
            name="Tom", gender="male", age=18, fk_student_id=2, fk_class_id=1
        ),
        models.StudentsInfo(
            name="Mary", gender="female", age=16, fk_student_id=3,
            fk_class_id=3
        ),
        models.StudentsInfo(
            name="Anna", gender="female", age=17, fk_student_id=4,
            fk_class_id=1
        ),
        models.StudentsInfo(
            name="Bobby", gender="male", age=18, fk_student_id=6, fk_class_id=2
        ),
    )
)

models.session.commit()

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```









## JOIN查询

INNER JOIN：

```
# 获取链接池、ORM表对象
import models

result = models.session.query(
    models.StudentsInfo.name,
    models.StudentsNumberInfo.number,
    models.ClassesInfo.number
).join(
    models.StudentsNumberInfo,
    models.StudentsInfo.fk_student_id == models.StudentsNumberInfo.id
).join(
    models.ClassesInfo,
    models.StudentsInfo.fk_class_id == models.ClassesInfo.id
).all()

print(result)
# [('Jack', 160201, 1602), ('Tom', 160101, 1601), ('Mary', 160301, 1603), ('Anna', 160102, 1601), ('Bobby', 160202, 1602)]

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```

LEFT JOIN只需要在每个JOIN中指定isouter关键字参数为True即可：

```
session.query(
	左表.字段,
	右表.字段
)
.join(
	右表,
	链接条件,
	isouter=True
).all()
```

RIGHT JOIN需要换表的位置，SQLALchemy本身并未提供RIGHT JOIN，所以使用时一定要注意驱动顺序，小表驱动大表（如果不注意顺序，MySQL优化器内部也会优化）：

```
session.query(
	左表.字段,
	右表.字段
)
.join(
	左表,
	链接条件,
	isouter=True
).all()
```





## UNION&UNION ALL

将多个查询结果联合起来，必须使用filter()，后面不加all()方法。

因为all()会返回一个列表，而filter()返回的是一个<class 'sqlalchemy.orm.query.Query'>查询对象，此外，必须单拿某一个字段，不能不指定字段直接query()：

```
# 获取链接池、ORM表对象
import models

students_name = models.session.query(models.StudentsInfo.name).filter()
students_number = models.session.query(models.StudentsNumberInfo.number)\
    .filter()
class_name = models.session.query(models.ClassesInfo.name).filter()

result = students_name.union_all(students_number).union_all(class_name)

print(result.all())
# [
#      ('Jack',), ('Tom',), ('Mary',), ('Anna',), ('Bobby',),
#      ('160101',), ('160102',), ('160201',), ('160202',), ('160301',), ('160302',),
#      ('one year one class',), ('one year three class',), ('one year two class',)
# ]

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```







## 子查询

子查询使用subquery()实现，如下所示，查询每个班级中年龄最小的人：

```
# 获取链接池、ORM表对象
import models
from sqlalchemy import func

# 子查询中所有字段的访问都需要加上c的前缀
# 如 sub_query.c.id、 sub_query.c.name等
sub_query = models.session.query(
    # 使用label()来为字段AS一个别名
    # 后续访问需要通过sub_query.c.alias进行访问
    func.min(models.StudentsInfo.age).label("min_age"),
    models.ClassesInfo.id,
    models.ClassesInfo.name
).join(
    models.ClassesInfo,
    models.StudentsInfo.fk_class_id == models.ClassesInfo.id
).group_by(
    models.ClassesInfo.id
).subquery()


result = models.session.query(
    models.StudentsInfo.name,
    sub_query.c.min_age,
    sub_query.c.name
).join(
    sub_query,
    sub_query.c.id == models.StudentsInfo.fk_class_id
).filter(
   sub_query.c.min_age == models.StudentsInfo.age
)

print(result.all())
# [('Jack', 17, 'one year two class'), ('Mary', 16, 'one year three class'), ('Anna', 17, 'one year one class')]

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```





## 正反查询

上面我们都是通过JOIN进行查询的，实际上我们也可以通过逻辑字段relationship进行查询。

下面是正向查询的示例，正向查询是指从有relationship逻辑字段的表开始查询：

```
# 查询所有学生的所在班级，我们可以通过学生的from_class字段拿到其所在班级
# 另外，对于学生来说，班级只能有一个，所以have_student应当是一个对象

# 获取链接池、ORM表对象
import models

students_lst = models.session.query(
    models.StudentsInfo
).all()

for row in students_lst:
    print(f"""
            student name : {row.name}
            from : {row.from_class.name}
          """)

# student name : Mary
# from : one year three class

# student name : Anna
# from : one year one class

# student name : Bobby
# from : one year two class


# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```

下面是反向查询的示例，反向查询是指从没有relationship逻辑字段的表开始查询：

```
# 查询所有班级中的所有学生，学生表中有relationship，并且它的backref为have_student，所以我们可以通过班级.have_student来获取所有学生记录

# 另外，对于班级来说，学生可以有多个，所以have_student应当是一个序列

# 获取链接池、ORM表对象
import models

classes_lst = models.session.query(
    models.ClassesInfo
).all()

for row in classes_lst:
    print("class name :", row.name)
    for student in row.have_student:
        print("student name :", student.name)

# class name : one year one class
#      student name : Jack
#      student name : Anna
# class name : one year two class
#      student name : Tom
# class name : one year three class
#      student name : Mary
#      student name : Bobby

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```

总结，正向查询的逻辑字段总是得到一个对象，反向查询的逻辑字段总是得到一个列表。



## 反向方法

使用逻辑字段relationship可以直接对一些跨表记录进行增删改查。

由于逻辑字段是一个类似于列表的存在（仅限于反向查询，正向查询总是得到一个对象），所以列表的绝大多数方法都能用。

```
<class 'sqlalchemy.orm.collections.InstrumentedList'>
    - append()
    - clear()
    - copy()
    - count()
    - extend()
    - index()
    - insert()
    - pop()
    - remove()
    - reverse()
    - sort()
```

下面不再进行实机演示，因为我们上面的几张表中做了很多约束。

```
# 比如
# 给老师增加班级
result = session.query(Teachers).first()
# extend方法：
result.re_class.extend([
    Classes(name="三年级一班",),
    Classes(name="三年级二班",),
])

# 比如
# 减少老师所在的班级
result = session.query(Teachers).first()

# 待删除的班级对象,集合查找比较快
delete_class_set = {
    session.query(Classes).filter_by(id=7).first(),
    session.query(Classes).filter_by(id=8).first(),
}

# 循换老师所在的班级
# remove方法：
for class_obj in result.re_class:
    if class_obj in delete_class_set:
        result.re_class.remove(class_obj)

# 比如
# 清空老师所任教的所有班级
# 拿出一个老师
result = session.query(Teachers).first()
result.re_class.clear()
```



## 查询案例

1）查看每个班级共有多少学生：

JOIN查询：

```
# 获取链接池、ORM表对象
import models

from sqlalchemy import func

result = models.session.query(
    models.ClassesInfo.name,
    func.count(models.StudentsInfo.id)
).join(
    models.StudentsInfo,
    models.ClassesInfo.id == models.StudentsInfo.fk_class_id
).group_by(
    models.ClassesInfo.id
).all()

print(result)
# [('one year one class', 2), ('one year two class', 2), ('one year three class', 1)]


# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```

正反查询：

```
# 获取链接池、ORM表对象
import models

result = {}
class_lst = models.session.query(
    models.ClassesInfo
).all()

for row in class_lst:
    for student in row.have_student:
        count = result.setdefault(row.name, 0)
        result[row.name] = count + 1

print(result.items())
# dict_items([('one year one class', 2), ('one year two class', 2), ('one year three class', 1)])

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```

2）查看每个学生的入学、毕业年份以及所在的班级名称：

JOIN查询：

```
# 获取链接池、ORM表对象
import models

result = models.session.query(
    models.StudentsNumberInfo.number,
    models.StudentsInfo.name,
    models.ClassesInfo.name,
    models.StudentsNumberInfo.admission,
    models.StudentsNumberInfo.graduation
).join(
    models.StudentsInfo,
    models.StudentsInfo.fk_class_id == models.ClassesInfo.id
).join(
    models.StudentsNumberInfo,
    models.StudentsNumberInfo.id == models.StudentsInfo.fk_student_id
).order_by(
    models.StudentsNumberInfo.number.asc()
).all()

print(result)
# [
#     (160101, 'Tom', 'one year one class', datetime.date(2016, 9, 1), datetime.date(2021, 6, 15)),
#     (160102, 'Anna', 'one year one class', datetime.date(2016, 9, 1), datetime.date(2021, 6, 15)),
#     (160201, 'Jack', 'one year two class', datetime.date(2016, 9, 1), datetime.date(2021, 6, 15)),
#     (160202, 'Bobby', 'one year two class', datetime.date(2016, 9, 1), datetime.date(2021, 6, 15)),
#     (160301, 'Mary', 'one year three class', datetime.date(2016, 9, 1), datetime.date(2021, 6, 15))
# ]

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```

正反查询：

```
# 获取链接池、ORM表对象
import models

result = []

student_lst = models.session.query(
    models.StudentsInfo
).all()

for row in student_lst:
    result.append((
        row.number_info.number,
        row.name,
        row.from_class.name,
        row.number_info.admission,
        row.number_info.graduation
    ))

print(result)
# [
#     (160101, 'Tom', 'one year one class', datetime.date(2016, 9, 1), datetime.date(2021, 6, 15)),
#     (160102, 'Anna', 'one year one class', datetime.date(2016, 9, 1), datetime.date(2021, 6, 15)),
#     (160201, 'Jack', 'one year two class', datetime.date(2016, 9, 1), datetime.date(2021, 6, 15)),
#     (160202, 'Bobby', 'one year two class', datetime.date(2016, 9, 1), datetime.date(2021, 6, 15)),
#     (160301, 'Mary', 'one year three class', datetime.date(2016, 9, 1), datetime.date(2021, 6, 15))
# ]

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```

3）查看David所教授的学生中年龄最小的学生：

JOIN查询：

```
# 获取链接池、ORM表对象
import models

result = models.session.query(
    models.TeachersInfo.name,
    models.StudentsInfo.name,
    models.StudentsInfo.age,
    models.ClassesInfo.name
).join(
    models.ClassesAndTeachersRelationship,
    models.ClassesAndTeachersRelationship.fk_class_id == models.ClassesInfo.id
).join(
    models.TeachersInfo,
    models.ClassesAndTeachersRelationship.fk_teacher_id == models.TeachersInfo.id
).join(
    models.StudentsInfo,
    models.StudentsInfo.fk_class_id == models.ClassesInfo.id
).filter(
    models.TeachersInfo.name == "David"
).order_by(
    models.StudentsInfo.age.asc(),
    models.StudentsInfo.id.asc()
).limit(1).all()

print(result)
# [('David', 'Mary', 16, 'one year three class')]

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```

正反查询：

```
# 获取链接池、ORM表对象
import models

david = models.session.query(
    models.TeachersInfo
).filter(
    models.TeachersInfo.name == "David"
).first()

student_lst = []

# 反向查询拿到任教班级，反向是一个列表，所以直接for
for row in david.mid:
    cls = row.mid_to_class
    # 通过任教班级，反向拿到其下的所有学生
    cls_students = cls.have_student
    # 遍历学生
    for student in cls_students:
        student_lst.append(
            (
                david.name,
                student.name,
                student.age,
                cls.name
            )
        )

# 筛选出年龄最小的
min_age_student_lst = sorted(
    student_lst, key=lambda tpl: tpl[2])[0]

print(min_age_student_lst)
# ('David', 'Mary', 16, 'one year three class')

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```

4）查看每个班级的负责人是谁，以及任课老师都有谁：

JOIN查询：

```
# 获取链接池、ORM表对象
import models

from sqlalchemy import func

# 先查任课老师
sub_query = models.session.query(
    models.ClassesAndTeachersRelationship.fk_class_id.label("class_id"),
    func.group_concat(models.TeachersInfo.name).label("have_teachers")
).join(
    models.ClassesInfo,
    models.ClassesAndTeachersRelationship.fk_class_id == models.ClassesInfo.id
).join(
    models.TeachersInfo,
    models.ClassesAndTeachersRelationship.fk_teacher_id == models.TeachersInfo.id
).group_by(
    models.ClassesAndTeachersRelationship.fk_class_id
).subquery()

result = models.session.query(
    models.ClassesInfo.name.label("class_name"),
    models.TeachersInfo.name.label("leader_teacher"),
    sub_query.c.have_teachers.label("have_teachers")
).join(
    models.TeachersInfo,
    models.ClassesInfo.fk_teacher_id == models.TeachersInfo.id
).join(
    sub_query,
    sub_query.c.class_id == models.ClassesInfo.id
).all()

print(result)
# [('one year one class', 'David', 'Jason,David'), ('one year two class', 'Jason', 'David'), ('one year three class', 'Lisa', 'David,Lisa')]

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```

正反查询：

```
# 获取链接池、ORM表对象
import models

result = []

# 获取所有班级
classes_lst = models.session.query(
    models.ClassesInfo
).all()

for cls in classes_lst:
    cls_message = [
        cls.name,
        cls.leader_teacher.name,
        [],
    ]
    for row in cls.mid:
        cls_message[-1].append(row.mid_to_teacher.name)
    result.append(cls_message)

print(result)
# [['one year one class', 'David', ['David', 'Jason']], ['one year two class', 'Jason', ['David']], ['one year three class', 'Lisa', ['David', 'Lisa']]]

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```





# 原生SQL

## 查看执行命令

如果一条查询语句是filter()结尾，则该对象的\_\_str\_\_方法会返回格式化后的查询语句：

```
print(
    models.session.query(models.StudentsInfo).filter()
)

SELECT `studentsInfo`.id AS `studentsInfo_id`, `studentsInfo`.name AS `studentsInfo_name`, `studentsInfo`.gender AS `studentsInfo_gender`, `studentsInfo`.age AS `studentsInfo_age`, `studentsInfo`.fk_student_id AS `studentsInfo_fk_student_id`, `studentsInfo`.fk_class_id AS `studentsInfo_fk_class_id`
FROM `studentsInfo`
```



## 执行原生命令

执行原生命令可使用session.execute()方法执行，它将返回一个cursor游标对象，如下所示：

```
# 获取链接池、ORM表对象
import models

cursor = models.session.execute(
    "SELECT * FROM studentsInfo WHERE id = (:uid)", params={'uid': 1})

print(cursor.fetchall())

# 关闭链接，亦可使用session.remove()，它将回收该链接
models.session.close()
```
