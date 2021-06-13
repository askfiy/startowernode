# subprocess简介

subprocess模块最早在Python2.4中引入，它会生成一个子进程，该子进程可以执行shell命令，并且会监听系统的标准输入管道、标准输出管道、标准错误管道，在命令执行完毕后，将结果进行返回到对应的管道中。

[官方文档](https://docs.python.org/zh-cn/3.6/library/subprocess.html#module-subprocess)

如下图所示：

![image-20210527190008666](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210527190008666.png)



它的功能虽然看上去十分单一，但是应用是非常广泛的。

试想，你是一名运维人员，编写了1个脚本，每天定时定点的在100台机器上获得它们的状态信息，就可以用到该模块。

让脚本在宿主机上通过该模块执行命令，并且拿到命令的返回结果，再通过网络返回结果并对其进行分析，依此判定各个宿主机的工作状态。



# 简单的使用

对于简单的使用，记住这4个方法即可，如下表所示：

| 方法                      | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| subprocess.Popen(...)     | 执行系统命令，并将执行结果放入对应的管道中，返回一个Popen对象 |
| PopenObject.stdout.read() | 从标准输出管道中获取执行结果                                 |
| PopenObject.stderr.read() | 从标准错误管道中获取执行结果                                 |
| PopenObject.stdin.write() | 通过标准输入管道与系统进行交互                               |

subprocess.Popen()的可指定参数比较多，下面是它的签名：

```
    def __init__(self, args, bufsize=-1, executable=None,
                 stdin=None, stdout=None, stderr=None,
                 preexec_fn=None, close_fds=_PLATFORM_DEFAULT_CLOSE_FDS,
                 shell=False, cwd=None, env=None, universal_newlines=False,
                 startupinfo=None, creationflags=0,
                 restore_signals=True, start_new_session=False,
                 pass_fds=(), *, encoding=None, errors=None):
```

常用参数的释义如下：

- args：将要执行的命令，可以是str类型或者list、tuple类型
- bufsize：指定缓冲大小，0是没有，1是默认
- executable：要执行的替换程序
- stdin/stdout/stderr：标准输入、输出、错误管道的句柄
- preexec_fn：仅在Unix平台下有效。指定一个可调用对象，通常是指函数，它将在fork出的子程序运行之前调用
- close_sds：在Windows平台下，如果该参数指定为True，则fork出的子程序将不会继承父程序的标准输入、输出、错误管道中传输的内容，一般设置默认即可
- shell：如果为True，则通过shell执行命令
- cwd：设置子进程执行时的工作目录
- env：用于指定子进程的环境变量，如果为None，子进程将继承父进程的环境变了
- Universal_newlines：如果为True，则不区分平台，统一将换行符定义为\\n

简单的执行命令并获取返回结果：

```
import subprocess

popenObject = subprocess.Popen(
    args="ping www.baidu.com",
    shell=True,
    stdin=subprocess.PIPE,
    stderr=subprocess.PIPE,
    stdout=subprocess.PIPE,
)

successMessage, errorMessage = popenObject.stdout.read(), popenObject.stderr.read()
print(successMessage.decode("gbk"))
print(errorMessage.decode("gbk"))

popenObject.stdout.close()
popenObject.stderr.close()
```

注意事项：

- 如果你的测试环境是Windows，则对执行结果的解码方式需要使用GBK，因为Windows的终端字符编码方式就是GBK。而如果是Unix平台只需要使用UTF8即可。



通过stdin与与宿主机进行交互：

```
import subprocess

# 执行的语句
stdInCommand = """print (\"Hello, world\")"""

# 通过subprocess在测试机上启动1个新的Python REPT交互式环境
popenObject = subprocess.Popen(
    args="python",
    shell=True,
    stdin=subprocess.PIPE,
    stderr=subprocess.PIPE,
    stdout=subprocess.PIPE,
    )

# 写入内容，进行交互，注意每次交互完毕后都要立即关闭管道
popenObject.stdin.write(stdInCommand.encode("utf8"))
popenObject.stdin.close()

# 获取并打印结果
successMessage, errorMessage = popenObject.stdout.read(), popenObject.stderr.read()
print(successMessage.decode("gbk"))
print(errorMessage.decode("gbk"))

popenObject.stdout.close()
popenObject.stderr.close()
```



# 更多的操作

以下方法了解即可，其实用的并不多：

| 方法                         | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| subprocess.run()             | 执行命令，不返回结果，拥有timeout参数，可设定超时时间        |
| subprocess.call()            | 执行命令，并且返回执行状态（bool类型，0成功，1失败）         |
| subprocess.check_call()      | 执行命令，并且返回执行结果和执行状态，如果命令执行失败则抛出异常， |
| subprocess.getstatusoutput() | 执行命令，返回1个tuple，[0]是执行状态，[1]是执行结果         |
| subprocess.getoutput()       | 执行命令，返回执行结果（str类型）                            |
| subprocess.check_output()    | 执行命令，返回执行结果（bytes类型）                          |

这里就不再进行演示了，感兴趣的可以参见官方文档。

