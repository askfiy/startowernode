# 序列化

序列化是指将在内存中的数据变成为可存储或者可传输的数据。

Python中称序列化为pickling，而其他编程语言中则称之为serialization、marshalling、flattening等等，都是一个意思。

序列化最重要的目的是数据持久化保存，以及数据跨平台传输：

- 持久化保存：数据无法在内存中长期驻留，因此可以将其转变为某种格式并写入到磁盘之中
- 跨平台传输：不同的编程语言中对于数据的表示都是不同的，如Python中的set在很多语言中就不具备，故可以将数据进行序列化，变为一种大家都认识的格式



# json

JSON格式最早来源于JavaScript语言，现在已经成为跨平台语言传输的通用格式。

它的操纵及其简单，以下是JSON与Python中数据类型的对应关系：

| Python数据类型 | JSON格式表示  |
| -------------- | ------------- |
| dict           | {}            |
| list           | []            |
| str            | string        |
| int or float   | int or float  |
| True or False  | true or false |
| None           | Null          |

JSON优点是操纵简单、跨语言传输十分方便，因为它采用字符串进行存储。

JSON缺点是仅支持Python基本数据类型，像函数、类这种都不被支持。

Python中进行JSON格式化，可以选择内置的json模块：

[官方文档](https://docs.python.org/zh-cn/3.6/library/json.html)

常用方法一览：

| 方法         | 描述                                             |
| ------------ | ------------------------------------------------ |
| json.dumps() | 将Python中的基本数据类型序列化为JSON格式的字符串 |
| json.loads() | 将JSON格式字符串反序列化为Python中的基本数据类型 |
| json.dump()  | 同json.dumps()，不过写入文件更方便               |
| json.load()  | 同json.loads()，不过读取文件更方便               |

## 序列化

使用json.dumps()可以将Python中的基本数据类型序列化为JSON格式的字符串：

```
>>> import json
>>> userMessage = {"name" : "yunya", "age" : "18", "gender" : True, "hobby" : ["read", "playGame"]}
>>> json.dumps(userMessage)
'{"name": "yunya", "age": "18", "gender": true, "hobby": ["read", "playGame"]}'
```

如果想将序列化的结果进行持久化保存，可以使用json.dump()方法，它可以指定输出对象为文件句柄，如下示例：

```
>>> import json
>>> userMessage = {"name" : "yunya", "age" : "18", "gender" : True, "hobby" : ["read", "playGame"]}
>>> with open(file="test.json", mode="wt", encoding="utf8") as f:
	    json.dump(userMessage,fp=f)
```



## 反序列化

使用json.loads()可以将JSON格式字符串反序列化为Python中的基本数据类型：

```
>>> userJsonStr = '{"name": "yunya", "age": "18", "gender": true, "hobby": ["read", "playGame"]}'
>>> json.loads(userJsonStr)
{'name': 'yunya', 'age': '18', 'gender': True, 'hobby': ['read', 'playGame']}
```

如果想从文件中读取JSON字符串并进行反序列化，可以使用json.load()方法，它可以指定读取对象为文件句柄，如下所示：

```
>>> import json
>>> with open(file="test.json", mode="rt", encoding="utf8") as f:
        userMessage = json.load(fp=f)
>>> userMessage
{'name': 'yunya', 'age': '18', 'gender': True, 'hobby': ['read', 'playGame']}
```



## 序列化的过程

Python的基本数据类型转换为JSON字符串时，经历了什么？

总计可分为2步：

- 修改str的单引号为双引号
- 根据JSON与Python中数据类型的对应关系，将数据进行包装转换为JSON表现形式

示例、修改str的单引号为双引号：

```
>>> pyStr = 'string'
>>> jsonStr = json.dumps(pyStr)
>>> jsonStr
'"string"
```

示例、根据JSON与Python中数据类型的对应关系，将数据进行包装转换为JSON表现形式：

```
>>> pyType = None
>>> jsonType = json.dumps(pyType)
>>> jsonType
'null'
```



## 中文显示

如果JSON序列化的字符串中带有中文，则将其转变为Unicode的16进制表现形式：

```
>>> pyStr = "云崖先生"
>>> json.dumps(pyStr)
'"\\u4e91\\u5d16\\u5148\\u751f"'
```

你可以指定序json.dumps()中的关键字参数ensure_ascii为False，此时不会对中文字符进行转换：

```
>>> json.dumps(pyStr, ensure_ascii=False)
'"云崖先生"'
```



## 猴子补丁介绍

Monkey Patch是指用一个补丁偷偷的将一个原本的功能进行替换，使用者并不知道目前使用的功能是已经替换后的功能。

第三方模块ujson相比于json来说性能更加的高效，你可以将它作为猴子补丁偷偷的替换掉json，只需要在项目运行时的入口加入并运行一个替换函数：

```
import json
import ujson

def monkeyPatchJson():
    json.__name__ = "ujson"
    json.dumps = ujson.dumps
    json.loads = ujson.loads
    
monkeyPatchJson()
```

修改完毕后，重启项目，后面的代码甚至不需要任何修改，就能使用性能更高的ujson了。

采用猴子补丁之后，如果发现ujson不符合预期，那也可以快速撤掉补丁，删除掉函数的执行语句即可。



## 序列化非基本数据类型

datetime类型并非是Python基本的数据类型，所以JSON不支持对它的序列化。

```
import datetime
import json

now = datetime.datetime.now()
strNow = json.dumps(now, ensure_ascii=False)
print(strNow)

# TypeError: Object of type 'datetime' is not JSON serializable
```

此时我们可以手动的扩展json.dumps()的功能，让其支持datetime的序列化。

具体思路是将非Python基本数据类型转换为基本数据类型后再使用json.dumps()对其进行序列化，实现步骤如下所示：

```
import datetime
import json
from json.encoder import JSONEncoder


class JsonRealize(JSONEncoder):
    """
    该类是自定义序列化非Python基本数据类型的逻辑实现类
    主要作用是继承并覆写父类JSONEncoder的default()
    """

    def default(self, serializeObject):
        # 发现序列化对象是datetime类型的话，就将其转换为str类型
        if isinstance(serializeObject, datetime.datetime):
            return str(serializeObject)

        # 如果是其他类型，则交由JSONEncoder的default()进行处理
        return JSONEncoder.default(self, serializeObject)


if __name__ == '__main__':
    now = datetime.datetime.now()
    strNow = json.dumps(now, cls=JsonRealize, ensure_ascii=False)
    print(strNow)

# "2021-05-23 20:40:49.446823"
```



## json模块使用注意事项



Python3.6以及Python2.7之前均不支持反序列化bytes类型。

也就是说json.loads()一个bytes类型会抛出异常。



# pickle

　pickle模块是Python自带的模块，它和json模块的方法全部一致，区别在于pickle序列化后的类型是bytes类型，而json序列化后的类型是字符串类型。

由于要考虑到多语言的兼容性问题，json模块并不支持Python除基本数据类型之外的类型。如：函数类型，类等等。

但是pickle由于只支持Python使用，所以有了更强的对Python序列化对象的支持。

pickle可以序列化函数，类等等，但是并不推荐这么做，因为保存的只有一个内存地址。

另外，由于pickle的局限性太强所以更推荐使用JSON进行序列化操作。

由于很少使用，以及与json模块的方法一致，这里不再进行演示了。

[官方文档](https://docs.python.org/zh-cn/3.6/library/pickle.html)



# shelve

shelves是Python自带的模块，它能够更加方便的将Python数据进行持久化保存。

[官方文档](https://docs.python.org/zh-cn/3.6/library/shelve.html)

它将整个文件看做一个大的字典，将字典中的key看做Python中的标识符，将value看做存储的对象，因此操作起来十分方便。

只需要记住2个方法即可：

- shelve.open()：打开一个文件，这个文件是可读可写的
- shelve.close()：关闭文件

示例演示：

```
>>> import shelve
>>> with shelve.open("test.txt") as f:
	f["name"] = "Yunya"
	f["age"] = 18
	f["hobby"] = ["readBook", "playGame"]
	
>>> with shelve.open("test.txt") as f:
	name = f.get("name")
	
>>> name
'Yunya'
```



