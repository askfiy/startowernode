# Python简述

Python诞生于1989年圣诞节，由[仁慈的独裁者](https://zh.wikipedia.org/wiki/%E7%BB%88%E8%BA%AB%E4%BB%81%E6%85%88%E7%8B%AC%E8%A3%81%E8%80%85)（Benevolent Dictator For Life，缩写BDFL）[吉多·范罗苏姆](https://zh.wikipedia.org/wiki/吉多·范罗苏姆)（Guido van Rossum）基于C语言开发。

Python的名字来源于龟叔（Guido van Rossum在Python界的爱称）十分喜欢的一部名为Monty Python's Flying的电视剧，一想到这个风靡全球的编程语言名字居然来的这么随意，不禁让人哑然失笑。

作为一门解释性的动态强类型语言，Python的开发效率奇高，因此在Python界流传着这样一句至理名言：

> 人生苦短，我用Python

![image-20210512172438655](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210512172438655.png)

Python并不是毫无缺点的孩子，作为一门动态语言，它的执行效率是偏低的。

作为80年代的产物，它的并发性支持也可能不太好，但是这并不妨碍我们对它的热爱，相信只要拥入Python的怀抱你一定会爱上这一门优雅的语言。





# Python解释器

由于是动态语言，Python在代码执行时必须先将代码转换为字节码，然后通过字节码再转换为机器可读的机器码。

而解释器就是负责这一切工作的小蜜蜂。

我们常说的Python是基于C语言开发而来的CPython，除此之外还有基于Java开发而来的Jython、以及基于C#开发而来的IronPython。

不论是Jython还是IronPython，我们在学习时都不会去考虑，而是统一选择CPython，因为它们相较于传统的CPython多了一个转换环节所以会导致执行效率的降低。



## 低效率的执行

动态语言是逐行翻译，我们可以将它理解为一种边跑边看的策略。

这样做的后果是方便代码排查，缺点是拉低执行效率。

常见的Python解释器（包括CPython）等都是采用这种策略，故Python的执行效率一直被人诟病。



## PyPy解释器

熟悉Python的朋友都知道，有一款Python解释器打破了人们对于Python低执行效率的印象。

它就是PyPy解释器，PyPy是另一个Python解释器，它的目标是执行速度。

PyPy采用[JIT技术](http://en.wikipedia.org/wiki/Just-in-time_compilation)，对Python代码进行动态编译（注意不是解释），所以可以显著提高Python代码的执行速度。

绝大部分Python代码都可以在PyPy下运行，但是PyPy和CPython有一些是不同的，这就导致相同的Python代码在两种解释器下执行可能会有不同的结果。

如果你的代码要放到PyPy下执行，就需要了解[PyPy和CPython的不同点](http://pypy.readthedocs.org/en/latest/cpython_differences.html)。

你可以简单的这么理解：

- 代码第一次运行时：进行动态编译，生成目标文件
- 代码第二次运行时：使用目标文件进行运行，没必要再进行逐行翻译，故执行效率提升

如果你需要较高的执行效率，可以选用该解释器，但是本专题中不会使用它。

因为PyPy终究不是正统，所以对很多第三方库的依赖性和兼容性不如CPython。





# 版本介绍

由于我们平常讲的Python实际上都为CPython，故我们接下来的学习也是围绕CPython（以下简称Python）展开的。

目前Python版本已经更迭到了3.9。

以下是Python的发展历程：

- 1989年，Guido开始写Python语言的解释器。

- 1991年，第一个Python解释器诞生，它是用C语言实现的，并能够调用C语言的库文件。



从一出生，Python已经具有了：类，函数，异常处理，包含表和词典在内的核心数据类型，以及模块为基础的拓展系统。

- Granddaddy of Python web frameworks, Zope 1 was released in 1999
- Python 1.0 - January 1994 增加了 lambda, map, filter and reduce.
- Python 2.0 - October 16, 2000，加入了内存回收机制，构成了现在Python语言框架的基础
- Python 2.4 - November 30, 2004, 同年目前最流行的WEB框架Django诞生
- Python 2.5 - September 19, 2006
- Python 2.6 - October 1, 2008
- Python 2.7 - July 3, 2010



In November 2014, it was announced that Python 2.7 would be supported until 2020, and reaffirmed that there would be no 2.8 release as users were expected to move to Python 3.4+ as soon as possible

Python 3.0 - December 3, 2008

- Python 3.1 - June 27, 2009
- Python 3.2 - February 20, 2011
- Python 3.3 - September 29, 2012
- Python 3.4 - March 16, 2014
- Python 3.5 - September 13, 2015
- Python 3.6 - 2016-12
- Python 3.7 - 2018
- Python 3.8 - 2019
- ...



细心的读者会发现，08年时就推出了3.0，2010年反而又推出了2.7？

这是因为3.0不向下兼容2.0，而很多公司已经基于2.0版本开发出了大量程序，公司已然投入了大量的人财物力，这就导致大家都拒绝升级3.0，无奈官方只能推出2.7过渡版本，之后我们都应该采用3.0解释器开发程序，但为了方便读者维护2.0版本的软件，在遇到两种版本的差异时会专门指出来。



# 应用方向

Python的应用领域十分广泛，如：人工智能，数据处理，爬虫，金融量化，云计算，WEB开发，自动化运维/测试，游戏开发，网络服务，图像处理等众多领域。

并且国内外很多知名的企业也都在使用Python，如：Youtube、DropBox、BT、Quora（中国知乎）、豆瓣、知乎、Google、Yahoo、FaceBook、NASA、百度、腾讯、汽车之家、美团等等。