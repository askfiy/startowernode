# 2种常用类型

在Python中，最常用的2种数据类型为list和dict。

你是否了解过它们的各种方法，时间复杂度到底如何？在那种策略下用那种方法更省时？

两种数据类型都拥有很多方法，常用的亦或是不常用的，因此在Python设计之初定下了一个原则：



> 让最常用的操作性能最好，牺牲不太常用的操作



在实际使用中，80%的功能其使用率往往只有20%，因此将剩下的20%的功能时间复杂度降低，而将不常用的80%功能时间复杂度增加，做到一种均衡的策略。

![image-20210418150011966](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210418150011966.png)

其实在Python官网上，已经贴出了每种数据类型方法的时间复杂度，[点我跳转](https://wiki.python.org/moin/TimeComplexity)



# dict

## 官方展示

The Average Case assumes parameters generated uniformly at random.

Internally, a list is represented as an array; the largest costs come from growing beyond the current allocation size (because everything must move), or from inserting or deleting somewhere near the beginning (because everything after that must move). If you need to add/remove at both ends, consider using a collections.deque instead.

| **Operation**                                                | **Average Case** | **[Amortized Worst Case](http://en.wikipedia.org/wiki/Amortized_analysis)** |
| ------------------------------------------------------------ | ---------------- | ------------------------------------------------------------ |
| Copy                                                         | O(n)             | O(n)                                                         |
| Append[1]                                                    | O(1)             | O(1)                                                         |
| Pop last                                                     | O(1)             | O(1)                                                         |
| Pop intermediate[2]                                          | O(n)             | O(n)                                                         |
| Insert                                                       | O(n)             | O(n)                                                         |
| Get Item                                                     | O(1)             | O(1)                                                         |
| Set Item                                                     | O(1)             | O(1)                                                         |
| Delete Item                                                  | O(n)             | O(n)                                                         |
| Iteration                                                    | O(n)             | O(n)                                                         |
| Get Slice                                                    | O(k)             | O(k)                                                         |
| Del Slice                                                    | O(n)             | O(n)                                                         |
| Set Slice                                                    | O(k+n)           | O(k+n)                                                       |
| Extend[1]                                                    | O(k)             | O(k)                                                         |
| [Sort](http://svn.python.org/projects/python/trunk/Objects/listsort.txt) | O(n log n)       | O(n log n)                                                   |
| Multiply                                                     | O(nk)            | O(nk)                                                        |
| x in s                                                       | O(n)             |                                                              |
| min(s), max(s)                                               | O(n)             |                                                              |
| Get Length                                                   | O(1)             | O(1)                                                         |



## pop()和inster()

pop()和inster()通常来说有2种情况：

- 如果都是操纵list[-1]，也就是最后一个数据项，它们的时间复杂度均为O(1)
- 如果操纵的是其他数据项，则时间复杂度均为O(n)

由于list底层是顺序存储，故任何一个非index-1的数据项的添加或删除都会引起整个列表的调整。

例如，从中部移除数据项的话，要把被移除数据项后面的全部数据项向前挪一个槽位。

虽然看起来有点笨拙，但这种实现方法能够保证列表按索引取值和赋值的操作很快，能够达到O(1)的良好情况。

这也算是一种对常用和不常用操作的折衷方案吧。

# dict

## 官方展示

The Average Case times listed for dict objects assume that the hash function for the objects is sufficiently robust to make collisions uncommon. The Average Case assumes the keys used in parameters are selected uniformly at random from the set of all keys.

Note that there is a fast-path for dicts that (in practice) only deal with str keys; this doesn't affect the algorithmic complexity, but it can significantly affect the constant factors: how quickly a typical program finishes.

| **Operation** | **Average Case** | **Amortized Worst Case** |
| ------------- | ---------------- | ------------------------ |
| k in d        | O(1)             | O(n)                     |
| Copy[3]       | O(n)             | O(n)                     |
| Get Item      | O(1)             | O(n)                     |
| Set Item[1]   | O(1)             | O(n)                     |
| Delete Item   | O(1)             | O(n)                     |
| Iteration[3]  | O(n)             | O(n)                     |



## 为什么字典这么快

dict内部采用hash存储，所以单点查找非常迅速，但不能使用范围查找。

这也是所有hash存储的特性，是属于一种典型的空间换时间的方案。