# sys简介

sys模块是Python内置模块，提供了各种系统相关的参数和函数。

[官方文档](https://docs.python.org/zh-cn/3.6/library/sys.html)

以下举例部分常用方法和属性：

| 方法/属性                          | 描述                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| sys.platform                       | 返回操作系统平台名称                                         |
| sys.version                        | 获取Python解释程序的版本信息                                 |
| sys.builtin_module_names           | 获取内置的所有模块名，元组形式返回                           |
| sys.modules                        | 返回以加载至内存之中的模块及路径                             |
| sys.path                           | 返回模块在硬盘中的搜索路径                                   |
| sys.stdin                          | Python标准输入通道，input()函数的底层实现                    |
| sys.stdout                         | Python标准输出通道，print()函数的底层实现                    |
| sys.stderr                         | Python标准输入错误通道                                       |
| sys.getrecursionlimit()            | 获取当前Python中最大递归层级                                 |
| sys.setrecursionlimit()            | 设置当前Python中最大递归层级                                 |
| sys._getframe(0).f_code.co_name    | 获取被调用函数的名称                                         |
| sys._getframe(1).f_code.co_name    | 获取被调用函数是被哪一个函数所嵌套调用的，若不是被嵌套调用则返回module |
| sys._getframe().f_back.f_lineno    | 获取被调用函数在被调用时所处代码行数                         |
| sys._getframe().f_code.co_filename | 获取被调用函数所在模块文件名                                 |
| sys.getrefcount()                  | 获取对象的引用计数                                           |
| sys.argv                           | 获取通过脚本调用式传递的数据                                 |





# 修改递归层级

修改递归层级已经介绍过一次了，默认Python的最大递归层级是1000层，我们可以对其进行修改：

```
>>> sys
>>> sys.getrecursionlimit()
1000
>>> sys.setrecursionlimit(10000)
>>> sys.getrecursionlimit()
10000
```



# 函数栈帧信息

sys._getframe()能够获取到函数的栈帧对象，我们知道函数的栈帧对象中封存了一些函数运行时的信息。

那么通过下面这些属性就能拿到函数里栈帧的某些数据：

- sys._getframe(0).f_code.co_name：获取被调用函数的名称
- sys._getframe(1).f_code.co_name：获取被调用函数是被哪一个函数所嵌套调用的，若不是被嵌套调用则返回module
- sys._getframe().f_back.f_lineno：获取被调用函数在被调用时所处代码行数
- sys._getframe().f_code.co_filename：获取被调用函数所在模块文件名

```
import sys


def func():
    print(sys._getframe(0).f_code.co_name)
    print(sys._getframe(1).f_code.co_name)
    print(sys._getframe().f_back.f_lineno)
    print(sys._getframe().f_code.co_filename)

func()
```





# 脚本传入参数

我们都知道Python解释器可以通过以下方式进行.py文件的调用：

```
$ python3 demo.py
```

但是你可能不知道通过sys.argv属性可以获取通过脚本调用式传递的数据，如下启动.py脚本时传入了1、2、3：

```
$ python3 demo.py  1 2 3
```

那么现在sys.argv就会接受到这3个数据，变成下面的格式：

```
sys.argv = ["scriptPath", "1", "2", "3"]
```

基于这个特性，我们来做一个下载模拟器：

```
import random
import sys
import time


def download():
    scale = 40
    print("开始下载文件:{0}".format(sys.argv[2]).center(scale + 10, '-'))
    totalSize = random.randint(1000, 2000)
    currentSize = totalSize / scale
    for i in range(scale + 1):
        a = '█' * i
        b = '_' * (scale - i)
        c = (i / scale) * 100
        print('''\r{0:^3.2f}% | {1}{2} | {3:.2f}/{4:.2f}(MB)'''.format(c, a, b, currentSize * i, totalSize), end="")
        time.sleep(0.1)
    print("\n" + "执行结束".center(scale + 10, '-'))


def main():
    try:
        runFunc = eval(sys.argv[1])
    except Exception:
        print("输入有误！", file=sys.stderr)
        exit()
    else:
        runFunc()


if __name__ == '__main__':
    main()
```

启动时输入参数：

```
 python3 ./bin/run.py download test.text
```

