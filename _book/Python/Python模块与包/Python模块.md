# Python模块

Python中的模块是一系列功能的集合体，总计可分为3大类：

- 内置模块：Python自带的一些内置库，开箱即用
- 第三方模块：通过pip命令下载到的模块，无需自己定义，Python拥有海量的第三方模块
- 自定义模块：一个.py文件就是一个模块，所以自己编写的.py文件也可以当做模块使用，例如文件名为m.py的文件就是一个名为m的自定义模块



# 自定义模块

自定义模块即自己写的模块，好处如下：

- 将相同功能的代码进行分类
- 降低代码耦合度，减少代码冗余
- 使整个程序组织结构更清晰，便于后期维护



## 结构分类

由于一个.py文件即为一个模块，故我们可以创建2个.py文件，一个当做程序入口文件，一个当做功能模块文件，以下是结构图：

```
.
├── m1.py    # 功能模块文件
└── run.py   # 程序入口文件
```

首先在m1.py中写一个基本的函数：

```
# m1.py

print("this is module m1")
def add(x, y):
    return x + y
```

其次是在run.py中写上一个打印语句：

```
# run.py

print("run ..")
```



## import简单使用

如果run.py中想要使用到m1.py中的add()函数，该怎么做？

只需要在run.py中导入m1.py里定义的add()函数即可，注意在使用时也必须按m1开头才行：

```
# run.py

import m1 # ❶

print("run ..")
result = m1.add(1, 2) # ❷
print(result)

# this is module m1
# run ..
# 3
```

❶：当解释器发现import m1时，会查找m1.py文件，并且会执行m1.py中的所有代码，所以下面会打印出 this is module m1的字样

❷：使用了m1中的add()函数



针对❶，提出一个问题，如果导入多次这个m1文件，是否也会执行多次其中的代码呢？

结果是否，也就是说只有第一次导入模块时，才会执行模块中的代码，多次导入只执行一次，其根本原因参照Python模块查找一节。

```
# run.py

import m1
import m1
import m1

print("run ..")
result = m1.add(1, 2)
print(result)

# this is module m1 ❶
# run ..
# 3
```



## 模块命名空间

现在我们有2个模块文件：

```
.
├── m1.py
├── m2.py
└── run.py
```

且2个模块文件中的代码都大部分相似：

```
# m1.py

print("this is module m1")
def add(x, y):
    return x + y
```

```
# m2.py

print("this is module m2")
def add(x, y):
    return x + y
```

在run中导入2个模块，且分别使用其下的add()函数时，内部发生了什么事情？

```
# run.py

import m1
import m2

print("run ..")
resultM1 = m1.add(1, 2)
resultM2 = m2.add(1, 2)

print(resultM1)
print(resultM2)

# this is module m1
# this is module m2
# run ..
# 3
# 3
```

1. run.py执行时，会按照import的顺序执行m1.py和m2.py文件中的代码
2. 当m1，m2执行完成之后，会产生一个模块的命名空间
3. run.py的全局命名空间中将产生2个标识符，分别指向了m1.py和m2.py的模块命名空间

模块命名空间如下所示：

![image-20210522141941653](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210522141941653.png)

当要使用m1.add()时，则run.py通过全局命名空间中的标识符m1，去m1模块的命名空间中查找函数标识符add。

当要使用m2.add()时，则run.py通过全局命名空间中的标识符m2，去m2模块的命名空间中查找函数标识符add。



## \_\_name\_\_与\_\_main\_\_

当一个模块编写完成后，将要对其进行测试工作，确保代码无误才能投入使用。

如下，m1的测试代码写上：

```
# m1.py

print("this is module m1")
def add(x, y):
    return x + y

# test
print(add(1, 2))
```

测试没问题后，run.py中对其进行功能引用：

```
# run.py

import m1
import m2

print("run ..")
resultM1 = m1.add(1, 2)
resultM2 = m2.add(1, 2)
print(resultM1)
print(resultM2)

# this is module m1
# 3  ❶
# this is module m2
# run ..
# 3
```

当运行run.py后，会发现1处多打印了个3，这是因为在执行m1模块时，也将测试代码给执行了。

