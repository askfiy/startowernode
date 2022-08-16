

# time简介

time模块是Python自带的模块，提供了时间的访问和转换功能。

[官方文档](https://docs.python.org/zh-cn/3.6/library/time.html)

time模块中，对时间的表示包含3个概念：

- 时间戳形式：从1970.1.1 08:00:00（Unix纪元）开始到现在所经历的毫秒数，它int类型
- 结构化形式：以元组包裹的形式进行时间的展示，它tuple类型
- 字符串形式：以字符串的形式进行时间的展示，它是str类型

3种表示时间的方式之间可以互相转换，如下图所示：

![image-20210610192050443](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210610192050443.png)

# 时间获取

获取时间的方法：

| 方法             | 描述                     | 表现形式 |
| ---------------- | ------------------------ | -------- |
| time.time()      | 获取时间戳形式的时间     | 时间戳   |
| time.localtime() | 获取结构化的本地时间     | 结构化   |
| time.gmtime()    | 获取结构化的世界时间     | 结构化   |
| time.asctime()   | 获取字符串形式的本地时间 | 字符串   |
| time.ctime()     | 获取字符串形式的世界时间 | 字符串   |
| time.strftime()  | 获取自定义格式的本地时间 | 字符串   |

Ps：本地时间在中国以东八区，上海时间时间为准，和世界时间（UTC）差了8小时



示例演示：

```
>>> import time
>>> time.time()
1621693706.0193129
>>> time.localtime()
time.struct_time(tm_year=2021, tm_mon=5, tm_mday=22, tm_hour=22, tm_min=28, tm_sec=43, tm_wday=5, tm_yday=142, tm_isdst=0)
>>> time.gmtime()
time.struct_time(tm_year=2021, tm_mon=5, tm_mday=22, tm_hour=14, tm_min=29, tm_sec=0, tm_wday=5, tm_yday=142, tm_isdst=0)
>>> time.asctime()
'Sat May 22 22:29:05 2021'
>>> time.ctime()
'Sat May 22 22:29:09 2021'
>>> time.strftime("%Y-%m-%d %H:%M:%S")
'2021-05-22 22:29:45'
```



## 结构化时间说明

在time.localtime()和time.gmtime()中，可以看到元组中有很多看不懂的数据项属性组成，它们的释义如下：

| 属性     | 描述                             |
| -------- | -------------------------------- |
| tm_year  | 年                               |
| tm_mon   | 月                               |
| tm_mday  | 日                               |
| tm_ hour | 时                               |
| tm_min   | 分                               |
| tm_sec   | 秒                               |
| tm_wday  | 星期几，从0开始计算，一周就是0-6 |
| tm_yday  | 该年份的第几天                   |
| tm_isdst | 夏令营时间                       |

这些属性都可以单独的提取出来，如获取这一年的年、月、日：

```
>>> time.localtime().tm_year
2021
>>> time.localtime().tm_mon
5
>>> time.localtime().tm_mday
22
```



## time.strftime()

放入一段字符串，将时间格式化出来，如下所示：

```
>>> time.strftime("%Y-%m-%d %H:%M:%S")
'2021-05-22 22:29:45'
```

%Y-%m%d这些都代表格式化时间的占位符，分别代表年月日等信息。

如下表所示：

| 符号 | 描述              |
| ---- | ----------------- |
| %Y   | 格式化年份        |
| %m   | 格式化月份        |
| %d   | 格式化天数        |
| %H   | 格式化小时        |
| %M   | 格式化分钟        |
| %S   | 格式化秒数        |
| %X   | 等同于 “%H:%M:%S” |

如想了解更多，参照官网示例截图：

%accordion%点我查看%accordion%

![image-20210522224109740](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210522224109740.png)

%/accordion%



# 时间转换

## 转换方法

以下是时间表现形式互相转换的方法：

| 方法             | 描述                                   |
| ---------------- | -------------------------------------- |
| time.mktime()    | 放入结构化时间，转换为时间戳时间       |
| time.strftime()  | 放入结构化时间，转换为字符串时间       |
| time.strptime()  | 放入字符串时间，转换为结构化时间       |
| time.localtime() | 放入时间戳时间，转换为结构化的本地时间 |
| time.gmtime()    | 放入时间戳时间，转换为结构化的世界时间 |

示例演示：

```
>>> time.mktime(time.localtime())
1621694964.0
>>> time.strftime("%Y-%m-%d %X", time.gmtime())
'2021-05-22 14:50:01'
>>> time.strptime(time.ctime())
time.struct_time(tm_year=2021, tm_mon=5, tm_mday=22, tm_hour=22, tm_min=50, tm_sec=15, tm_wday=5, tm_yday=142, tm_isdst=-1)
>>> time.localtime(time.time())
time.struct_time(tm_year=2021, tm_mon=5, tm_mday=22, tm_hour=22, tm_min=50, tm_sec=33, tm_wday=5, tm_yday=142, tm_isdst=0)
>>> time.gmtime(time.time())
time.struct_time(tm_year=2021, tm_mon=5, tm_mday=22, tm_hour=14, tm_min=51, tm_sec=4, tm_wday=5, tm_yday=142, tm_isdst=0)
```



## 常用操作

将时间戳转换为固定的UTC时间字符串格式：

```
>>> time.ctime(00)
'Thu Jan  1 08:00:00 1970'
```

将时间戳转换为本地时间的字符串表现形式：

```
>>> t = 1293495903                                   # 有一个时间戳
>>> stuct = time.localtime(t)                        # 先将其转为本地的结构化时间
>>> stringTime = time.strftime("%Y-%m-%d %X", stuct) # 再将其转换为字符串时间
>>> stringTime
'2010-12-28 08:25:03'
```

将时间戳转换为世界时间的字符串表现形式：

```
>>> t = 1293495903                                   # 有一个时间戳
>>> stuct = time.gmtime(t)                           # 先将其转为世界的结构化时间
>>> stringTime = time.strftime("%Y-%m-%d %X", stuct) # 再将其转换为字符串时间
>>> stringTime
'2010-12-28 00:25:03'
```

将一个字符串时间转换为时间戳：

```
>>> stringTime = "1998-01-26 00:00:10"               # 有一个字符串时间
>>> stuct = time.strptime(stringTime, "%Y-%m-%d %X") # 先将其转换为结构化时间
>>> stamp = time.mktime(stuct)                       # 再将其转换为时间戳
>>> stamp
885744010.0
```



# 线程睡眠

通过time.sleep()方法，可指定主线程睡眠多少秒，如下所示，第2个print()将在2秒后运行：

```
import time
print("start")
time.sleep(2)
print("end")
```



# 其他操作

## 日期判断

根据时间戳，获取7天后的时间：

```
>>> currentTime = time.time()
>>> sum7dayTime = currentTime + 7 * 86400
>>> time.strftime("%Y-%m-%d", time.localtime(sum7dayTime))
'2021-05-29'
```

根据时间戳，获取3天前的时间：

```
>>> currentTime = time.time()
>>> sub3dayTime = currentTime - 3 * 86400
>>> time.strftime("%Y-%m-%d", time.localtime(sub3dayTime))
'2021-05-19'
```

如果是时间戳操作，谨记1天是86400秒即可。



## 定时任务

脚本启动后，每隔一分钟，向屏幕打印一次hello world：

```
import time

currentSec = time.localtime().tm_sec
while 1:
    if time.localtime().tm_sec == currentSec:
        print("hello world")
        time.sleep(1)
```



## 延时任务

脚本启动后的一分钟时，打印一次hello world：

```
import time

currentTime = time.time()
runTime = currentTime + 60

while 1:
    if time.time() == runTime:
        print("hello world")
        break
print("任务执行完毕")
```

