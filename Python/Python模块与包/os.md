# os 简介

os 模块是 Python 内置模块，提供了各种使用 Python 对操作系统提供操纵的接口。

[官方文档](https://docs.python.org/zh-cn/3.6/library/os.html)

以下举例部分常用方法和属性：

| 方法/属性                        | 描述                                                                                                                          |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| os.curdir                        | 总是返回一个字符串，“.”，代指当前目录                                                                                         |
| os.pardir                        | 总是返回一个字符串，“..”，代指当前父级目录                                                                                    |
| os.sep                           | 返回当前平台下的路径分隔符，Windows 下为“\”，Unix 下为“/”                                                                     |
| os.linesep                       | 返回当前平台下的行终止符，Windows 下为“\r\n”，Unix 下为“\n”                                                                   |
| os.pathsep                       | 返回当前平台下的用于分割文件的分隔符，Windows 下为“;”，Unix 下为“:”                                                           |
| os.name                          | 返回当前平台的信息，Windows 下为“nt”，Unix 下为“posix”                                                                        |
| os.environ                       | 获取系统环境变量                                                                                                              |
| os.system(“command”)             | 运行 shell 命令                                                                                                               |
| os.listdir(“dirName”)            | 获取指定目录下的所有项目，相当于 ls 命令，以列表方式返回结果                                                                  |
| os.getcwd()                      | 获取当前脚本的工作目录，相当于 pwd 命令                                                                                       |
| os.chdir(“dirName”)              | 改变当前脚本的工作目录，相当于 cd 命令                                                                                        |
| os.makedir(“dirName”)            | 生成单级的空目录，相当于 mkdir 命令                                                                                           |
| os.makedirs(“dirName1/dirName2”) | 生成多层递归目录，相当于 mkdir -p 命令, 默认当目录已经存在则抛出异常, 若 exist_ok 为 True, 则不会抛出异常，默认该参数为 False |
| os.rmdir(“dirName”)              | 删除单级的空目录                                                                                                              |
| os.removedirs(“dirName”)         | 删除多层递归目录，前提是该目录必须为空                                                                                        |
| os.remove(“fileName”)            | 删除一个文件                                                                                                                  |
| os.rename(“oldName”, “newName”)  | 重命名文件/目录                                                                                                               |
| os.path.abspath(“path”)          | 返回当前 path 的绝对路径                                                                                                      |
| os.path.split(“path”)            | 将 path 分为 2 部分，返回元组，索引 0 是路径，索引 1 是文件                                                                   |
| os.path.dirname(“path”)          | 返回 path 的路径部分，相当于上面方法的索引 0                                                                                  |
| os.path.basename(“path”)         | 返回 path 的路径部分，相当于上面方法的索引 1，如果 path 是以“/”或者“\”结尾，则返回 None                                       |
| os.path.join(“path1”, “path2”)   | 将多个 path 进行组合，相当于 os.path.split()的反操作                                                                          |
| os.path.exists(“path”)           | 判断 path 是否存在，返回布尔值                                                                                                |
| os.path.isabs(“path”)            | 判断 path 是否是绝对路径，返回布尔值                                                                                          |
| os.path.isfile(“path”)           | 判断 path 是否是一个文件路径，返回布尔值                                                                                      |
| os.path.isdir(“path”)            | 判断 path 是否是一个目录路径，返回布尔值                                                                                      |
| os.stat(“path”)                  | 获取 path 所指文件/目录的相关信息                                                                                             |
| os.path.getatime(“path”)         | 获取 path 所指文件/目录的最后存取时间                                                                                         |
| os.path.getmtime(“path”)         | 获取 path 所指文件/目录的最后修改时间                                                                                         |
| os.path.getsize(“path”)          | 获取 path 所指文件/目录的大小                                                                                                 |

# 平台信息

根据以下一些属性，可获取平台信息：

| 属性       | 描述                                                                |
| ---------- | ------------------------------------------------------------------- |
| os.curdir  | 总是返回一个字符串，“.”，代指当前目录                               |
| os.pardir  | 总是返回一个字符串，“..”，代指当前父级目录                          |
| os.sep     | 返回当前平台下的路径分隔符，Windows 下为“\”，Unix 下为“/”           |
| os.linesep | 返回当前平台下的行终止符，Windows 下为“\t\n”，Unix 下为“\n”         |
| os.pathsep | 返回当前平台下的用于分割文件的分隔符，Windows 下为“;”，Unix 下为“:” |
| os.name    | 返回当前平台的信息，Windows 下为“nt”，Unix 下为“posix”              |
| os.environ | 获取系统环境变量                                                    |

os.environ 是一个全局字典，你可以将它当做普通字典进行操作。

```
>>> os.environ
...
>>> os.environ["k1"] = "v1"
>>> os.environ.get("k1")
'v1'
```

此外，它是全局的，这意味着同一个项目之中任何地方都能随时获取到它，因此可以用它来存储一些较为私密的信息，如数据库链接 IP+PORT+USER+PASSWORLD。

使用该字典时需要注意 key 必须是 str 类型，若是其他类型则会抛出异常。

# 目录操作

使用以下一些方法，可对目录做出操作：

| 方法                             | 描述                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| os.system(“command”)             | 运行 shell 命令                                              |
| os.listdir(“dirName”)            | 获取指定目录下的所有项目，相当于 ls 命令，以列表方式返回结果 |
| os.getcwd()                      | 获取当前脚本的工作目录，相当于 pwd 命令                      |
| os.chdir(“dirName”)              | 改变当前脚本的工作目录，相当于 cd 命令                       |
| os.makedir(“dirName”)            | 生成单级的空目录，相当于 mkdir 命令                          |
| os.makedirs(“dirName1/dirName2”) | 生成多层递归目录，相当于 mkdir -p 命令                       |
| os.rmdir(“dirName”)              | 删除单级的空目录                                             |
| os.removedirs(“dirName”)         | 删除多层递归目录，前提是该目录必须为空                       |
| os.remove(“fileName”)            | 删除一个文件                                                 |
| os.rename(“oldName”, “newName”)  | 重命名文件/目录                                              |

os.system()应该是一个比较常用的方法，它可以运行任何的 shell 命令：

```
>>> os.system("tree .")
>>> os.system("ifconfig")
```

但是 os.system()如果在 Windows 环境下的 PyCharm 中进行使用，则会抛出异常。

因为 Windows 平台执行命令的返回结果是采用 GBK 编码，而 PyCharm 中使用 UTF8 对结果进行解码就会产生乱码问题。

# 路径操作

路径操作应该是 os 模块中比较常用的：

| 方法                           | 描述                                                                                    |
| ------------------------------ | --------------------------------------------------------------------------------------- |
| os.path.abspath(“path”)        | 返回当前 path 的绝对路径                                                                |
| os.path.split(“path”)          | 将 path 分为 2 部分，返回元组，索引 0 是路径，索引 1 是文件                             |
| os.path.dirname(“path”)        | 返回 path 的路径部分，相当于上面方法的索引 0                                            |
| os.path.basename(“path”)       | 返回 path 的路径部分，相当于上面方法的索引 1，如果 path 是以“/”或者“\”结尾，则返回 None |
| os.path.join(“path1”, “path2”) | 将多个 path 进行组合，相当于 os.path.split()的反操作                                    |
| os.path.exists(“path”)         | 判断 path 是否存在，返回布尔值                                                          |
| os.path.isabs(“path”)          | 判断 path 是否是绝对路径，返回布尔值                                                    |
| os.path.isfile(“path”)         | 判断 path 是否是一个文件路径，返回布尔值                                                |
| os.path.isdir(“path”)          | 判断 path 是否是一个目录路径，返回布尔值                                                |

示例演示 os.path.join()和 os.path.split()：

```
>>> dirName, fileName = os.path.split("/Users/yunya/document/os模块.md")
>>> dirName
'/Users/yunya/document'
>>> fileName
'os模块.md'
>>> newPath = os.path.join("/", "Users", "yunya", "document", "os模块.md")
>>> newPath
'/Users/yunya/document/os模块.md'
>>>
```

# 信息获取

信息获取也有时候会用到：

| 方法                     | 描述                                  |
| ------------------------ | ------------------------------------- |
| os.stat(“path”)          | 获取 path 所指文件/目录的相关信息     |
| os.path.getatime(“path”) | 获取 path 所指文件/目录的最后存取时间 |
| os.path.getmtime(“path”) | 获取 path 所指文件/目录的最后修改时间 |
| os.path.getsize(“path”)  | 获取 path 所指文件/目录的大小         |

# 项目模块查找

## 项目启动不了?

在之前介绍 Python 模块一章节中说到 PyCharm 和原生解释器在查找模块时的 sys.path 会有所不同。

PyCharm 会自动的新增几行模块查找路径，而原生解释器则不会进行新增。

这样会产生一个问题，即项目上线后通过原生解释器进行启动项目时会发现找不到模块。

如，我们有一个下面结构的项目：

```
Project
│
├── bin
│   └── run.py          # 入口文件
└── view
    ├── __init__.py
    └── views.py        # 视图层 定义了main()函数
```

当 run.py 进行执行后，Python 工作目录就被定义在了 Project/bin/run.py 一层。

```
from view.views import main

if __name__ == '__main__':
    main()
```

如果在 PyCharm 中执行 run.py，则不会抛出异常，它能顺利的找到 view 模块，这是因为 PyCharm 将工作目录的上层、上上层也加入到了 sys.path 即模块查找路径中：

```
[
'/Users/yunya/PycharmProjects/Project/bin',
'/Users/yunya/PycharmProjects/Project',
'...'
]

```

这样查找模块范围就会大很多：

```
Project  # 上上层能找到
│
├── bin  # 上层找不到
│   └── run.py   # 本层找不到
└── view
    ├── __init__.py
    └── views.py
```

但是如果在 Python 原生解释器环境下，调用执行 run.py 脚本，则会提示找不到 view 模块，因为 view 模块仅能在上上层被找到：

```
Project  # 这里才能找到
│
├── bin
│   └── run.py     # 找不到
└── view
    ├── __init__.py
    └── views.py
```

如何解决这个问题？只需要在 run.py 脚本中将上上层路径加入至 sys.path 即可：

```
import sys
import os
sys.path.append(os.path.abspath(os.path.dirname(os.path.dirname(__file__))))

from view.views import main


if __name__ == '__main__':
    main()
```

这样再次使用 Python 原生解释器通过 run.py 脚本启动项目，就不会发生任何问题了。

## OpenStack 对路径的处理

在 OpenStack 中，对这种项目模块查找路径的处理采用了截然不同的方式：

```
import os
import os, sys

print(os.path.abspath(__file__))
possible_topdir = os.path.normpath(os.path.join(  # ❶
    __file__,
    os.pardir,  # 上一级，相当于手动输入".."
    os.pardir,
))
print(possible_topdir)
```

❶：os.path.normpath()可以将一个不规范的路径变为规范路径

这样也能够达到相同的效果。

## Django3.x 对路径的处理

Django3 以前，对项目模块查找路径的处理采用了和我们相同的方式：

```
BASE_DIR = os.path.dirname(os.path.dirname(__file__))
print(BASE_DIR)
```

在 Django3 之后，则使用 pathlib 模块代替了 os 模块，其实本质都是一样的：

```
from pathlib import Path
root = Path(__file__)
res = root.parent.parent # ❶
print(res)
```

❶：取上层的上层

补充一点 pathlib 的知识，对于 pathlib 的路径拼接直接使用 / 符号即可，符号左边为 Path 对象，右边为 str 类型。

与 os.path.join()拥有相同的效果。

```
print(Path("User/YunYa") / r"a/b/c")

# User\YunYa\a\b\c
```

无论是 os.path.join()还是 pathlib 的/进行路径拼接，都会选择出适合当前平台的路径分隔符。

## \_\_package\_\_ 变量

Python 内置了一个 build-in global variable \_\_package\_\_，有的时候我们必须手动区分当前文件是不是被模块的方式导入的。

若当前文件是以模块形式导入的，则 \_\_package\_\_ 为当前模块的名称。

若当前文件并非以模块形式导入的，则 \_\_package\_\_ 为 None。

所以，对于一个启动文件来说，我们可以:

```
import os
import sys

if not __package__:
    # sys.path.append(os.path.join(os.path.dirname(__file__), os.pardir))  效果和下面一样的
    sys.path.append(os.path.abspath(os.path.dirname(os.path.dirname(__file__))))
```
