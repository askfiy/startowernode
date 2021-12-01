# pip工具

pip是Python的包管理工具，该工具提供了对Python第三方包的查找、下载、安装、卸载等功能。

Python 2.7.9+ 或 Python 3.4+ 以上版本都自带pip工具，在安装Python时会一并安装。

如果你安装了两个版本的Python，则对应的pip命令就是pip2或者pip3。

```
$ ls /usr/local/bin/pip*
/usr/local/bin/pip    /usr/local/bin/pip2   /usr/local/bin/pip2.7 /usr/local/bin/pip3   /usr/local/bin/pip3.6
```

Ps：类似于NodeJS的NPM



# 常用命令

在shell中可使用的pip常用命令如下：

| 命令                       | 描述               |
| -------------------------- | ------------------ |
| pip --version              | 显示版本和路径     |
| pip --help                 | 获取帮助           |
| pip install -U pip         | pip升级            |
| pip install 包名           | 安装包             |
| pip install --upgrade 包名 | 升级包             |
| pip uninstall 包名         | 卸载包             |
| pip search 包名            | 搜索包             |
| pip list                   | 查看所有已安装的包 |
| pip list -o                | 查看所有可升级的包 |





# pip升级

使用以下命令进行pip工具进行升级：

Windows平台：

```
$ python -m pip install -U pip     # python2.x
$ python -m pip3 install -U pip    # python3.x
```

Linux&Mac平台：

```
$ pip install --upgrade pip    # python2.x
$ pip3 install --upgrade pip   # python3.x
```



# 换源配置

pip下载的包默认是从国外源下载，速度较慢，因此可以为其设置为国内源（阿里云）。

下面介绍类Unix平台与Windows平台的换源方式。



## Windows

直接在user目录中创建一个pip目录，如：C:\Users\username\pip，新建文件pip.ini，然后填入以下内容：

```
[global]
index-url = https://mirrors.aliyun.com/pypi/simple
```



## Unix

类Unix平台需要依次运行如下命令：

```
$ cd ~                            # 进入用户家目录
$ mkdir .pip                      # 创建隐藏文件夹
$ vim .pip/pip.conf               # 创建pip3的配置文件
```

然后在配置文件pip.conf中填入以下内容：

```
[global]
index-url = https://mirrors.aliyun.com/pypi/simple
```

