# functools简介

functools是非常强大的内置模块，它提供了许多装饰器与函数，适用于对所有可调用对象的应用。

[官方文档](https://docs.python.org/zh-cn/3.6/library/functools.html)

这里主要着重介绍2种常用的函数与装饰器，它们适用于绝大部分的场景。

| 函数/装饰器 | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| partial()   | 冻结可调用对象的某些参数，因此该函数也被称为偏函数           |
| @lru_cache  | 为函数提供缓存功能，当某一函数的两次调用参数均一致，则直接返回前一次调用的结果 |

在该模块中，我们之前也已经接触过它所提供的redue()与@warps装饰器，所以这里不再进行举例。



# partial()

传入1个可调用对象和它的某一个或多个调用参数，返回1个新的可调用对象，并且该对象中的某些参数是被固定的。

函数签名如下：

```
functools.partial(func, *args, **keywords)
```

官方文档实现：

```
def partial(func, *args, **keywords):
    def newfunc(*fargs, **fkeywords):
        newkeywords = keywords.copy()
        newkeywords.update(fkeywords)
        return func(*args, *fargs, **newkeywords)
    newfunc.func = func
    newfunc.args = args
    newfunc.keywords = keywords
    return newfunc
```

示例演示：

```
import functools


def add(x, y):
    return x + y


# newFunc = add(1, 2)
newFunc = functools.partial(add, 1, 2)
print(newFunc())

# 3
```

再来一个2进制转10进制的函数：

```
import functools

binToDecimal = functools.partial(int, base=2)
print(binToDecimal("110"))

# 6
```





# @lru_cache

一个为函数提供缓存功能的装饰器，缓存maxsize组传入参数，在下次以相同参数调用时直接返回上一次的结果。用以节约高开销或I/O函数的调用时间。

由于使用了字典存储缓存，所以被装饰的函数固定参数和关键字参数必须是可哈希的。



函数签名如下：

```
@functools.lru_cache(maxsize=128, typed=False)
```

参数释义：

- 如果maxsize设置为None，LRU功能将被禁用且缓存数量无上限。maxsize设置为2的幂时可获得最佳性能。

- 如果typed设置为true，不同类型的函数参数将被分别缓存。例如，f(3)和f(3.0)将被视为不同而分别缓存。



一个简单的例子：

- 第一次运行函数，传入参数1和2，计算结果为3，缓存这2个参数和结果
- 第二次运行函数，传入参数1和2，查询缓存，缓存有就直接获得结果，根本不运行函数，所以没有看到print()的打印效果
- 第三次运行函数，传入参数1.0和2，查询缓存，由于typed为True，故严格区分浮点型和整形，再次运行函数，结果计算为3.0

如下示例：

```
import functools


@functools.lru_cache(maxsize=128, typed=True)
def add(x, y):
    print("add run...")
    return x+y


print(add(1, 2))
print(add(1, 2))
print(add(1.0, 2))

# add run...
# 3
# 3
# add run...
# 3.0
```

乍一看之下好像没什么作用，不就是缓存了一下嘛，实际上，在对递归函数上加上该装饰器，性能将会得到质的提升。

如下示例了加上该装饰器函数求解上楼梯问题和不加该装饰器函数求解上楼梯问题的总计运行时间。

对35阶梯楼梯的计算，加了该装饰器的运行几乎是瞬间完成，而不加该装饰器大概需要耗费十秒左右：

```
import functools
import time


@functools.lru_cache(maxsize=256, typed=False)
def haveCache(n):
    if n == 1:
        return 1
    if n == 2:
        return 2
    return haveCache(n - 1) + haveCache(n - 2)


def dontHaveCache(n):
    if n == 1:
        return 1
    if n == 2:
        return 2
    return dontHaveCache(n - 1) + dontHaveCache(n - 2)


s = time.time()
haveCache(35)
print("HaveCache - >", time.time() - s)

s = time.time()
dontHaveCache(35)
print("dontHaveCache - >", time.time() - s)

# HaveCache - > 0.0
# dontHaveCache - > 11.06638216972351
```

如果最大缓存设为2，则运行时间会慢一点，但是32阶的上楼梯问题也是在短短0.2秒之内得到解决了：

```
@functools.lru_cache(maxsize=2, typed=False)
...


# HaveCache - > 0.23421168327331543
```

