# importlib 简介

importlib 模块作为 Python 内置模块，提供了更多导入模块的方式。

[官方文档](https://docs.python.org/zh-cn/3.6/library/importlib.html)

常用方法：

- importlib.import_module(str)：根据字符串导入 1 个模块，该字符串以.为路径分割，如"package.module"

# 项目示例

假设我的项目之中包含了多个中间件，并且这些中间件会在项目正式启动之前先行启动。

我该如何集中式的管理这些中间件，让它们在需要的时候能够快速加上，不需要的时候能够快速去除呢？

使用 importlib 模块是最明智的选择，整个项目目录如下：

```
PYTHONPROJECT
│
├─bin
│  |  run.py
│
├─middleware
│  │  first_middle.py
│  │  init.py
│  │  second_middle.py
│  │  __init__.py
│
├─view
│   │  main.py
│
│  settings.py
```

首先先查看一下 run.py，它主要处理项目模块路径、中间件初始化以及主程序的运行：

```
#　run.py

import os
import sys

BASE_DIR =  os.path.dirname(os.path.dirname(__file__))
sys.path.append(BASE_DIR)

from middleware.init import init
from view.main import main

if __name__ == "__main__":
    # init middleware
    init()
    # run main
    main()
```

再来看一下中间件的启动，它会从配置文件中拿到被集中化管理的需要启动的中间件，然后利用 importlib 模块对其进行导入后执行其下的某一个方法：

```
# middleware/init.py

import importlib
import settings

def init():
    for stringPath in settings.LOAD_MIDDLEWARE:
        modulePath, funcName = stringPath.rsplit(".", maxsplit=1)
        # 利用importlib模块根据字符串路径导入模块
        module = importlib.import_module(modulePath)
        # 模块也是对象，所以利用反射拿到函数对象并执行
        funcObject = getattr(module, funcName)
        funcObject()
```

被集中管理的需要加载的中间件被定义在了 settings.py 的 1 个列表中。

其中每条数据项代表 1 个需要被加载的中间件，以 BASE_DIR 为准，用.进行分割，确定了中间件的导入路径与运行函数：

```
# settings.py

LOAD_MIDDLEWARE = [
    "middleware.first_middle.m1",
    "middleware.second_middle.m2"
]
```

当后续需要新增中间件，只需要在 middleware 包中添加好.py 文件并且在 settings.py 中按照格式把路径和运行函数填入即可。

如果要取消某个中间件的加载，直接在列表中对它进行注释即可。

总结 2 点：

- 如何规定模块导入的格式，参见 LOAD_MIDDLEWARE 列表
- 如何使用 importlib 快速导入模块且执行模块下的某一个函数，参见 init.py 文件

把其他代码也补上吧，中间件的启动函数：

```
# middleware/first_middle.py
def m1():
    print("middleware 1 run..")

# middleware/second_middle.py
def m2():
    print("middleware 2 run..")
```

主程序函数：

```
# view/main.py

def main():
    print("view main run...")
```

运行结果：

```
middleware 1 run..
middleware 2 run..
view main run...
```
