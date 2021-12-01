# collections简介

collections模块提供了许多容器的数据类型，是Python内置数据类型的一种升级。

[官方文档](https://docs.python.org/zh-cn/3.6/library/collections.html)

collections模块所提供的内置容器或者基类如下所示：

| 容器/基类                                                    | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [namedtuple()](https://docs.python.org/zh-cn/3.6/library/collections.html#collections.namedtuple) | 创建命名元组子类的工厂函数                                   |
| [deque](https://docs.python.org/zh-cn/3.6/library/collections.html#collections.deque) | 类似列表(list)的容器，实现了在两端快速添加(append)和弹出(pop) |
| [ChainMap](https://docs.python.org/zh-cn/3.6/library/collections.html#collections.ChainMap) | 类似字典(dict)的容器类，将多个映射集合到一个视图里面         |
| [Counter](https://docs.python.org/zh-cn/3.6/library/collections.html#collections.Counter) | 字典的子类，提供了可哈希对象的计数功能                       |
| [OrderedDict](https://docs.python.org/zh-cn/3.6/library/collections.html#collections.OrderedDict) | 字典的子类，保存了他们被添加的顺序                           |
| [defaultdict](https://docs.python.org/zh-cn/3.6/library/collections.html#collections.defaultdict) | 字典的子类，提供了一个工厂函数，为字典查询提供一个默认值     |
| [UserDict](https://docs.python.org/zh-cn/3.6/library/collections.html#collections.UserDict) | 封装了字典对象，简化了字典子类化                             |
| [UserList](https://docs.python.org/zh-cn/3.6/library/collections.html#collections.UserList) | 封装了列表对象，简化了列表子类化                             |
| [UserString](https://docs.python.org/zh-cn/3.6/library/collections.html#collections.UserString) | 封装了列表对象，简化了字符串子类化                           |



# ChainMap

ChainMap能够让多个字典链接起来，返回类似于字典视图的功能，可以直接将多个互相分离的字典当做1个大的整体字典来用，ChainMap支持所有字典方法，如get()，pop()等。

可能有的朋友会想，那为什么不新创建1个字典然后update()旧的字典呢？

这是因为ChainMap和字典视图很相似，所以oldDict的数据如果发生更新，则ChainMap也会同步进行更新。

但使用dict.update()创建的newDict就没有这种特性了，它不会随着oldDict的数据改变而发生改变。

如下图所示：

![image-20210530173011793](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210530173011793.png)







## 快速使用

如下示例，将链接2个字典，组成英文字母大小写对照ASCII码表：

```
from collections import ChainMap

uppercaseAlphabet = {chr(i): i for i in range(65, 91)}
lowercaseAlphabet = {chr(i): i for i in range(97, 123)}
letterTable = ChainMap(uppercaseAlphabet, lowercaseAlphabet)

print(letterTable)
```

生成的对象如下所示：

```
ChainMap({'A': 65, 'B': 66, 'C': 67, 'D': 68, 'E': 69, 'F': 70, 'G': 71, 'H': 72,
'I': 73, 'J': 74, 'K': 75, 'L': 76, 'M': 77, 'N': 78, 'O': 79, 'P': 80, 'Q': 81, 'R': 82, 'S': 83, 'T': 84, 'U': 85, 'V': 86, 'W': 87, 'X': 88, 'Y': 89, 'Z': 90}, {'a': 97, 'b': 98, 'c': 99, 'd': 100, 'e': 101, 'f': 102, 'g': 103, 'h': 104, 'i':
105, 'j': 106, 'k': 107, 'l': 108, 'm': 109, 'n': 110, 'o': 111, 'p': 112, 'q': 113, 'r': 114, 's': 115, 't': 116, 'u': 117, 'v': 118, 'w': 119, 'x': 120, 'y': 121, 'z': 122})
```





## maps

ChainMap的底层其实是用了1个列表，来存放了oldDict的引用。

格式如下：

```
[
	{oldDict1..},
	{oldDict2..},
	...
]
```

在查询时，如果oldDict1和oldDict2具有重复的key，则会查出oldDict1，因为它是挨个字典的向后进行查找，其他的操作也是同理：

![image-20210530173439119](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210530173439119.png)

如下示例，oldDict1和oldDict2都有为a的key，那么操纵ChainMap的a时必定先拿到oldDict1的a：

```
from collections import ChainMap

oldDict1 = {"a": 1, "b": 2, "c": 3}
oldDict2 = {"a": 10, "b": 20, "c": 30}
newMap = ChainMap(oldDict1, oldDict2)

print(newMap.get("a"))

# 1
```



如果直接想操作oldDict2的a，则可以通过ChainMap.maps拿到存储字典映射的列表，指定索引值来操作第2个字典也就是oldDict2，示例如下：

```
from collections import ChainMap

oldDict1 = {"a": 1, "b": 2, "c": 3}
oldDict2 = {"a": 10, "b": 20, "c": 30}
newMap = ChainMap(oldDict1, oldDict2)

print(newMap.maps[1].get("a"))

# 10
```



## new_child(m=None)

该方法会在旧的ChainMap上生成1个新的ChainMap，并且新增1个空字典在最前面。

如下图所示：

![image-20210530173942234](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210530173942234.png)



代码示例：

```
from collections import ChainMap

oldDict1 = {"a": 1, "b": 2, "c": 3}
oldDict2 = {"a": 10, "b": 20, "c": 30}
oldMap = ChainMap(oldDict1, oldDict2)
newMap = oldMap.new_child()

print(newMap)

# ChainMap({}, {'a': 1, 'b': 2, 'c': 3}, {'a': 10, 'b': 20, 'c': 30})
```

你也可以选择，在使用该方法的时候传入1个新的字典，让它填补第1个位置：

```
...
newMap = oldMap.new_child({"new": None})

print(newMap)

# ChainMap({'new': None}, {'a': 1, 'b': 2, 'c': 3}, {'a': 10, 'b': 20, 'c': 30})
```





## parents

返回ChainMap的父ChainMap，相较于子ChainMap来说，父ChainMap永远没有子ChainMap的第1个oldDict。

如下图所示：

![image-20210530174509917](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210530174509917.png)

示例如下：

```
from collections import ChainMap

oldDict1 = {"a": 1, "b": 2, "c": 3}
oldDict2 = {"a": 10, "b": 20, "c": 30}
oldMap = ChainMap(oldDict1, oldDict2)

print(oldMap.parents)

# ChainMap({'a': 10, 'b': 20, 'c': 30})
```



## 使用场景

在官方文档中举例了一个非常好的使用场景。

有1个简单的脚本，它拥有一些默认的参数变量。

当启动该脚本时，会有以下3种情况发生：

- 如果在命令行启动脚本时，指定了参数，则使用命令行指定的参数
- 如果没有在命令行启动脚本时指定参数，则会查找os的环境变量试图获取该参数
- 如果os的环境变量中也没有该参数，则使用默认的参数

这里它就是用了ChainMap来实现的，具体代码如下，避免了大量的if和else，非常方便：

```
import os
import argparse
from collections import ChainMap


defaults = {'color': 'red', 'user': 'guest'}

parser = argparse.ArgumentParser()
parser.add_argument('-u', '--user')
parser.add_argument('-c', '--color')
namespace = parser.parse_args()
command_line_args = {k: v for k, v in vars(namespace).items() if v}

combined = ChainMap(command_line_args, os.environ, defaults)

print(combined['color'])
print(combined['user'])
```

测试1，命令行传入了参数：

```
python3 demo.py -c Black -u Yunya
Black
Yunya
```

测试2，命令行和os的环境变量中都没有参数，则用默认的参数：

```
python3 demo.py
red
guest
```



# Counter

Counter能够快速的获取一个可迭代对象中每一个数据项所出现的次数。

因此可以用来做词频统计，排行榜等一类的工具。

Counter是Dict的派生类，故支持大部分的字典方法，如get()，pop()等。



## 序列记数

直接为Counter()传入1个可迭代对象，返回一个dict，包含数据项和出现次数，按照降序排列，如下所示：

```
from collections import Counter

seq = ["A", "A", "B", "C", "C", "D"]
c = Counter(seq)

print(c)
# Counter({'A': 2, 'C': 2, 'B': 1, 'D': 1})
```





## 计数获取

如果你想获取1个数据项的出现次数，可通过[]的操作或者get()方法来完成，像操纵字典一样操纵Counter即可。

值得一提的是，当Counter通过[item]来获取数据项出现次数的话，如果这个数据项不存在则会返回0，而不是抛出KeyError。

```
from collections import Counter

seq = ["A", "A", "B", "C", "C", "D"]
c = Counter(seq)

print(c.get("A"))
print(c.get("Z"))
print(c["Z"])

# 2
# None
# 0
```



## 排行获取

Counter.most_common()会返回1一个列表，按照Counter的排列顺序从大到小进行返回，如下所示，返回出现次数最多的3个数据项：

```
from collections import Counter

seq = ["A", "A", "B", "C", "C", "D"]
c = Counter(seq)

print(sorted(c.most_common(3)))

# [('A', 2), ('C', 2), ('B', 1)]
```

如果想返回出现次数最少的3个数据项该怎么办呢？

如下所示：

```
from collections import Counter

seq = ["A", "A", "B", "C", "C", "D"]
c = Counter(seq)

print(sorted(c.items(), key=lambda li:li[1])[:3])

# [('B', 1), ('D', 1), ('A', 2)]
```



## 词频统计

打开一个文件，做词频统计：

```
from collections import Counter


result = Counter()
with open("./data/input.txt","r") as f:
    while True:
        lines = f.read(1024).splitlines()
        if lines==[]:
            break
        lines = [lines[i].split(" ") for i in range(len(lines))]
        words = []
        for line in lines:
            words.extend(line)
        tmp = Counter(words)
        result+=tmp

print (result.most_common(10))
```





# deque

deque是collections模块提供的一大杀器，名为双端队列。

虽然普通的list也能够在队列2端进行数据项操作，但诸如insert(0, item)， pop(0)等方法都会引起数据项在内存中的挪动，从而使这2个方法的时间复杂度降低到O(n)。

而deque针对insert(0, item)和pop(0)做出了优化，让它们的时间复杂度都降到了O(1)。

如果仅在队首、队尾做操作，那么使用双端队列是最合适的。

如果要在队列中部做操作，还是推荐使用list。



## 方法一览

下面是deque所提供的方法和属性：

| 方法/属性                | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| deque(iterable, maxlen)  | 返回新的双端队列，可指定该队列的最大容量，如果不指定最大容量，则内部会根据数据项个数进行自动扩容 |
| append(item)             | 添加数据项至队尾                                             |
| appendleft(item)         | 添加数据项至队首                                             |
| clear()                  | 清空队列中的数据项                                           |
| copy()                   | 创建一份浅拷贝                                               |
| count(item)              | 返回item在队列中出现的次数                                   |
| extend(iterable)         | 通过附加来自可迭代对象的数据项来扩展队列，数据项添加至队尾   |
| extendleft(iterable)     | 通过附加来自可迭代对象的数据项来扩展队列，数据项添加至队首   |
| index(item, start, stop) | 返回第一个数据项在队列中出现位置的索引，若值不存在，则抛出ValueError，可指定start和stop的索引区间 |
| insert(index, item)      | 在索引之前插入数据项                                         |
| pop()                    | 删除并弹出队尾的数据项，若队列为空则抛出IndexError           |
| popleft()                | 删除并弹出队首的数据项，若队列为空则抛出IndexError           |
| remove(item)             | 删除队列中第一次出现的数据项。如果不存在该数据项，则引发ValueError |
| reverse()                | 翻转整个队列，返回None，即原地翻转                           |
| rotate(n=1)              | 如果为正数，将队尾n个数据项移动至队首，如果是负数，将队首n个数据项移至队尾 |
| maxlen                   | 返回队列的最大容量                                           |

除了以上方法和属性之外，deque还支持迭代，枚举，len(d), reversed(d), copy.copy(d), copy.deepcopy(d), 成员测试 [in](https://docs.python.org/zh-cn/3.6/reference/expressions.html#in) 操作符，和下标引用 d[-1]，但是不支持切片[::] 。

注意！len()和maxlen是不同的：

- len()取的是队列中以有多少个数据项
- maxlen取的是队列中最多可容纳多少数据项

如下所示：

```
from collections import deque

q = deque(range(3), maxlen=10)

print(q.maxlen)
print(len(q))

# 10
# 3
```

此外，rotate()方法可以在队列中进行数据项的位置迁移，如下所示。

n为正数，将队尾n个数据项移动至队首：

```
from collections import deque

q = deque(range(5), maxlen=10)

print(q)
q.rotate(2)
print(q)

# deque([0, 1, 2, 3, 4], maxlen=10)
# deque([3, 4, 0, 1, 2], maxlen=10)
```

n为负数，将队首n个数据项移至队尾：

```
from collections import deque

q = deque(range(5), maxlen=10)

print(q)
q.rotate(-2)
print(q)

# deque([0, 1, 2, 3, 4], maxlen=10)
# deque([2, 3, 4, 0, 1], maxlen=10)
```



# defaultdict

defaultdict本身是一个字典，继承了dict类并覆写了\_\_missing\_\_()方法。

在实例化defaultdict对象时我们可以为字典设置1个默认值，当使用[key]获取value时，若[key]不存在将会返回默认值而不是直接抛出keyError。



## 快速使用

如下所示，在使用了defaultdict后，用[key]操作获取value时若key不存在则会返回设定的默认值：

```
from collections import defaultdict

dic = defaultdict(lambda : None)  #　❶
dic["k1"] = "v1"
print(dic["k1"])
print(dic["k2"])

# v1
# None
```

❶：设定默认值，必须是1个可调用对象，将其返回值作为defaultdict的默认值



## 计数统计

统计数据项在列表中出现的次数，如果是普通的dict你可能需要这么做：

```
li1 = ["A", "A", "B", "C", "C", "D"]
dic = {}

for item in li1:
    dic.setdefault(item, 0)
    dic[item] += 1

print(dic)

# {'A': 2, 'B': 1, 'C': 2, 'D': 1}
```



如果是defaultdict，则可以让代码更精简一点：

```
from collections import defaultdict

li1 = ["A", "A", "B", "C", "C", "D"]
dic = defaultdict(lambda: 0)

for item in li1:
    dic[item] += 1

print(dic)

# defaultdict(<function <lambda> at 0x0127B6A8>, {'A': 2, 'B': 1, 'C': 2, 'D': 1})
```





# namedtuple

namedtuple翻译过来就是具名元组，因为元组的主要功能是数据的展示，如果能够将1个元组中每个数据项的意思也表达出来就更好了。

如下所示，一个普通的元组：

```
("Jack", 18, "male", "123456")
```

乍一看前3个你可能都能看懂是什么意思，那么最后1个呢？是不是一脸懵逼？

而通过具名元组，你就能知道最后1个的意思了，如下所示：

```
(name='Jack', age=18, gender='male', password='123456')
```

可能这里有的同学会说，那我为什么不用dict？而偏要这么麻烦的用collections中的namedtuple？

别搞忘了，dict是可变类型，namedtuple则是继承了tuple的特性，是不可变的，数据仅作展示时使用具名元组是最好的选择。

此外，具名元组是元组的子类，所以可以使用所有元组的方法，除此之外它还新增了一些方法和属性。



## 对象创建

使用namedtuple()方法来初始化一个类。

函数签名如下：

```
def namedtuple(typename, field_names, *, verbose=False, rename=False, module=None):
	pass
```

参数释义：

- typename：将要实例化出的类的名称
- field_names：具名元组中每个字段的名称
- ...

示例演示：

```
from collections import namedtuple

UserMessage = namedtuple("userMessage", ["name", "age" ,"gender", "password"])
u = UserMessage("Jack", 18, "male", "123456")
print(u) 

# userMessage(name='Jack', age=18, gender='male', password='123456')
```



## _make()

若1个序列的数据项个数与具名元组类的字段个数相同，通过该方法可直接将这个序列传入并生成新的具名元组对象。

如下所示，u的数据项个数与具名元组类的字段个数相同，直接根据u创建1个具名元组对象：

```
from collections import namedtuple

UserMessage = namedtuple("userMessage", ["name", "age" ,"gender", "password"])
u = ("Jack", 18, "male", "123456")
print(UserMessage._make(u))

# userMessage(name='Jack', age=18, gender='male', password='123456')
```



## _asdict()

将具名元组对象转换为有序字典（注：不是dict，而是Orderdict）：

```
from collections import namedtuple

UserMessage = namedtuple("userMessage", ["name", "age" ,"gender", "password"])
u = UserMessage("Jack", 18, "male", "123456")
print(u._asdict())

# OrderedDict([('name', 'Jack'), ('age', 18), ('gender', 'male'), ('password', '123456')])
```



## _replace()

由于具名元组不可改变，所以通过该方法会生成1个新的具名元组，用于替换旧的具名元组中的某一个数据项值：

```
from collections import namedtuple

UserMessage = namedtuple("userMessage", ["name", "age" ,"gender", "password"])
oldUser = UserMessage("Jack", 18, "male", "123456")
newUser = oldUser._replace(name="Tom", password="abcdef")
print(newUser)

# userMessage(name='Tom', age=18, gender='male', password='abcdef')
```



## _fields

返回具名元组的字段列表：

```
from collections import namedtuple

UserMessage = namedtuple("userMessage", ["name", "age" ,"gender", "password"])
u = UserMessage("Jack", 18, "male", "123456")
print(u._fields)

# ('name', 'age', 'gender', 'password')
```



## 字典转换

若想将具名元组转换为普通字典，可通过如下方法：

```
from collections import namedtuple

UserMessage = namedtuple("userMessage", ["name", "age" ,"gender", "password"])
u = UserMessage("Jack", 18, "male", "123456")
print(
    dict(zip(u._fields, u[:]))
    )

# {'name': 'Jack', 'age': 18, 'gender': 'male', 'password': '123456'}
```

若想将普通字典转换为具名元组，可通过如下方法：

```
from collections import namedtuple

userDict = {'name': 'Jack', 'age': 18, 'gender': 'male', 'password': '123456'}
UserMessage = namedtuple("userMessage", ["name", "age" ,"gender", "password"])
print(UserMessage(**userDict))

# userMessage(name='Jack', age=18, gender='male', password='123456')
```





# OrderDict

OrderDict见字生意，即有序字典。

但是Python3.6之后字典已经变的有序了，所以这里不再举例它的用法。

感兴趣可以参照collections的官方文档进行查阅。





# 继承基类

collections中提供了3个基类，分别是UserList，UserDict，UserString。

它们并没有什么实质性的功能，只是针对list、dict、string的C语言实现用Python重写了一遍。

如果你对这些数据类型的实现比较感兴趣，可翻阅一下它们的源码。

此外，如果在自定义序列时，也建议继承这3个类，而不是继承内置的3个类。
