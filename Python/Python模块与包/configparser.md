

# configparser简介

configparser模块是Python的内置模块，提供了配置文件创建、解析、修改等功能。

[官方文档](https://docs.python.org/zh-cn/3.6/library/configparser.html)

值得注意的是，在Python2中，它的命名是驼峰式的，为ConfigParser。



# 认识配置文件

配置文件常以.ini或者.cfg作为后缀，注释方式有2种。

\#注释与;注释，一个配置项是以键值对方式进行存储，通过:或者=分割。

```
[regulator]
user_name : Yunya
age = 21
sex = male
is_admin = true
salary = 20

[path]
RUN_LOG_FILE = log/run.log $true
ERROR_LOG_FILE = log/error.log $true
```

如果某一个配置项后面加上了$true，则$true在解析的时候将被替换为BASE_DIR，也就说该$true会被替换为完整的路径。

我个人喜欢将一个配置文件分成3部分，尽管下面的叫法并不是非常的正确：

- 配置项分类（classify）：以[]包裹的数据项，或者称为block
- 配置项的键（key）：:或者=左边的数据项
- 配置项的值（value）：:或者=右边的数据项



# 字典一样操作

configparser模块能够让配置文件像字典一样进行操作。

下面介绍3个该模块提供的基本方法：

| 方法                                          | 描述               |
| --------------------------------------------- | ------------------ |
| ConfigParser()                                | 创建文档对象       |
| documentObject.read(filenames, encoding=None) | 读取配置文件       |
| documentObject.write(fp)                      | 将文档对象写入磁盘 |

如下示例，对配置文件进行读取：

```
import configparser


# 创建文档对象，并且读取配置文件
documentObject = configparser.ConfigParser()
documentObject.read(filenames="./config.ini", encoding="u8")

# 获取所有classify
classifyAllTuple = tuple(documentObject.items())

# 获取所有的key和value
for classify in documentObject.values():
    print(dict(classify.items()))

# 获取指定classify下的指定key的value
# 需要自己做类型转换
userAge = documentObject["regulator"]["age"]
print(int(userAge))
```

新增一个配置文件，先创建一个空的文档对象，然后加入一些子字典，将他看做嵌套字典，最后进行写入磁盘即可：

```
import configparser

# 先创建一些classify以及kev-value配置项
defaultClassify = {
    "ip": "0.0.0.0",
    "port": 65535,
}

serverClassify = {
    "ip": "192.168.0.120",
    "port": 65536,
}

loginClassify = {
    "user": "root",
    "password": "123456",
    "db": 1,
    "verify": False,
}

# 创建一个空文档对象
noneDocumentObject = configparser.ConfigParser()

# 为这个空文档对象，添加classify，将它看成字典操作即可
noneDocumentObject["DEFAULT"] = defaultClassify
noneDocumentObject["SERVER"] = serverClassify
noneDocumentObject["LOGIN"] = loginClassify

# 将文档对象写入到磁盘
with open(file="./newConfig.ini", mode="wt", encoding="utf8") as f:
    noneDocumentObject.write(fp=f)
```

修改一个配置项，对字典中的value进行更新。最后将文档对象写入到磁盘：

```
import configparser

# 创建文档对象，并且读取配置文件
documentObject = configparser.ConfigParser()
documentObject.read(filenames="./config.ini", encoding="u8")

# 修改regulator下的age为30岁，注意这里必须为str类型
documentObject["regulator"]["age"] = "30"

# 写入磁盘
with open("./config.ini", mode="wt", encoding="utf8") as f:
    documentObject.write(fp=f)
```



# 读取配置文件

configparser模块此外也为文档对象提供了一些专用的方法，来操作配置文件。

如下所示：

| 方法                                              | 描述                                                         |
| ------------------------------------------------- | ------------------------------------------------------------ |
| sections(self)                                    | 拿到所有的classify                                           |
| options(self, section)                            | 指定一个classify，拿到其下所有的key                          |
| items(self, section=_UNSET, raw=False, vars=None) | 指定一个classify，拿到其下所有的key和value                   |
| get(self, section, option)                        | 指定classify和key，拿到具体的value，并将其转换为Python中的str类型 |
| getint(self, section, option)                     | 指定classify和key，拿到具体的value，并将其转换为Python中的int类型 |
| getfloat(self, section, option)                   | 指定classify和key，拿到具体的value，并将其转换为Python中的float类型，保留1位小数 |
| getboolean(self, section, option)                 | 指定classify和key，拿到具体的value，并将其转换为Python中的bool类型 |

示例演示：

```
import configparser

# 初始化文档对象，并且读取配置文件
documentObject = configparser.ConfigParser()
documentObject.read(filenames="./config.ini", encoding="utf8")

# 获取配置文件中所有的classify
allClassify = documentObject.sections()
print(allClassify)
# ['regulator', 'path']

# 获取classify（regulator）下所有的key
regulatorKeys = documentObject.options(section=allClassify[0])
print(regulatorKeys)
# ['user_name', 'age', 'sex', 'is_admin', 'salary']

# 获取classify(regulator)下所有的键值对
regulatorItems = documentObject.items(section=allClassify[0])
print(regulatorItems)
#　[('user_name', 'Yunya'), ('age', '21'), ('sex', 'male'), ('is_admin', 'true'),('salary', '20')]

# 获取classify(regulator)下的key(user_name)对应的value
# get()会自动将value转换为str类型
name = documentObject.get(section=allClassify[0], option="user_name")
print(name)
# Yunya

# 获取classify(regulator)下的key(age)对应的value
# getint()会自动将value转换为int类型
age = documentObject.getint(section=allClassify[0], option="age")
print(age)
# 21

# 获取classify(regulator)下的key(is_admin)对应的value
# getboolean()会自动将value转换为bool类型
isAdimin = documentObject.getboolean(section=allClassify[0], option="is_admin")
print(isAdimin)
# True

# 获取classify(regulator)下的key(salary)对应的value
# getfloat()会自动将value转换为float类型，并保留一位小数
salary = documentObject.getfloat(section=allClassify[0], option="salary")
print(salary)
# 20.0
```



# 修改配置文件

configparser模块也提供了一些修改配置文件的方法。

如下表所示，但我很少会使用到，所以不再进行案例书写了：

| 方法                                   | 描述                                 |
| -------------------------------------- | ------------------------------------ |
| add_section(self, section)             | 向文档对象中增加一个classify         |
| set(self, section, option, value=None) | 设置或添加文档对象的键值对           |
| remove_section(self, section)          | 删除文档对象中的的一个classify       |
| remove_options(self, section, option)  | 删除文档对象中的一组键值对           |
| has_section(self, section)             | 判断文档对象中的一个classify是否存在 |
| has_options(self, section, option)     | 判断文档对象中的一个key是否存在      |

个人更加倾向于通过字典的方式操纵文档对象。



# 特殊的DEFAULT

配置文件中有1个名为DEFAULT的classify，它提供了一些默认设置。

如果你的配置文件中没有显式的配置DEFAULT这个classify，则通过字典操作任然会获取到到它。但是若通过方法进行操作，该classify则不会出现。
