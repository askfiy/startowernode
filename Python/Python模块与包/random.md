# random简介

random模块是Python自带的模块，该模块实现了各种分布的伪随机数生成器。

[官方文档](https://docs.python.org/zh-cn/3.6/library/random.html)

以下举例部分常用方法：

| 方法                   | 描述                                                   |
| ---------------------- | ------------------------------------------------------ |
| random.randint(1, 3)   | 从1-3之间随机生成一个整数                              |
| random.randrange(1, 3) | 从1-2之间随机生成一个整数                              |
| random.random()        | 生成大于0且小于1的浮点数                               |
| random.uniform(1, 3)   | 生成大于1且小于3的浮点数                               |
| random.choice(seq)     | 从序列中随机取出1个数据项                              |
| random.sample(seq, 2)  | 从序列中随机取出指定个数据项，这里是2，以列表形式返回  |
| random.shuffle(seq)    | 将传入的拥有索引的序列进行打乱，原地打乱，不返回新序列 |





# 整数生成

random.randint()是顾头顾尾的生成随机整数：

```
>>> import random
>>> random.randint(1, 3)
1
>>> random.randint(1, 3)
2
>>> random.randint(1, 3)
3
```

random.randrange()是顾头不顾尾的生成随机整数：

```
>>> random.randrange(1, 3)
1
>>> random.randrange(1, 3)
2
```



# 浮点数生成

random.random()生成浮点数的范围总是介于0和1之间：

```
>>> random.random()
0.818462343335113
```

random.uniform()可指定生成浮点数的范围：

```
>>> random.uniform(1, 3)
2.810443694822667
```



# 数据项抽取

random.choice()可以从一个序列中随机抽取出一个数据项：

```
>>> random.choice(range(10))
1
```

random.sample()可以从一个序列中随机抽取出多个数据项：

```
>>> random.sample(range(10), 2)
[1, 0]
```

它们貌似均不支持字典的随机抽取。

# 生成乱序列

random.shuffle()可以传入一个线性结构的序列，并将其中的数据项进行随机的打乱：

```
>>> li1 = [i for i in range(10)]
>>> random.shuffle(li1)
>>> li1
[1, 5, 6, 3, 7, 2, 9, 8, 4, 0]
>>> random.shuffle(li1)
>>> li1
[1, 5, 4, 3, 7, 0, 9, 6, 8, 2
```





# 生成验证码

验证码总是随机的，因此可以通过random模块进行实现生成：

```
import random


def getVerificationCode(bitNumber):
    code = ""
    for i in range(bitNumber):
        code += random.choice(
            [
                str(random.randint(1, 9)),
                chr(random.randint(65, 91))  # ❶
            ]
        )
    return code


code = getVerificationCode(6)
print(code)
```

❶：65-90是大写的A-Z的ASCII码表示范围