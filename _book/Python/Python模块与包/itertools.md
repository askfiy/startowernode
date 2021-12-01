# itertools简介

itertools是Python内置模块，提供了大量为高效循环而创建的迭代器函数，当你有以下一些特殊需求时就可以使用它们，而不必再自己动手造轮子。

[官方文档](https://docs.python.org/zh-cn/3.6/library/itertools.html)

由于提供的迭代器众多，故不可能每个都记得，这里放上摘自官网的迭代器一览表。

首先是无穷迭代器如下表所示，即能够无限被迭代的迭代器：

| 迭代器                                                       | 实参          | 结果                                | 示例                                  |
| :----------------------------------------------------------- | :------------ | :---------------------------------- | :------------------------------------ |
| [count()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.count) | start, [step] | start, start+step, start+2*step, …  | count(10) --> 10 11 12 13 14 ...      |
| [cycle()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.cycle) | p             | p0, p1, … plast, p0, p1, …          | cycle('ABCD') --> A B C D A B C D ... |
| [repeat()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.repeat) | elem [,n]     | elem, elem, elem, … 重复无限次或n次 | repeat(10, 3) --> 10 10 10            |

其次是可停止的迭代器：

| 迭代器                                                       | 实参                        | 结果                                           | 示例                                                     |
| :----------------------------------------------------------- | :-------------------------- | :--------------------------------------------- | :------------------------------------------------------- |
| [accumulate()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.accumulate) | p [,func]                   | p0, p0+p1, p0+p1+p2, …                         | accumulate([1,2,3,4,5]) --> 1 3 6 10 15                  |
| [chain()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.chain) | p, q, …                     | p0, p1, … plast, q0, q1, …                     | chain('ABC', 'DEF') --> A B C D E F                      |
| [chain.from_iterable()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.chain.from_iterable) | iterable – 可迭代对象       | p0, p1, … plast, q0, q1, …                     | chain.from_iterable(['ABC', 'DEF']) --> A B C D E F      |
| [compress()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.compress) | data, selectors             | (d[0] if s[0]), (d[1] if s[1]), …              | compress('ABCDEF', [1,0,1,0,1,1]) --> A C E F            |
| [dropwhile()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.dropwhile) | pred, seq                   | seq[n], seq[n+1], … 从pred首次真值测试失败开始 | dropwhile(lambda x: x<5, [1,4,6,4,1]) --> 6 4 1          |
| [filterfalse()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.filterfalse) | pred, seq                   | seq中pred(x)为假值的元素，x是seq中的元素。     | filterfalse(lambda x: x%2, range(10)) --> 0 2 4 6 8      |
| [groupby()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.groupby) | iterable[, key]             | 根据key(v)值分组的迭代器                       |                                                          |
| [islice()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.islice) | seq, [start,] stop [, step] | seq[start:stop:step]中的元素                   | islice('ABCDEFG', 2, None) --> C D E F G                 |
| [starmap()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.starmap) | func, seq                   | func(*seq[0]), func(*seq[1]), …                | starmap(pow, [(2,5), (3,2), (10,3)]) --> 32 9 1000       |
| [takewhile()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.takewhile) | pred, seq                   | seq[0], seq[1], …, 直到pred真值测试失败        | takewhile(lambda x: x<5, [1,4,6,4,1]) --> 1 4            |
| [tee()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.tee) | it, n                       | it1, it2, … itn 将一个迭代器拆分为n个迭代器    |                                                          |
| [zip_longest()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.zip_longest) | p, q, …                     | (p[0], q[0]), (p[1], q[1]), …                  | zip_longest('ABCD', 'xy', fillvalue='-') --> Ax By C- D- |

然后是排列组合迭代器：

| 迭代器                                                       | 实参               | 结果                                            |
| :----------------------------------------------------------- | :----------------- | :---------------------------------------------- |
| [product()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.product) | p, q, … [repeat=1] | 笛卡尔积，相当于嵌套的for循环                   |
| [permutations()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.permutations) | p[, r]             | 长度r元组，所有可能的排列，无重复元素           |
| [combinations()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.combinations) | p, r               | 长度r元组，有序，无重复元素                     |
| [combinations_with_replacement()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.combinations_with_replacement) | p, r               | 长度r元组，有序，元素可重复                     |
| product('ABCD', repeat=2)                                    |                    | AA AB AC AD BA BB BC BD CA CB CC CD DA DB DC DD |
| permutations('ABCD', 2)                                      |                    | AB AC AD BA BC BD CA CB CD DA DB DC             |
| combinations('ABCD', 2)                                      |                    | AB AC AD BC BD CD                               |
| combinations_with_replacement('ABCD', 2)                     |                    | AA AB AC AD BB BC BD CC CD DD                   |

自认为比较重要的方法有：chain()，zip_longset()，permutations()。

除此之外的方法局限性太强，不适合所有场景。



# 无穷迭代器

## count()

创建1个迭代器，初始值是start，步长是step，结束是无限。

函数签名如下：

```
itertools.count(start=0, step=1)
```

官方文档的实现：

```
def count(start=0, step=1):
    # count(10) --> 10 11 12 13 14 ...
    # count(2.5, 0.5) -> 2.5 3.0 3.5 ...
    n = start
    while True:
        yield n
        n += step
```

注意，可以生成浮点数，这是range所不支持的。



## cycle()

创建1个迭代器，对传入的可迭代对象元素进行无限复制。

函数签名如下：

```
itertools.cycle(iterable)
```

官方文档的实现：

```
def cycle(iterable):
    # cycle('ABCD') --> A B C D A B C D A B C D ...
    saved = []
    for element in iterable:
        yield element
        saved.append(element)
    while saved:
        for element in saved:
              yield element
```



## repeat()

创建1个迭代器，对传入的可迭代对象进行无限复制，或通过times参数指定复制的次数：

函数签名如下：

```
itertools.repeat(object[, times])
```

官方文档的实现：

```
def repeat(object, times=None):
    # repeat(10, 3) --> 10 10 10
    if times is None:
        while True:
            yield object
    else:
        for i in range(times):
            yield object
```



# 可停止的迭代器

## accumulate()

该函数可接收2个参数，1个可迭代对象和1个具有2参数的可调用对象，返回1个迭代器。

该迭代器的值生成基于可调用对象对传入数据项的处理。

函数签名如下：

```
itertools.accumulate(iterable[, func])
```

官方文档实现：

```
def accumulate(iterable, func=operator.add):
    'Return running totals'
    # accumulate([1,2,3,4,5]) --> 1 3 6 10 15
    # accumulate([1,2,3,4,5], operator.mul) --> 1 2 6 24 120
    it = iter(iterable)
    try:
        total = next(it)
    except StopIteration:
        return
    yield total
    for element in it:
        total = func(total, element)
        yield total
```

示例演示：

```
import itertools

print(list(itertools.accumulate(range(5), lambda x,y:x+y)))

# [0, 1, 3, 6, 10]
```

过程解析：

```
# range(5) lambda x,y : x+y
# [0, 1, 2, 3, 4]
# 第一次：返回0  -> [0]
# 第二次：0 + 1 = 1 -> [0, 1]
# 第三次：1 + 2 = 3 -> [0, 1, 3]
# 第四次：3 + 3 = 6 -> [0, 1, 3, 6]
# 第五次：4 + 6 = 10 -> [0, 1, 3, 6, 10]
# 结果：
# [0, 1, 3, 6, 10]
```



## chain()

该函数可接收无限多的可迭代对象，并将它们进行合并成1个迭代器进行返回。

函数签名如下：

```
itertools.chain(*iterables)
```

官方文档实现：

```
def chain(*iterables):
    # chain('ABC', 'DEF') --> A B C D E F
    for it in iterables:
        for element in it:
            yield element
```



## from_iterable()

该函数可接收一个多维度的可迭代对象，并将多维展开合并成1个平面迭代器进行返回。

函数签名如下：

```
itertools.chain.from_iterable(iterable)
```

官方文档实现：

```
def from_iterable(iterables):
    # chain.from_iterable(['ABC', 'DEF']) --> A B C D E F
    for it in iterables:
        for element in it:
            yield element
```



## compress

创建一个迭代器，它返回data中经selectors真值测试为True的元素。迭代器在两者较短的长度处停止。

函数签名如下：

```
itertools.compress(data, selectors)
```

官方文档实现：

```
def compress(data, selectors):
    # compress('ABCDEF', [1,0,1,0,1,1]) --> A C E F
    return (d for d, s in zip(data, selectors) if s)
```



## dropwhile()

创建一个迭代器，如果predicate为true，迭代器丢弃这些元素，然后返回其他元素。

注意，迭代器在predicate首次为false之前不会产生任何输出，所以可能需要一定长度的启动时间。

函数签名如下：

```
itertools.dropwhile(predicate, iterable)
```

官方文档实现：

```
def dropwhile(predicate, iterable):
    # dropwhile(lambda x: x<5, [1,4,6,4,1]) --> 6 4 1
    iterable = iter(iterable)
    for x in iterable:
        if not predicate(x):
            yield x
            break
    for x in iterable:
        yield x
```



## filterfalse()

创建一个迭代器，只返回iterable中predicate为False 的元素。如果predicate是None，返回真值测试为false的元素。

其实说白了就相当于filter()的反函数。

函数签名如下：

```
itertools.filterfalse(predicate, iterable)
```

官方文档实现：

```
def filterfalse(predicate, iterable):
    # filterfalse(lambda x: x%2, range(10)) --> 0 2 4 6 8
    if predicate is None:
        predicate = bool
    for x in iterable:
        if not predicate(x):
            yield x
```



## islice()

创建一个迭代器，该函数接收3个参数，总体效果和切片取子序列类似，能指定开始位置、结束位置、步长等。

函数签名如下：

```
itertools.islice(iterable, start, stop[, step]
```

官方文档实现：

```
def islice(iterable, *args):
    # islice('ABCDEFG', 2) --> A B
    # islice('ABCDEFG', 2, 4) --> C D
    # islice('ABCDEFG', 2, None) --> C D E F G
    # islice('ABCDEFG', 0, None, 2) --> A C E G
    s = slice(*args)
    start, stop, step = s.start or 0, s.stop or sys.maxsize, s.step or 1
    it = iter(range(start, stop, step))
    try:
        nexti = next(it)
    except StopIteration:
        # Consume *iterable* up to the *start* position.
        for i, element in zip(range(start), iterable):
            pass
        return
    try:
        for i, element in enumerate(iterable):
            if i == nexti:
                yield element
                nexti = next(it)
    except StopIteration:
        # Consume to *stop*.
        for i, element in zip(range(i + 1, stop), iterable):
            pass
```



## starmap()

创建一个迭代器，使用从可迭代对象中获取的参数来计算该函数。当参数对应的形参已从一个单独可迭代对象组合为元组时（数据已被“预组对”）可用此函数代替 [map()](https://docs.python.org/zh-cn/3.6/library/functions.html#map)。[map()](https://docs.python.org/zh-cn/3.6/library/functions.html#map)与[starmap()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.starmap)之间的区别可以类比 function(a,b)与function(*c)的区别。

函数签名如下：

```
itertools.starmap(function, iterable)
```

官方文档实现：

```
def starmap(function, iterable):
    # starmap(pow, [(2,5), (3,2), (10,3)]) --> 32 9 1000
    for args in iterable:
        yield function(*args)
```

注意，function必须接收2个参数，而传入的Iterable也必须符合上述格式。



## takewhile()

相当于filter()，创建一个迭代器，只要predicate为真就从可迭代对象中返回元素。

函数签名如下：

```
itertools.takewhile(predicate, iterable)
```

官方文档实现：

```
def takewhile(predicate, iterable):
    # takewhile(lambda x: x<5, [1,4,6,4,1]) --> 1 4
    for x in iterable:
        if predicate(x):
            yield x
        else:
            break
```



## zip_longset()

创建一个迭代器，从每个可迭代对象中收集元素。如果可迭代对象的长度未对齐，将根据fillvalue填充缺失值。迭代持续到耗光最长的可迭代对象。

函数签名如下：

```
itertools.zip_longest(*iterables, fillvalue=None)
```

官方文档实现：

```
class ZipExhausted(Exception):
    pass

def zip_longest(*args, **kwds):
    # zip_longest('ABCD', 'xy', fillvalue='-') --> Ax By C- D-
    fillvalue = kwds.get('fillvalue')
    counter = len(args) - 1
    def sentinel():
        nonlocal counter
        if not counter:
            raise ZipExhausted
        counter -= 1
        yield fillvalue
    fillers = repeat(fillvalue)
    iterators = [chain(it, sentinel(), fillers) for it in args]
    try:
        while iterators:
            yield tuple(map(next, iterators))
    except ZipExhausted:
        pass
```



## 其他的迭代器

这里省略了2个迭代器，[tee()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.tee)和[groupby()](https://docs.python.org/zh-cn/3.6/library/itertools.html#itertools.groupby)，因为我实在想不通在怎样的场景下会去使用它们..





# 排序组合迭代器

## product()

返回可迭代对象输入的笛卡儿积，存在重复元素。

函数签名如下：

```
itertools.product(*iterables, repeat=1)
```

官方文档实现：

```
def product(*args, repeat=1):
    # product('ABCD', 'xy') --> Ax Ay Bx By Cx Cy Dx Dy
    # product(range(2), repeat=3) --> 000 001 010 011 100 101 110 111
    pools = [tuple(pool) for pool in args] * repeat
    result = [[]]
    for pool in pools:
        result = [x+[y] for x in result for y in pool]
    for prod in result:
        yield tuple(prod)
```



## permutations()

连续返回由iterable元素生成长度为r的排列。

如果r未指定或为None ，r默认设置为iterable的长度，这种情况下，生成所有全长排列。

排列依字典序发出。因此，如果iterable是已排序的，排列元组将有序地产出。

即使元素的值相同，不同位置的元素也被认为是不同的。如果元素值都不同，每个排列中的元素值不会重复。

函数签名如下：

```
itertools.permutations(iterable, r=None
```

官方文档实现：

```
def permutations(iterable, r=None):
    # permutations('ABCD', 2) --> AB AC AD BA BC BD CA CB CD DA DB DC
    # permutations(range(3)) --> 012 021 102 120 201 210
    pool = tuple(iterable)
    n = len(pool)
    r = n if r is None else r
    if r > n:
        return
    indices = list(range(n))
    cycles = list(range(n, n-r, -1))
    yield tuple(pool[i] for i in indices[:r])
    while n:
        for i in reversed(range(r)):
            cycles[i] -= 1
            if cycles[i] == 0:
                indices[i:] = indices[i+1:] + indices[i:i+1]
                cycles[i] = n - i
            else:
                j = cycles[i]
                indices[i], indices[-j] = indices[-j], indices[i]
                yield tuple(pool[i] for i in indices[:r])
                break
        else:
            return
```





## combinations()

返回由输入iterable中元素组成长度为 *r* 的子序列。

组合按照字典序返回。所以如果输入iterable是有序的，生成的组合元组也是有序的。

即使元素的值相同，不同位置的元素也被认为是不同的。如果元素各自不同，那么每个组合中没有重复元素。

函数签名如下：

```
itertools.combinations(iterable, r)
```

官方文档实现：

```
def combinations(iterable, r):
    # combinations('ABCD', 2) --> AB AC AD BC BD CD
    # combinations(range(4), 3) --> 012 013 023 123
    pool = tuple(iterable)
    n = len(pool)
    if r > n:
        return
    indices = list(range(r))
    yield tuple(pool[i] for i in indices)
    while True:
        for i in reversed(range(r)):
            if indices[i] != i + n - r:
                break
        else:
            return
        indices[i] += 1
        for j in range(i+1, r):
            indices[j] = indices[j-1] + 1
        yield tuple(pool[i] for i in indices)
```



## combinations_with_replacement()

返回由输入iterable中元素组成的长度为r的子序列，允许每个元素可重复出现。

组合按照字典序返回。所以如果输入iterable是有序的，生成的组合元组也是有序的。

不同位置的元素是不同的，即使它们的值相同。因此如果输入中的元素都是不同的话，返回的组合中元素也都会不同。

函数签名如下：

```
itertools.combinations_with_replacement(iterable, r)
```

官方文档实现：

```
def combinations_with_replacement(iterable, r):
    # combinations_with_replacement('ABC', 2) --> AA AB AC BB BC CC
    pool = tuple(iterable)
    n = len(pool)
    if not n and r:
        return
    indices = [0] * r
    yield tuple(pool[i] for i in indices)
    while True:
        for i in reversed(range(r)):
            if indices[i] != n - 1:
                break
        else:
            return
        indices[i:] = [indices[i] + 1] * (r - i)
        yield tuple(pool[i] for i in indices)
```



