# multiprocessing模块

Python中提供了multiprocessing模块来实现进程并发编程，官方文档如下：

[官方文档](https://docs.python.org/zh-cn/3.6/library/multiprocessing.html)

由于GIL锁的存在，所以CPython中多线程是不能够并行运行的，但是多进程可以并行运行，该模块用到的地方基本很少，但是仍然需要进行掌握。

此外，它和threading模块99%的接口都一模一样，只有少量的差别。



# 添加子进程

## 针对不同平台选择添加子进程的方式



multiprocessing模块针对不同的平台，添加子进程的方式也有所区别：

- spawn：该方式是Windows平台下的默认方式，它会创建一个新的解释器进程，速度比较慢
- fork：该方式是Unix平台下的默认方式，内部会通过os.fork()产生一个新的解释器分叉

需要注意的是，如果平台是Windows，则必须将启动代码书写到if \_\_name\_\_ == "\_\_main\_\_”语句的下面，否则将会抛出异常。

如何指定新进程的启动方式？示例如下：

```
import multiprocessing

if __name__ == "__main__":
    multiprocessing.set_start_method("fork")
```



## 实例化Process类

使用该方式新增子线程任务是比较常见的，也是推荐使用的。

简单的代码示例如下，创建3个子进程并向其添加任务，然后运行并打印它们的PID和进程名字：

```
import multiprocessing
import time


def task(params):
    print("sub process run")
    currentThread = multiprocessing.current_process()
    time.sleep(3)
    print("current subProcess id : %s\ncurrent sub process name : %s\ncurrent sub process params : %s" % (
        currentThread.ident, currentThread.name, params))


if __name__ == "__main__":
    print("main Process run")
    for item in range(3):
        subProcessIns = multiprocessing.Process(target=task, args=(item, ))
        subProcessIns.start()
    print("main Process run end")

# main Process run
# main Process run end
# sub process run
# current subProcess id : 7761
# current sub process name : Process-1
# current sub process params : 0
# sub process run
# current subProcess id : 7762
# current sub process name : Process-2
# current sub process params : 1
# sub process run
# current subProcess id : 7763
# current sub process name : Process-3
# current sub process params : 2
```





## 自定义类覆写run()方法

上面的子进程任务对象是一个全局函数，我们也可以将它作为方法来进行调用。

书写一个类并继承Process类，覆写run()方法即可：

```
import multiprocessing
import time


class TaskClass(multiprocessing.Process):
    def __init__(self, params):
        self.params = params
        super(__class__, self).__init__()

    def run(self):
        print("sub process run")
        currentProcess = multiprocessing.current_process()
        time.sleep(3)
        print("current sub process id : %s\ncurrent sub process name : %s\ncurrent sub process params : %s" % (
            currentProcess.ident, currentProcess.name, self.params))


if __name__ == "__main__":
    print("main process run")
    for item in range(3):
        subThreadIns = TaskClass(item)
        subThreadIns.start()
    print("main process run end")

# main process run
# main process run end
# sub process run
# current sub process id : 7800
# current sub process name : TaskClass-1
# current sub process params : 0
# sub process run
# current sub process id : 7801
# current sub process name : TaskClass-2
# current sub process params : 1
# sub process run
# current sub process id : 7802
# current sub process name : TaskClass-3
# current sub process params : 2
```



# multiprocessing模块方法大全

以下是multiprocessing模块提供的类或方法：

| 类或方法                                      | 描述                                           | 返回值               |
| --------------------------------------------- | ---------------------------------------------- | -------------------- |
| multiprocessing.Process(target, args, kwargs) | 创建并返回一个进程对象                         | processObject        |
| multiprocessing.active_children()             | 查看当前进程下的所有子进程对象，以列表形式返回 | [processObject, ...] |
| multiprocessing.current_process()             | 获取当前的进程对象                             | processObject        |

以下是好伙伴os模块所提供的2个方法：

| 方法         | 描述                      | 返回值 |
| ------------ | ------------------------- | ------ |
| os.getpid()  | 返回当前进程pid           | int    |
| os.getppid() | 返回当前进程的父进程的pid | int    |

# processObject方法大全

以下是针对进程对象提供的属性或者方法：

| 方法/属性                        | 描述                                                         | 返回值      |
| -------------------------------- | ------------------------------------------------------------ | ----------- |
| processObject.start()            | 通知系统该进程调度完毕，可以随时进行启动，一个进程对象只能运行一次该方法，若多次运行则抛出RunTimeError异常 | ...         |
| processObject.join(timeout=None) | 主进程默认会等待子进程运行结束后再继续执行，timeou为等待的秒数，如不设置该参数则一直等待。 | ...         |
| processObject.close()            | 关闭进程                                                     | ...         |
| processObject.terminate()        | 终止进程                                                     | ...         |
| processObject.kill()             | 终止进程                                                     | ...         |
| processObject.is_alive()         | 查看进程对象是否存活                                         | bool        |
| processObject.ident              | 获取进程对象的编号                                           | int         |
| processObject.pid                | 获取进程对象的编号                                           |             |
| processObject.name               | 获取或者设置进程对象的名字                                   | str or None |
| processObject.daemon             | 查看进程对象是守护进程                                       | bool        |
| processObject.exitcode           | 子进程的退出代码。如果进程尚未终止，这将是None。负值 *-N* 表示子进程被信号 *N* 终止 | int         |
| processObject.authkey            | 获取进程的身份验证密码                                       | bytes       |





# 守护进程示例

multiprocessing模块的守护进程和threading的守护线程设置有所不同。

它是通过赋值来进行设置的，如下所示：

```
import multiprocessing
import time


class TaskClass(multiprocessing.Process):

    def run(self):
        pName = multiprocessing.current_process().name
        print("%s start run" % pName)
        time.sleep(3)
        print("%s run end" % pName)


if __name__ == "__main__":
    print("main process start run")
    processLst = []
    for i in range(3):
        processLst.append(TaskClass())
    for ins in processLst:
        # 注意，守护进程的设置必须在进程未启动时设置
        ins.daemon = True
        ins.start()

    print("main process carry on run")
    print("main process run end")

# main process start run
# TaskClass-1 start run
# main process carry on run
# main process run end
```



# multiprocessing与threading模块异同

　　1.创建子进程的方式针对不同平台有着差异化

　　2.关于守护线程的设置接口是setDaemon(True)，而关于守护进程的接口是deamon = True

　　3.multiprocessing模块下的获取进程名与设置进程名没有threading模块下的getName()和setName()，而是直接采取属性name进行操作



# 锁的使用

multiprocessing模块中锁的接口和使用与threading中锁的接口和使用一致。

所以这里仅介绍一个lock锁即可：

```
import multiprocessing

num = 0


def add():
    lock.acquire()
    global num
    for i in range(10_000_000):
        num += 1
    lock.release()


def sub():
    lock.acquire()
    global num
    for i in range(10_000_000):
        num -= 1
    lock.release()


if __name__ == "__main__":
    lock = multiprocessing.Lock()

    subProcess01 = multiprocessing.Process(target=add)
    subProcess02 = multiprocessing.Process(target=sub)

    subProcess01.start()
    subProcess02.start()

    subProcess01.join()
    subProcess02.join()

    print("num result : %s" % num)

# 结果三次采集
# num result : 0
# num result : 0
# num result : 0
```

