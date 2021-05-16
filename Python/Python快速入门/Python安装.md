# 本机示例

虽然目前的Python版本已经更迭到了3.9，但是在这里将会使用Python3.6.8来进行开发。

Python3.6系列应该是目前使用最为广泛的一个版本系列，相较于最新版的3.9来说它支持的库更多、兼容性更好。

需要注意的是在类Unix平台中，Python2版本已经自带，所以我们只需安装Python3.6即可。



# MAC安装

MAC下有源码安装Source，和界面化.pkg安装，我们选择pkg安装。

第一步：打开Python官网，[点我跳转](https://www.python.org/)：

![image-20210512174911639](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210512174911639.png)

第二步，下载对应版本的pkg安装程序：

![image-20210430153628767](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210430153628767.png)

第三步，一直点击下一步即可：

![image-20210430153730402](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210430153730402.png)



上述的安装方式会自动将Python安装至以下目录：

```
/Library/Frameworks/Python.framework/Versions/3.6/
```

安装完成之后系统会自动的将常用软件ln -s到/usr/local/bin中：

```
$ ls /usr/local/bin/ | grep p.*3
pip3
pip3.6
pydoc3
pydoc3.6
python3
python3-config
python3.6
python3.6-config
python3.6m
python3.6m-config
pyvenv-3.6
```



# Linux安装

Linux下有源码安装Source，和免编译安装Binary，我们选择免编译安装。

第一步，下载gcc工具：

```
$ yum install gcc gcc-c++ -y
```

第二步，下载wget工具：

```
$ yum install wget -y
```

第三步，下载Python3.6.8的Binary版本：

```
$ cd ~
$ wget https://www.python.org/ftp/python/3.6.8/Python-3.6.8.tgz
```

第五步，将归档文件tgz解压至当前目录：

```
$ > tar xvf Python-3.6.8.tgz -C ./
$ > ls
anaconda-ks.cfg  Python-3.6.8  Python-3.6.8.tgz
```

第六步，进入解压的目录中，进行编译安装（由于CPython基于C语言，而C语言又是编译性语言，故需要进行编译后安装）：

```
$ > cd Python-3.6.8
$ > ./configure --prefix=/usr/local/Python36
$ > make
$ > make install
```

第七步，添加环境变量：

```
$ vim /etc/profile

    # 写入内容
    export PATH=/usr/local/Python36/bin:$PATH

$ source /etc/profile
```



# 交互测试

安装完成之后，分别在shell中输入python（自带的版本）以及python3，查看是否能进入交互式环境中。

如果能成功进入则按照成功，如果不能请谷歌排查原因。

若想退出交互式环境，输入exit()即可：

```
$ python
Python 2.7.10 (default, Feb 22 2019, 21:55:15)
[GCC 4.2.1 Compatible Apple LLVM 10.0.1 (clang-1001.0.37.14)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()

$ python3
Python 3.6.8 (v3.6.8:3c6b436a57, Dec 24 2018, 02:04:31)
[GCC 4.2.1 Compatible Apple LLVM 6.0 (clang-600.0.57)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()
```