如何避免这种问题？我们可以在m1.py中加上一个判断语句：

```
# m1.py

print("this is module m1")
def add(x, y):
    return x + y

# test
if __name__ == "__main__":  # ❶
    print(add(1, 2))
```

❶：\_\_name\_\_：如果该.py文件当做脚本被执行，则该变量为\_\_main\_\_，如果该.py文件当做模块导入被执行，则该变量为.py文件的名字，如m1.py就是m1

所以说，加入这条测试语句的目的在于，.py文件在不同的方式使用时可以执行不同的代码：

- 当做脚本被执行时，会运行测试代码
- 当做模块被导入时，不会运行测试代码



# 模块导入

## import ..

import语句的使用方式：

- import 模块名
- 导入的最小单元是模块

使用import导入模块的优缺点：

- 优点：不会和当前的全局命名空间标识符产生冲突
- 缺点：在使用模块功能时必须加上import右边的标识符前缀

如下所示，2个不同模块的相同标识符函数并不会产生冲突：

```
import time
import datetime

print(time.time())
print(datetime.time())

# 1621665686.779634
# 00:00:00
```

也可以一行导入多个模块，使用逗号进行分割：

```
import time, datetime
```





## from .. import ..

from语句的使用方式：

- from 模块名 import 标识符
- 导入的最小单元是具体功能

使用from语句导入模块的优缺点：

- 优点：在使用模块功能时必须加上import右边的标识符前缀，如果直接导入了一个具体功能，则不用加前缀
- 缺点：容易和当前的全局命名空间标识符产生冲突

如下示例，由于datetime模块后导入，所以它的time函数标识符替代了time模块的time函数标识符：

```
from time import time
from datetime import time

print(time())
print(time())

# 00:00:00
# 00:00:00
```

也可以在一行导入同一模块下的多个功能，以逗号进行分割：

```
from time import time, sleep, ctime
```



## 别名的使用

使用as语句来为冲突的标识符取一个别名：

```
from time import time as ttime
from datetime import time as dtime

print(ttime())
print(dtime())

# 1621665953.605452
# 00:00:00
```



## \*与\_\_all\_\_

使用from 模块名 import \*的方式，可以导入该模块下的所有标识符。

如果你是该模块的开发者，则可以通过\_\_all\_\_属性规定这种导入方式允许哪些标识符被导入。

- 在\_\_all\_\_中的标识符，可以被from 模块名 import \*的方式进行导入
- 未在\_\_all\_\_中的标识符，不会被from 模块名 import \*的方式进行导入
- 如果未定义\_\_all\_\_属性，则所有的标识符都会from 模块名 import \*的方式进行导入



如下，在m1.py模块文件中，定义了1个getMax()的接口暴露函数，此外还有1个内部处理函数computeMax()以及模块说明变量desc：

```
# m1.py

def getMax(iterable):
    currentMax = None
    for index, item in enumerate(iterable):
        if index == 0:
            currentMax = computeMax(item, iterable[index + 1])
        elif currentMax != item:
            currentMax = computeMax(currentMax, item)
    return currentMax


def computeMax(x, y):
    return x if x > y else y


desc = "this is module m1"

__all__ = ("getMax", "desc")  # ❶
```

❶：\_\_all\_\_的格式必须是Tuple(str, str)



现在run.py中如果使用from m1 import \*，则会将\_\_all\_\_中的所有标识符进行导入，下面示例中由于使用了未在\_\_all\_\_中定义的标识符，则抛出NameError的异常：

```
# run.py

from m1 import *

print(getMax)
print(desc)

print(computeMax)

# <function getMax at 0x102bff6a8>
# this is module m1
# NameError: name 'computeMax' is not defined
```



## 循环导入问题

模块a中导入模块b，模块b中又导入了模块a，且导入语句都在首行，此时将引发循环导入的问题。

示例如下：

```
# run.py

import m1

print(m1.desc)
```

```
# m1.py

import m2

desc = "this is module m1"

print(m2.desc)
```

```
# m2.py

import m1

desc = "this is module m2"

print(m1.desc)
```

运行run.py，结果如下：

```
AttributeError: module 'm2' has no attribute 'desc'
```

