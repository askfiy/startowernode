# 虚拟环境

虚拟环境是真实的Python解释器的一份拷贝。

由于真实的Python解释器可能在不断的添加各种第三方库，而我们的项目中又没有用到这些库所以就会造成一个环境的污染，可能会造成打包exe文件后体积过大等问题。

一个项目的环境干净是十分重要的，而虚拟环境就是为了净化项目环境而生的一种措施。

我们在这里使用virtualenv与virtualenvwrapper这两个第三方模块来更加方便的管理我们的虚拟环境。



# Windows

需要使用两个第三方模块，在终端中使用pip3命令下载安装：

```
$ pip3 install virtualenv
$ pip3 install virtualenvwrapper-win
```

创建虚拟环境工作目录，新建一个文件夹，从此之后所有的虚拟环境都会存放至该文件夹下：

```
$ D: mkdir .virtualenvs
```

然后需要配置虚拟环境工作目录：

1. 打开环境变量，在用户变量中新建，变量名为WORKON_HOME，值为虚拟环境的配置路径，即.virtualenvs的路径
2. 打开原本的Python3环境安装目录，找到Scripts文件夹，双击执行其下的virtualenvwrapper.bat



# Linux&Mac

需要使用两个第三方模块，在shell中使用pip3命令下载安装：

```
$ pip3 install -i https://pypi.douban.com/simple virtualenv
$ pip3 install -i https://pypi.douban.com/simple virtualenvwrapper
```

创建虚拟环境工作目录，新建一个文件夹，从此之后所有的虚拟环境都会存放至该文件夹下：

```
$ mkdir ~/.virtualenvs
```

配置环境变量，编辑userHome/.bash_profile文件，如果是Mac用户且终端为zsh，则配置userHome/.zshrc文件，新增以下内容：

```
# Setting virtual environment save path 
# 填入你的虚拟环境存放目录
export WORKON_HOME="~/.virtualenvs" 

# Setting virtual environment copy python path
# 填入你的真实Python3解释器路径，用于虚拟环境的拷贝
export VIRTUALENVWRAPPER_PYTHON=/Library/Frameworks/Python.framework/Versions/3.6/bin/python3.6

# Setting virtualenvwrapper.sh run path
# 填入virtualenvwrapper.sh的脚本路径
source /Library/Frameworks/Python.framework/Versions/3.6/bin/virtualenvwrapper.sh                                                               
```

最后刷新一下该文件的配置：

```
$ source ~/.bash_profile
```



# 命令概览

shell中可指定的命令如下：

| 命令                                 | 描述               |
| ------------------------------------ | ------------------ |
| workon                               | 列出所有虚拟环境   |
| mkvirtualenv -p python3 虚拟环境名字 | 创建新的虚拟环境   |
| rmvirtualenv 虚拟环境名字            | 删除指定的虚拟环境 |
| workon 虚拟环境名字                  | 使用指定的虚拟环境 |

示例演示，创建一个LearnPython的虚拟环境：

```
$ mkvirtualenv -p python3 LearnPython
$ workon
```

现在你会发现，shell的提示符改变了：

```
(LearnPython) YunYadeMacBook-Pro:~ yunya$
```

