# None

None是Python中经常出现的一种类型，但其实关于它的描述并不是很多，很多人也会忽略。

其实None里也有一些需要了解的知识点。

它常用于初始化数据，并且是函数默认的返回值。

None是一种不可变类型，同时也是基础的原子类型，即不可分割，不能容纳其他对象。

# 基本声明

Node的声明方式仅有字面量声明：

```
empty = None
print("值:%r,类型：%r" % (empty, type(empty)))

# 值:None,类型：<class 'NoneType'>
```



# NoneType与单例

尽管可以看到None的实例化类为NoneType，但是你可能无法直接找到NoneType：

```
print(NoneType)

# NameError: name 'NoneType' is not defined
```

NoneType实现了单例模式，我们虽然无法直接拿到NoneType这个类，但是可以通过 \_\_class\_\_属性拿到。

以下实例化多个None，查看id()是否相同：

```
NoneType = None.__class__
none1 = NoneType()
none2 = NoneType()

print(id(none1))
print(id(none2))

# 4377856088
# 4377856088
```



# 绝对引用

None类型也是绝对引用，无论是深拷贝还是浅拷贝，都不会获得其副本，而是直接对源对象进行引用：

```
>>> import copy
>>> oldNone = None
>>> id(oldNone)
4369221720
>>> no1 = copy.copy(oldNone)
>>> id(no1)
4369221720
>>> no2 = copy.deepcopy(oldNone)
>>> id(no2)
4369221720
```



# None的使用

None一般用于对一个变量进行初始化，可能我们还没想好这个变量存什么内容时可以用None先代替进行存入。

这种变量可称之为临时变量，即只在一定的场景下进行使用，而并不会常驻使用：

```
temp = None
```

除此之外，None也常见于function的返回值中，并且Python中function默认的返回值就是None。