# 运行方式

## REPL交互式

REPL名为交互式解释器（Read Eval Print Loop），提供了一个CLI(command-line interface:命令行界面)下读取值、求值、输出值、循环代码的环境。

这种交互式的方式会经常用到，适用于快速的进行一些功能测试，如函数传参、求值测试等。

现在让我们以REPL交互式来运行第一条Python代码，在shell中进行执行：

```
$ python3
>>> print("hello world")
hello world
>>>
```

如果要退出REPL环境，则使用exit()即可。



## 脚本调用式

### 外部调用

在一个文件中，书写好Python代码后进行调用的方式被称为脚本调用式。

也是非常常用的一种方式，通常文件后缀名以py结尾，标识这是一个Python脚本文件。



> 使用Python调用脚本时，其实并不关心文件后缀名是什么，后缀名更多的作用是给使用者看的，便于区分不同类型的文件



```
$ echo "print('hello world')" > helloWorld.py
$ python3 helloWorld.py
hello world
```

在此示例中我们使用Python3的解释器在外部对其进行调用，并执行了其中的代码打印了hello world。



### 内部调用

除开外部调用，我们也可以在脚本内部指定Python解释器，并通过./进行代码执行，对上述文件内容进行修改：

```
#!/usr/bin/env python3

print('hello world')
```

修改完成并保存，再修改文件的执行权限：

```
$ chmod 775 helloWorld.py
```

直接使用./进行代码执行：

```
$ ./helloWorld.py
hello world
```

使用内部调用需要注意：指定Python解释器的语句一定要放在文件头部，因此该代码也被称为头文件代码。

常见的头文件代码除开指定Python解释器以外，还有指定解释器解码格式、声明作者和日期等，如下所示：

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
# author: YunYa
# date: 2017-01-28
```

头文件代码调用说明：env是类Unix平台的环境变量别称，当我们在头文件代码中指定/usr/bin/env python3的时候，它内部会运行env | grep python3，找到python3解释器，并对文件进行执行，前提是该文件必须具有可执行权限。

```
$ env | grep python3
VIRTUALENVWRAPPER_PYTHON=/Library/Frameworks/Python.framework/Versions/3.6/bin/python3.6
```





## 执行过程

一个Python程序被解释器解释并执行，可粗略的分为三个步骤：



> 1. 启动Python解释器
> 2. Python解释器发起系统调用，将脚本内容载入内存，此时并不会做任何处理
> 3. Python解释器开始识别Python语法，解释并执行内存中存储的脚本文件内容



# IDE介绍

## IDE简介

IDE的全称是Integrated Development Environment，即集成开发环境。

它是一种辅助程序开发人员开发[软件](https://zh.wikipedia.org/wiki/軟體)的[应用软件](https://zh.wikipedia.org/wiki/應用軟體)，在开发工具内部就可以辅助编写源代码文本、并编译打包成为可用的程序，有些甚至可以设计图形接口。

如果单纯的touch出一个文本，并在其内部书写代码是很容易出现问题的。

1. 没有语法高亮，这意味着某个字符少打了、某个单词拼错了都不能被及时发现，并且这样学习对新手难度极大
2. 没有debug功能，当业务逻辑越来越复杂时，程序不可避免的会出现一些bug，此时如果没有debug程序来检测出现问题的原因，整个排查的过程将变的异常艰难





而使用IDE开发则可以完全避免这些问题。



## Python IDLE

Python自带了一款类似的工具，名为Python IDLE，是Python的集成开发和学习环境。

官方文档：https://docs.python.org/3/library/idle.html



<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210430182235461.png" alt="image-20210430182235461" style="zoom:50%;" />



它自带了2种模式，一种是REPL交互模式，一种是脚本调用模式。

当你打开它后默认会进入REPL交互模式，此时只需要新建一个文件即可开始编写我们的代码。

- CTRL+N：新建文件
- CTRL+S：保存文件
- F5：运行程序



可以发现，当我们想使用print()功能时，它会提示该功能需要哪些参数：

![image-20210501233752626](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210501233752626.png)



当编写完代码后，即可使用F5来进行文件的保存与运行，如下所示：

![image-20210501233856456](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210501233856456.png)



IDLE适用于刚入门起步学习Python的同学，但对于大的项目构建来说还是十分的不方便。

推荐指数：⭐️⭐️⭐️



## PyCharm

PyCharm是由JetBrains公司开发，也是最受欢迎的Python IDE工具。

官方网站：https://www.jetbrains.com/pycharm/

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210430183130314.png" alt="image-20210430183130314" style="zoom:50%;" />

优点是功能强大，你能想到的不能想到的它都给你提供了。

缺点是收费，并且软件本身比较臃肿，但是比起它的优秀来说这些缺点可以忽略不计。

推荐指数：⭐️⭐️⭐️⭐️⭐️





## VsCode

VsCode是由Microsoft Corporation开发，是近年来最火的一款轻量级编辑器。

官方网站：https://code.visualstudio.com/

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210430183600647.png" alt="image-20210430183600647" style="zoom:67%;" />

优点是支持多语言，Golang、C、Python等手到擒来，此外它是完全免费的。

缺点是配置比较繁琐，对新手不太友好。

推荐指数：⭐️⭐️⭐️⭐️





# 个人使用

我自己平常写一些小的脚本，会首选VsCode，若是要搭建一些比较大型的项目，则会选择PyCharm。

在这里并没有详细指出每种工具的安装，因为互联网上类似的教程太多了。

推荐在初次接触Python时，建议统一使用Python IDLE，因为PyCharm本身会对Python许多地方做出优化，而恰好这些优化对新手并不友好。