# hashlib简介

密码学是一个庞大的领域，总体来说可将该领域中的加密方式分为2大类：

- 对称加密：可通过加密值反解出内容
- 非对称加密：不可通过加密值反解出内容

而今天介绍的hashlib模块是Python3中所独有的，提供了一系列的非对称加密算法：hash算法。

在Python2中hashlib模块被拆分成了md5模块和sha模块，它们提供的功能和Python3的hashlib模块相同。

[官方文档](https://docs.python.org/zh-cn/3.6/library/hashlib.html)

以下是该模块提供的部分常用方法及属性：

| 属性/方法                     | 描述                                                     |
| ----------------------------- | -------------------------------------------------------- |
| hashlib.algorithms_guaranteed | 以集合方式，列出所有平台所支持的hash算法                 |
| hashlib.algorithms_available  | 以集合方式，列出当前所运行的Python解释器所支持的hash算法 |
| hash.digest_size              | 以字节表示结果hash对象的大小                             |
| hash.block_size               | 以字节表示的hash算法的内部块大小                         |
| hash.name                     | 返回hash对象的规范名称                                   |
| hash.copy()                   | 返回hash对象的拷贝副本                                   |
| hash.update()                 | 在已有基础上对hash对象的内容进行更新                     |
| hash.hexdigest()              | 返回16进制的字符串hash值                                 |
| hash.digest()                 | 返回2进制的字节串hash值                                  |



# hash特性

Python的字典在键值对数据存储和读取时，就用到了hash算法。

比如："k1" : "v1"的键值对在存储过程中，"k1"会通过hash()函数得出1个hash值，该hash值与v1一一对应，后续通过dict.get()方法通过"k1"找"v1"时，内部也是利用的这个hash值来进行查找。

通过字典的种种特性，我们可以顺势推导出hash的一些特性：

- 相同的内容求hash值，得到的hash结果也必然相同
- 不能通过hash值反解出内容（或者说反解的代价大到不可能实现，但也不是绝对的）
- 如果采用相同的hash算法，无论需要校验的内容由多大，得到的hash值长度总是固定的

我们使用内置的hash()函数来验证这3点结论：

1）相同的内容求hash值，得到的hash结果也必然相同：

```
>>> hash("hello world")
-484803057
>>> hash("HELLO WORLD")
264022494
>>> hash("hello world")
-484803057
```

2）不能通过hash值反解出内容：

```
>>> hash("k1")
-714364401
>>> hash("-714364401")
1936952577
```

3）如果采用相同的hash算法，无论需要校验的内容有多大，得到的hash值长度总是固定的：

```
>>> hash("hello")
313408759
>>> hash("hello, Python3")
-1705693388
```



# 算法区别

由于hash算法的特性，它常被用于一致性校验、密码存储等领域。

其中最著名的hash算法就是MD5，它被称为永不可破的hash算法，但随着技术的发展MD5已经不那么可靠了，它可以用撞库的方式对其进行反解。

而SHA256作为MD5的加强版，是目前的主流方案。

关于MD5和SHA家族的区别在于使用的加密算法不一样，以及它们生成的hash值长度不同：

- MD5较SHA家族的hash值要短一些，因此生成速度更快一点
- 对暴力破解来说，SHA家族的hash值比MD5的hash值更安全，更值得信赖
- MD5：128位
- SHA1：160位
- SHA256：256位

如果你的项目安全等级较高，可采用SHA256作为加密方式，其他情况下使用MD5即可。



# 模块使用

hashlib模块的使用非常简单，总体来说先要生成一个hash对象，然后再填入字节串即可。

首先是普通的使用，以MD5举例：

```
>>> import hashlib
>>> m = hashlib.md5("hello world".encode("u8"))
>>> m.digest()
b'^\xb6;\xbb\xe0\x1e\xee\xd0\x93\xcb"\xbb\x8fZ\xcd\xc3'
```

如果对一个大字符串生成hash值，可使用update()方法在原有hash对象基础上进行内容更新：

```
>>> m = hashlib.md5()
>>> m.update("line1".encode("u8"))
>>> m.update("line2".encode("u8"))
>>> m.update("line3".encode("u8"))
>>> m.digest()
b'\xcc\x0c\x81\xcd<\xfa)\x8e:\x06\x9c\xcal\x91\x9e\xdb
```

sha256的加密方式与md5的加密使用相同，如下所示：

```
>>> m = hashlib.sha256("hello world".encode("u8"))
>>> m.digest()
b"\xb9M'\xb9\x93M>\x08\xa5.R\xd7\xda}\xab\xfa\xc4\x84\xef\xe3zS\x80\xee\x90\x88\xf7\xac\xe2\xef\xcd\xe9"
```

基本使用就介绍完毕了，是不是非常简单呢？



# 撞库介绍

在密码破解领域，有一个百试不爽的方法就是撞库破解。

撞库是指通过一个庞大的数据库来记录未加密字符串与加密后的值的一种映射关系，理论上来说只要这个数据库无限大，那么生成的hash值都能在这里找到其对应的生成字符串。

举个例子：

我现在有1个字符串，I LOVE YOU。

对他进行hash加密得到的结果假设为3242。

现在将这个对应关系放到数据库中，及3242这个hash值对应的字符串为I LOVE YOU。

如果有人要对3242进行反解，通过查询数据库即可知道结果。

这个思路非常的简单粗暴，但个人是不可能进行数据库的完善和搭建。

在Google上如果搜索MD5反解，应该能找到一些撞库网站，但大多数都是付费的，如果感兴趣可以试一试。



# 加盐验证

为了防止你的加密内容被撞库反解，我们可以使用加盐的策略来对已加密的内容进行二次加密。

整体思路如下，我们以一个普通的用户登陆作为案例：

- Server端有一个固定的字符串，被称之为盐
- 用户第一次注册后，要将用户名和密码写入到数据库中，此时数据库中的密码应当密文存储，且不可被反解，做到仅有用户知道自己的密码，连开发人员都不得而知的状态是最完美的
- 存储密码的时候，对明文密码进行hash加密，并在其中掺盐，得到密文hash密码进行存储
- 用户登陆的时候，将用户登陆时发送的明文密码也进行hash加密和掺盐，并且通过登陆的用户名获取到存储在数据库中的密文hash密码，两者进行比对，若一致则登陆成功，若不一致则登陆失败
- 当数据库被黑客攻破后，只要保证盐不泄露，那么他就没有任何办法破解出用户的密码

理论很复杂，实操很简单。如下所示：

```
>>> salt = "slat".encode("u8")
>>> userPwd = "123456".encode("u8")
>>> hashObject = hashlib.md5(salt)  # ❶
>>> hashObject.update(userPwd)  # ❷
>>> savePwd = hashObject.digest()  # ❶ 
>>> savePwd
b'ELr\x05\x14$z=\x1d\x19(^4L>n'
>>>
>>> 
>>> reLoginPwd = "123456".encode("u8") 
>>> hashObject = hashlib.md5(salt)  # ❶
>>> hashObject.update(reLoginPwd)　 # ❷
>>> getPwd = hashObject.digest()  # ❸
>>> getPwd == savePwd  # ❹
True
```

❶：加盐

❷：加入用户内容

❸：获得存储密码

❹：对比用户重登陆的密码hash值是否和以存储的密码hash值一致



# 文件校验

在Server端对Client端发送文件的过程中，该文件可能被黑客截取做出一些篡改，如下所示：

```
server端    --------->     client端
                |
                |
        可能被黑客窃取，修改下载文件
```

此时就需要使用文件校验来确保安全性了：

- 在发送文件的时候要让用户知道我们文件本身的hash校验值
- 用户下载完成后将得出的结果与我们的hash校验值做对比
- 如果一致则文件没有被篡改过
- 如果不一致则文件已被篡改过

我们有2种方式，来进行文件校验的实现。

下面将采用模拟Server端生成文件校验hash值的整个过程。

首先是方式1，将文件所有内容hash校验一遍，安全系数最高，速度最慢。

```
res = ""m =  hashlib.sha256()f = open(file="test.txt",mode="rb")while 1:    temp = f.read(1024)    # ❶    m.update(temp) # ❷    if not len(temp):        f.close()        hash_res = m.hexdigest() # ❸        breakprint(hash_res) # 48dd13d8629b4a15f791dec773cab271895187a11683a3d19d4877a8c256cb70
```

❶：更新hash值

❷：由于打开文件的模式是rb，故temp本身就是bytes类型，所以不用encode()

❸：当所有内容读取完毕后，生成文件的校验hash值



其次是方式2，文件指定指针点来更新hash值，安全系数小幅度降低，但速度大幅度提升。

迅雷等下载软件均采用此种方式，前提是要让用户知道我们seek()的文件指针点在哪里:

```
m =  hashlib.sha256()f = open(file="1.txt",mode="rb")# ❶f.seek(20,0)temp = f.read(10)m.update(temp)# ❷f.seek(20,1)temp = f.read(10)m.update(temp)# ❸f.seek(-20,2)temp = f.read(10)m.update(temp)# ❹hash_res = m.hexdigest()print(hash_res) # daffa21b2be95802d2beeb1f66ce5feb61195e31074120a605421563f775e360
```

❶：在文件的开始位置，读取10个bytes，用作生成hash值的源内容部分

❷：在文件的中间位置，读取10个bytes，用作生成hash值的源内容部分

❸：在文件的末尾位置，读取10个bytes，用作生成hash值的源内容部分

❹：生成文件校验的hash值，该hash值共由30个bytes组成，分别来自文件的开始、中间、末尾位置。Ps：指针点越多，安全性越高，但是速度越慢



# hmac模块

hmac模块的使用与hashlib大同小异。但是在某些方面会比hashlib更优秀：

它也是一个内置模块，以下是简单的使用：

```
>>> import hmac>>> hmacObject = hmac.new("hello world".encode("u8"), digestmod="md5")>>> hmacObject.update("salt".encode("u8"))>>> hashValue = hmacObject.digest()>>> hashValueb'\xf3Q\xff\xb2V{\x88\xfe\x0e\x9aX\x19\xbf\x12\xf3<'>>> 
```



