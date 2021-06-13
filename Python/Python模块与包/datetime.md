# datetime简介

datetime模块是Python内置模块，相比于time模块能够更加方便的操纵时间。

[官方文档](https://docs.python.org/zh-cn/3.6/library/datetime.html)

以下举例部分常用方法：

| 方法                                 | 描述                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| datetime.datetime()                  | 实例化返回一个datetime的对象                                 |
| datetime.datetime.now()              | 获取本地时间，返回一个datetime的对象                         |
| datetime.datetime.utcnow()           | 获取世界时间，返回一个datetime的对象                         |
| datetime.datetime.fromtimestamp()    | 放入时间戳时间，直接转换为本地的datetime对象时间             |
| datetime.datetime.utcfromtimestamp() | 放入时间戳时间，直接转换为世界的的datetime对象时间           |
| datetime.timedelta()                 | 在一个datetime对象时间的基础上进行加减，返回一个新的datetime的对象 |
| datetime.datetime.weekday()          | 放入一个datetime对象，获取该对象是那一周的第几天，从0开始计算，一周就是0-6 |



# 对象获取

datetime.datetime.now()和datetime.datetime.utcnow()都可以获取一个表示当前时间的datetime对象。

```
>>> import datetime
>>> datetime.datetime.now()
datetime.datetime(2021, 5, 22, 23, 31, 43, 468077)
>>> datetime.datetime.utcnow()
datetime.datetime(2021, 5, 22, 15, 31, 52, 841214)
```

为datetime.datetime.fromtimestamp()放入一个时间戳可直接获取到表示本地时间的datetime的对象：

```
>>> datetime.datetime.fromtimestamp(11111)
datetime.date(1970, 1, 1)
>>> datetime.datetime.utcfromtimestamp(11111)
datetime.datetime(1970, 1, 1, 3, 5, 11)
```

datetime对象是str类型的更高一级封装，你可以将datetime对象转换为str类型：

```
>>> str(datetime.datetime.now())
'2021-05-23 00:06:03.271989'
```



# 对象属性

一个datetime对象拥有很多时间属性，如下表所示：

| 属性        | 描述          |
| ----------- | ------------- |
| year        | 年份（int）   |
| month       | 月份（int）   |
| day         | 天数（int）   |
| hour        | 时数（int）   |
| minute      | 分数（int）   |
| second      | 秒数（int）   |
| microsecond | 毫秒数（int） |

你可以快速的获取它们：

```
>>> currentTime = datetime.datetime.now()
>>> currentTime.year
2021
>>> currentTime.month
5
>>> currentTime.day
22
>>> currentTime.hour
23
>>> currentTime.minute
48
>>> currentTime.second
52
>>> currentTime.microsecond
527012
```



# 对象方法

一个datetime对象拥有很多方法，如下表所示：

| 方法                          | 描述                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| datetimeObject.timestamp()    | 返回一个时间戳，不同于time模块的时间戳，该方法返回的是一个float类型 |
| datetimeObject.timetuple()    | 返回与time.localtime()兼容的本地时间元组                     |
| datetimeObject.utctimetuple() | 返回与time.gmtime()兼容的UTC时间元组                         |
| datetimeObject.ctime()        | 返回ctime()样式字符串                                        |
| datetimeObject.isoformat()    | 根据ISO格式返回时间字符串                                    |
| datetimeObject.strptime()     | 类似于time.strptime()                                        |
| datetimeObject.tzname()       | 返回时区名字                                                 |
| datetimeObject.replace()      | 替换一个字符串格式的时间中某部分，返回一个新的datetime的对象 |

由于这些方法使用的时候并不多，所以只举例一个时间替换。

将当前时间的年份替换为1998年：

```
>>> currentTime = datetime.datetime.now()
>>> currentTime.replace(year=1998)
datetime.datetime(1998, 5, 23, 0, 22, 24, 698922)
```



# 时间加减

datetime对象允许通过和datetime.timedelta()进行加减，以便进行时间的计算。

- 时间加减中不支持年份的计算，可以用365天代替

当前时间加3天：

```
>>> sum3dayTime = datetime.timedelta(+3) + currentTime
>>> sum3dayTime
datetime.datetime(2021, 5, 26, 0, 15, 15, 405378)
```

当前时间-3天：

```
>>> sub3dayTime = datetime.timedelta(-3) + currentTime
>>> sub3dayTime
datetime.datetime(2021, 5, 20, 0, 15, 15, 405378)
```

当前时间加3小时：

```
>>> sum3hours = datetime.timedelta(hours=+3) + currentTime
>>> sum3hours
datetime.datetime(2021, 5, 23, 3, 15, 15, 405378)
```

当前时间减3小时：

```
>>> sub3hours = datetime.timedelta(hours=-3) + currentTime
>>> sub3hours
datetime.datetime(2021, 5, 22, 21, 15, 15, 405378)
```