异常原因在于：

run.py首行导入了m1，m1首行导入了m2，m2首行又导入了m1，导致m1.desc未能成功进行对象声明，故抛出异常。

执行步骤：

1. run.py：import m1 (将m1加载至内存中)
2. m1.py：import m2 (将m2加载至内存中)
3. m2.py：import m1 (m1已经执行了，不重复执行了)
4. m2.py：desc = "this is module m2”
5. m2.py：print(m1.desc)）(m1.desc没有进行对象声明，抛出异常)



解决办法：

- 将m1导入m2的语句放在行尾
- 将m1导入m2的语句放入函数中，并在行尾执行函数



办法1：

```
# m1.py

desc = "this is module m1"

import m2

print(m2.desc)
```

办法2：

```
desc = "this is module m1"

def importFunction():
    import m2
    print(m2.desc)

importFunction()
```





# 模块查找

## 查找优先级

无论是from .. import ..语句还是import语句，在导入模块时都会涉及到模块位置查找的问题。

模块查找优先级如下：

- 先查找内存
- 后查找硬盘

当一个模块被导入过一次后，就会加载至内存中，重复导入便可直接从内存中拿到该模块，而存在于内存中的模块代码是不会被执行的。

## sys.modules

sys.modules用于查看存在于内存中的模块，如果要导入的模块存在于这里面，就直接进行导入，而不执行其中的代码：

```
>>> import sys
>>> sys.modules
...
```

当一个存在于硬盘之上的模块被导入时，则会将该模块加载至内存中，只要是存在于内存中的模块，重复导入时就不会执行其中的代码了。

如下示例，第一次导入存在于硬盘之上的m1模块后，它被加载至了内存中：

```
>>> tuple(sys.modules.items())[-1]
('rlcompleter', <module 'rlcompleter' from '/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/rlcompleter.py'>)

>>> import m1
>>> tuple(sys.modules.items())[-1]
(('m1', <module 'm1' from '/Users/yunya/PycharmProjects/Python基础/m1.py'>))
```

Ps：其实当Python解释器启动时，会自动的运行一遍所有的内置模块并加载至内存中，因为这些内置模块也是存放在磁盘下的，你可以在Python解释器安装根目录的lib目录下找到它们。



## sys.path

当内存中没有模块路径时，将按照sys.path的路径顺序依次在磁盘中查找。

如果在PyCharm下打印sys.path，它会做一些优化处理，比原生的REPL环境多出一些查找路径，下面使用#号进行标注：

```
[
# '/Users/yunya/PycharmProjects/Project',  
# '/Users/yunya/PycharmProjects/Project', 
# '/Applications/PyCharm.app/Contents/plugins/python/helpers/pycharm_display', 

'/Library/Frameworks/Python.framework/Versions/3.6/lib/python36.zip', '/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6', '/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/lib-dynload', '/Users/yunya/Library/Python/3.6/lib/python/site-packages', '/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages',   

# '/Applications/PyCharm.app/Contents/plugins/python/helpers/pycharm_matplotlib_backend'
]

```

原生的REPL环境打印：

```
>>> import sys
>>> sys.path
[
'', 
'/Library/Frameworks/Python.framework/Versions/3.6/lib/python36.zip', '/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6', '/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/lib-dynload', '/Users/yunya/Library/Python/3.6/lib/python/site-packages', '/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages'
]
>>>
```



# 模块导入规范

导入模块时要遵循的一些规范：

- 导入顺序：内置模块在最上面，第三方模块在中间，自定义模块在下面

- 自定义模块名风格：蛇形式命名

    Ps：Python2中有些模块是驼峰式命名，但是在Python3中都更改为蛇形式命名了，如PyMySQL，更名为pymysql

此外，模块也是一等公民，运行被赋值、传参等等。

# 模块编写规范

如果要编写自定义模块，也需要遵循一些规范：

- 首行添加模块文档描述，让别人知道你的模块是干嘛的
- 减少全局变量的使用，这样在第一次导入模块时会加快导入速度
- 模块中的类、函数等都需要写好注释
- 使用 if \_\_name\_\_ == “\_\_main\_\_”:的语句写好测试案例