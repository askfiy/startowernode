# 协程

协程（coroutine）并不是一个系统层面上真实存在的东西，而是由程序员进行创造。

你可以理解为协程是用户态的“线程”，因此协程也被称为“微线程”或者“纤程（Fiber）”。

协程能够做到在单线程下实现多线程的并发操作，这是非常厉害的一点。

既然协程和线程很像，那么它对比线程有什么优势呢？

- 协程和线程一样能够做切换，但是其切换代价远远小于线程，极大的提升了运行效率
- 协程中修改共享数据时不需要为数据加锁，因为协程本身就是一个单线程

协程有 2 大重要的概念，如下所示：

1. 作为用户态线程，它必然存在于内核态线程中，也就是说协程本身是一个非常纯粹的单线程
2. 协程最重要的意义就是切换

普通的代码运行总是顺序执行的，而如果我们有某种机制让它能够遇见 I/O 后进行自动切换执行就非常牛逼了。

例如下面这个例子，普通的运行打印结果是 1、2、3、4，而我们如果加上遇见 I/O 自动切换执行的策略的话它的打印结果就会变成 1、3、2、4：

```
import time


def task_01():
    print(1)
    time.sleep(1)  # I/O操作
    print(2)


def task_02():
    print(3)
    time.sleep(1)  # I/O操作
    print(4)


task_01()
task_02()

# 1
# 2
# 3
# 4
```

如果真的实现了上述的功能，那么使用单线程实现并发就变的不是那么遥不可及了。

## 生成器

我们可以利用生成器函数的 yield 关键字来实现代码的切换，如下所示：

```
import time


def task_01():
    print(1)
    time.sleep(1)  # I/O操作
    yield          # 手动切换
    print(2)


def task_02():
    print(3)
    time.sleep(1)  # I/O操作
    yield          # 手动切换
    print(4)


# 创建生成器对象
genObject_01 = task_01()
genObject_02 = task_02()

# 待执行任务列表
task_list = [genObject_01, genObject_02]

# 开启循环
while 1:

    # 可执行任务列表
    executable_list = task_list.copy()
    # 已执行任务列表
    completed_list = []

    for run_generator in executable_list:
        try:
            next(run_generator)
        except StopIteration as e:
            # 如果任务执行完毕，添加到已执行任务列表中
            completed_list.append(run_generator)

    for end_generator in completed_list:
        # 从待执行任务列表、已执行任务列表中删除已完成任务
        task_list.remove(end_generator)
        executable_list.remove(end_generator)

    else:
        # 清空已完成任务列表
        completed_list.clear()

    if not task_list:
        break

# 1
# 3
# 2
# 4
```

就这样一个基础的协程就做好了，但是你可以发现它的编码难度较大。

并且每次遇见 I/O 操作后都需要我们手动进行切换，十分的不方便。

## gevent 模块

针对 yield 协程的缺点，我们可以利用 gevent 模块，让整个碰见 I/O 操作就切换的过程变为自动进行。

它是一个第三方模块，所以你需要手动进行安装下载：

```
$ pip3 install gevent
```

代码如下所示：

```
import gevent
import time

from gevent import monkey

# 声明：下面代码碰见I/O操作自动切换
monkey.patch_all()


def task_01():
    print(1)
    time.sleep(1)  # I/O操作，自动切换
    print(2)
    time.sleep(1)  # I/O操作，自动切换


def task_02():
    print(3)
    time.sleep(1)  # I/O操作，自动切换
    print(4)


# 创建协程对象
fiberObject_01 = gevent.spawn(task_01, )  # 后面可传递参数
fiberObject_02 = gevent.spawn(task_02, )

# 任务列表
task_list = [fiberObject_01, fiberObject_02]

# 开始执行
gevent.joinall(
    task_list
)

# 1
# 3
# 2
# 4
```

## asyncio 模块

Python3.4 之后，官方提供了 asyncio 模块来提供对协程的支持。

下面是代码示例：

```
import asyncio


# 函数头部加上该装饰器，表明该函数是一个协程函数
@asyncio.coroutine
def task_01():
    print(1)
    yield from asyncio.sleep(1)  # I/O操作  自动切换
    print(2)


@asyncio.coroutine
def task_02():
    print(3)
    yield from asyncio.sleep(1)   # I/O操作  自动切换
    print(4)


# 创建协程对象，并将它包装为未来对象
fiberObject_01 = asyncio.ensure_future(task_01())
fiberObject_02 = asyncio.ensure_future(task_02())

# 任务列表
task_list = [fiberObject_01, fiberObject_02]

# 获取并开启循环
loop = asyncio.get_event_loop()

# 运行任务列表，并等待所有协程任务执行完毕
loop.run_until_complete(asyncio.wait(task_list))

# 1
# 3
# 2
# 4
```

我们要注意，如果一个协程函数中要调用另一个协程函数，则必须使用 yield from 关键字，yield form 后面可以运行的对象类型：

- 协程对象（期程对象，一个协程对象调用后的结果就是期程对象）
- 未来对象
- task 任务对象

另外，如果你想在协程函数中运行一些模块方法，那么一定要保证模块所提供的方法是异步方法，否则协程切换无效。

有关于 yield from 的使用，请参照 Python 基础生成器一章节。

## async&awit 语法

async 和 awit 语法在 Python3.5 中被支持，它本质和 asyncio 模块使用没有任何区别，只是简化了语法。

- async：用于定义协程函数，而不再使用@asyncio.coroutine 装饰器来进行定义
- awit：相当于 yield from 的简写形式，必须书写在协程函数中

以下是它的使用示例：

```
import asyncio


async def task_01():
    print(1)
    await asyncio.sleep(1)  # I/O操作  自动切换
    print(2)


async def task_02():
    print(3)
    await asyncio.sleep(1)   # I/O操作  自动切换
    print(4)


# 创建协程对象，并将它包装为未来对象
fiberObject_01 = asyncio.ensure_future(task_01())
fiberObject_02 = asyncio.ensure_future(task_02())

# 任务列表
task_list = [fiberObject_01, fiberObject_02]

# 获取并开启循环
loop = asyncio.get_event_loop()

# 运行任务列表，并等待所有协程任务执行完毕
loop.run_until_complete(asyncio.wait(task_list))

# 1
# 3
# 2
# 4
```

## 协程的作用

单纯的协程只是能够做切换，没有其他的任何特定功能。

也就是说，协程本身并不能提高并发量，但是如果能够加上碰见 I/O 自动切换的机制，那么协程的真正意义才能够被体现。

注意一点：

- 对于计算密集型的操作来说，利用协程来回进行切换是没有任何意义的，来回切换并保存状态，反倒会降低性能
- 对于 I/O 密集型的操作来说，利用协程在 I/O 等待时间中去切换并执行其他任务，当 I/O 结束后再进行回调，那么就会大大节省系统资源并提供高性能从而实现异步编程

接下来我们将使用一个简单的爬虫案例，来探究协程和多线程的执行效率到底提升了多少。

下面是多线程爬虫的示例，需要用到 requests 模块，所以你必须先安装它：

```
$ pip3 install requests
```

一个任务负责爬取资源，一个任务负责解析资源，为了更加方便对比，我们将整个运行时长都\*10：

```
import requests
import time

from concurrent.futures import ThreadPoolExecutor

headers = {
    "user-agent": "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36"
}

urls = [
    "https://www.jianshu.com/",
    "https://www.csdn.net/",
    "https://www.cnblogs.com/",
]

start = time.time()

# 任务函数
def func(url):
    response = requests.get(url, headers=headers)
    return response.text

# 绑定回调函数
def callback(futuresObject):
    print(futuresObject.result())


if __name__ == "__main__":
    with ThreadPoolExecutor() as executor:
        for url in urls:
            futuresObject = executor.submit(func, url)
            futuresObject.add_done_callback(callback)

    end = time.time()
    print(f"总计花费时长{(end - start) * 10}")

# 总计花费时长21.973369121551514
```

下面是协程爬虫的示例，由于协程中不允许同步方法的出现，而 requests 模块下的请求方法都是同步请求方法，所以需要使用 aiohttp 模块下的异步请求方法完成网络请求，你必须先安装它：

```
$ pip3 install aiohttp
```

示例如下：

```
import asyncio
import aiohttp
import time

headers = {
    "user-agent": "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36"
}

urls = [
    "https://www.jianshu.com/",
    "https://www.csdn.net/",
    "https://www.cnblogs.com/",
]

start = time.time()

# 任务函数
async def func(url):

    # async wait 相当于是执行一个异步的 __enter__ 方法
    async with aiohttp.ClientSession() as session:

        # 防止ssl抛出错误
        async with session.get(url, headers=headers, verify_ssl=False) as response:
            return await response.text()

# 绑定回调函数
def callback(futuresObject):
    print(futuresObject.result())


if __name__ == "__main__":
    # 创建协程任务列表
    task_list = []

    # 创建协程任务
    for url in urls:
        # 创建协程对象，并将它包装为未来对象
        fiberObject = asyncio.ensure_future(func(url))
        # 为未来对象绑定回调函数
        fiberObject.add_done_callback(callback)
        # 添加到协程任务列表
        task_list.append(fiberObject)

    # 创建事件循环
    event_loop = asyncio.get_event_loop()
    # 执行任务，并且主线程会等待协程任务列表中的所有任务处理完毕后再执行
    event_loop.run_until_complete(asyncio.wait(task_list))

    end = time.time()
    print(f"总计花费时长{(end - start) * 10}")

# 总计花费时长14.334436416625977
```

可以看见，协程爬虫比多线程爬虫的执行效率提高了不止一星半点，它真正称得上是 Python 中 I/O 密集型任务处理的大杀器。

# asyncio 详细探究

基于协程实现高性能的异步编程，这是 Python 未来发展方向，诸如 fastapi、tornado 等非常出名的异步 web 框架内部其实都与协程息息相关。

此外，Python web 领域大火的 Django 也在 3.x 版本中正式迈入异步领域，这意味着：异步编程，永远的神。

## 事件循环

事件循环是 asyncio 的关键，你可以将它理解为一个 while 循环，它会在循环中周期性的执行协程任务，并且在特定的条件下终止循环。

你可以参照生成器实现协程的代码或者下面的伪代码，这就是事件循环最直观的体现。

```
待执行任务列表 = [协程任务, 协程任务]

while 1:

    可执行任务列表 = 待执行任务列表[:]
    已执行任务列表 = []

    for 可执行任务 in 可执行任务列表：
        try:
            next(可执行任务)
        except StopIteration as e:
            已执行任务列表.append(可执行任务)

    for 已执行任务 in 已执行任务列表:
        待执行任务.remove(已执行任务)
        可执行任务.remove(已执行任务)

    else:
        已执行任务列表.clear()

    if not 待执行任务列表:
        break
```

使用 asyncio 模块时，你可以直接通过下面方式获取到该事件循环：

```
event_loop = asyncio.get_event_loop()
```

其实 asyncio 模块内部的事件循环是使用了一个双端队列实现的，类似：

```python
import collections


def f1():
    print("A")
    yield
    print("B")


def f2():
    print("C")
    yield
    print("D")


ready_tasks = collections.deque()


def create_task(generator):
    ready_tasks.append(generator)


def run():
    print("loop start")
    while True:
        try:
            gen = ready_tasks.popleft()
            next(gen)
            create_task(gen)
        except IndexError as e:
            print("loop done")
            break

        except StopIteration as e:
            pass


create_task(f1())
create_task(f2())

run()
```

## 协程对象

协程函数就是加上了@asyncio.coroutine 装饰器的函数，或者以 async 开头定义的函数，如下所示：

```
async def task():
    pass
```

它和生成器有着一样的特性，即加括号时不会调用函数体内部代码，而是返回一个协程对象：

```
fiberObject = task()
print(type(fiberObject))

# <class 'coroutine'>
```

## 任务执行

当要执行任务的时候，必须先获取事件循环，然后再将协程对象任务添加到事件循环中：

```
import asyncio

async def task():
    print("run task ...")

fiberObject = task()

event_loop = asyncio.get_event_loop()
event_loop.run_until_complete(fiberObject)
print("run main ...")

# run task ...
# run main ...
```

event_loop.run_until_complete()方法会添加协程对象至事件循环中，并且执行协程对象，直至协程对象运行完毕后才跳出事件循环。

在 Python3.7 之前，我们每次要运行一个协程对象都必须先获取事件循环，再调用 event_loop.run_until_complete()添加协程对象并执行，这很麻烦。

在 Python3.7 之后 asyncio 模块新增了 run()方法，它简化了这 2 步操作，如下所示：

```
import asyncio

async def task():
    print("run task ...")

fiberObject = task()

asyncio.run(fiberObject)

print("run main ...")

# run task ...
# run main ...
```

## await

await 关键字只能在协程函数中使用，类似于 yield from 只能在生成器函数中使用一样。

它与生成器函数相同，都用于运行嵌套在一个协程函数中的另一个协程函数，具体可参照 yield from：

```
import asyncio

async def inner():
    print("run ... inner")
    return 1

async def warpper():
    print("run ... warpper")
    result = await inner()
    print(result)

if __name__ == "__main__":
    fiberObject = warpper()
    event_loop = asyncio.get_event_loop()
    event_loop.run_until_complete(fiberObject)

# run ... warpper
# run ... inner
# 1
```

await 后面可以运行的对象类型：

- 协程对象（期程对象）
- 未来对象
- task 任务对象

## 未来对象

上述所有的代码都是在事件循环中添加一个协程对象，当事件循环中仅有一个对象时是无法做到遇见 I/O 就切换的操作的。

同时，协程对象本身不能绑定回调函数，所以要想绑定回调函数必须将它包装为未来对象。

在 Python3.7 之前，你可以使用 asyncio.ensure_future()函数将一个协程对象包装为未来对象，这样它就可以绑定回调函数了：

```
import asyncio

async def task():
    await asyncio.sleep(3)
    return 1

def callback(fiberObject):
    print(fiberObject.result())

if __name__ == "__main__":
    fiberObject = asyncio.ensure_future(task())
    fiberObject.add_done_callback(callback)

    event_loop = asyncio.get_event_loop()
    event_loop.run_until_complete(fiberObject)

# 1
```

如果想同时在事件循环中添加多个未来对象，你可以创建一个任务列表，并使用 asyncio.wait()方法来将任务列表中所有的未来对象都添加到事件循环中，如下所示：

```
import asyncio


async def task(param):
    print(f"run task{param}")
    await asyncio.sleep(3)
    return param


def callback(fiberObject):
    print(fiberObject.result())


if __name__ == "__main__":
    task_list = []
    for i in range(3):
        fiberObject = asyncio.ensure_future(task(i))
        fiberObject.add_done_callback(callback)
        task_list.append(fiberObject)

    event_loop = asyncio.get_event_loop()
    event_loop.run_until_complete(
        asyncio.wait(task_list)
    )

# run task0
# run task1
# run task2
# 0
# 1
# 2
```

这样一个异步提交的案例就出现了。

## 任务对象

任务对象是基于未来对象以及期程对象的一个封装。

如果你的 Python 版本是 Python3.7 或者更高，则可以直接创建任务对象并绑定回调函数，然后将它丢入的事件循环循环中：

```
import asyncio


async def task(param):
    print(f"run task{param}")
    await asyncio.sleep(3)
    return param


def callback(fiberObject):
    print(fiberObject.result())


async def main():
    task_list = []
    for i in range(3):
        fiberObject = asyncio.create_task(task(i), name=f"task{i}")
        fiberObject.add_done_callback(callback)
        task_list.append(fiberObject)

    done, pending = await asyncio.wait(task_list, timeout=None)
    print(done, pending)

if __name__ == "__main__":

    asyncio.run(main())

# run task0
# run task1
# run task2
# 0
# 1
# 2
```

## concurrent.futures.Futures

在 concurrent.futures 模块中，也有一个未来对象，该模块用于提供线程执行器和进程执行器，能够更加方便管理线程和进程，详情参照前面<<使用执行器提交任务>>一章。

concurrent.futures.Futures 和 asyncio 的 Futures 对象还是有所不同的，下面是官方文档的说明：

- 与 asyncio Futures 不同， concurrent.futures.Future 实例不能等待
- asyncio.Future.result() 和 asyncio.Future.exception() 不接受超时参数
- asyncio.Future.result() 和 asyncio.Future.exception() 在 Future 未完成时引发 InvalidStateError 异常
- 使用 asyncio.Future.add_done_callback() 注册的回调不会立即调用。 它们是用 loop.call_soon() 来安排的
  asyncio Future 与 concurrent.futures.wait() 和 concurrent.futures.as_completed() 函数不兼容

如果想将 concurrent.futures.Futures 变的和 asyncio.Future 相同，则可以使用 asyncio 所提供的方法 wrap_future()。

其实，一般在程序开发中我们要么统一使用 asycio 的协程实现异步操作、要么都使用进程池和线程池实现异步操作。但如果 协程的异步和 进程池/线程池的异步 混搭时，那么就会用到此功能了。

```
import time
import asyncio
import concurrent.futures

def func1():
    # 某个耗时操作
    time.sleep(2)
    return 1

async def main():
    loop = asyncio.get_running_loop()

    # 1. Run in the default loop's executor ( 默认ThreadPoolExecutor )
    # 第一步：内部会先调用 ThreadPoolExecutor 的 submit 方法去线程池中申请一个线程去执行func1函数，并返回一个concurrent.futures.Future对象

    # 第二步：调用asyncio.wrap_future将concurrent.futures.Future对象包装为asycio.Future对象。
    # 因为concurrent.futures.Future对象不支持await语法，所以需要包装为 asycio.Future对象 才能使用

    fut = loop.run_in_executor(None, func1)
    result = await fut
    print('default thread pool', result)

    # 2. Run in a custom thread pool:
    # with concurrent.futures.ThreadPoolExecutor() as pool:
    #     result = await loop.run_in_executor(
    #         pool, func1)
    #     print('custom thread pool', result)

    # 3. Run in a custom process pool:
    # with concurrent.futures.ProcessPoolExecutor() as pool:
    #     result = await loop.run_in_executor(
    #         pool, func1)
    #     print('custom process pool', result)

asyncio.run(main())
```

应用场景：当项目以协程式的异步编程开发时，如果要使用一个第三方模块，而第三方模块不支持协程方式异步编程时，就需要用到这个功能，例如：

```
import asyncio
import requests


async def download_image(url):
    # 发送网络请求，下载图片（遇到网络下载图片的IO请求，自动化切换到其他任务）
    print("开始下载:", url)

    loop = asyncio.get_event_loop()
    # requests模块默认不支持异步操作，所以就使用线程池来配合实现了。
    future = loop.run_in_executor(None, requests.get, url)

    response = await future
    print('下载完成')

    # 图片保存到本地文件
    file_name = url.rsplit('_')[-1]
    with open(file_name, mode='wb') as file_object:
        file_object.write(response.content)


if __name__ == '__main__':
    url_list = [
        'https://www3.autoimg.cn/newsdfs/g26/M02/35/A9/120x90_0_autohomecar__ChsEe12AXQ6AOOH_AAFocMs8nzU621.jpg',
        'https://www2.autoimg.cn/newsdfs/g30/M01/3C/E2/120x90_0_autohomecar__ChcCSV2BBICAUntfAADjJFd6800429.jpg',
        'https://www3.autoimg.cn/newsdfs/g26/M0B/3C/65/120x90_0_autohomecar__ChcCP12BFCmAIO83AAGq7vK0sGY193.jpg'
    ]

    tasks = [download_image(url) for url in url_list]

    loop = asyncio.get_event_loop()
    loop.run_until_complete(asyncio.wait(tasks))
```

# async for

## 异步可迭代对象

async for 语句是针对异步可迭代对象所使用的。

异步可迭代对象是指内部是实现了\_\_aiter\_\_()方法的对象，它必须返回一个 await iterator 对象

## 异步迭代器

异步迭代器是指内部实现了\_\_aiter\_\_()和\_\_anext\_\_()方法的对象，其中\_\_anext\_\_()方法必须返回一个 awaitable 对象。

async for 会处理异步迭代器的 \_\_anext\_\_()方法所返回的可等待对象，直到其引发一个 StopAsyncIteration 异常。由 [PEP 492](https://link.zhihu.com/?target=https%3A//www.python.org/dev/peps/pep-0492)引入。

## 代码实现

下面实现一个异步可迭代对象，它的可迭代器就是本身。

```
import asyncio


class Reader(object):
    """ 自定义异步迭代器（同时也是异步可迭代对象） """

    def __init__(self):
        self.count = 0

    async def readline(self):
        # await asyncio.sleep(1)
        self.count += 1
        if self.count == 100:
            return None
        return self.count

    def __aiter__(self):
        return self

    async def __anext__(self):
        val = await self.readline()
        if val == None:
            raise StopAsyncIteration
        return val


async def func():
    # 创建异步可迭代对象
    async_iter = Reader()
    # async for 必须要放在async def函数内，否则语法错误。
    async for item in async_iter:
        print(item)

asyncio.run(func())
```

# async wait

## 异步上下文管理

此种对象通过定义\_\_aenter\_\_()和\_\_aexit\_\_()方法来对 async with 语句中的环境进行控制。由 [PEP 492](https://link.zhihu.com/?target=https%3A//www.python.org/dev/peps/pep-0492) 引入。

```
import asyncio


class AsyncContextManager:
    def __init__(self):
        self.conn = conn

    async def do_something(self):
        # 异步操作数据库
        return 666

    async def __aenter__(self):
        # 异步链接数据库
        self.conn = await asyncio.sleep(1)
        return self

    async def __aexit__(self, exc_type, exc, tb):
        # 异步关闭数据库链接
        await asyncio.sleep(1)


async def func():
    async with AsyncContextManager() as f:
        result = await f.do_something()
        print(result)


asyncio.run(func())
```

这个异步的上下文管理器还是比较有用的，平时在开发过程中 打开、处理、关闭 操作时，就可以用这种方式来处理。

## \_\_await\_\_

当 await 一个非协程对象的普通对象时，Python 解释器会尝试调用其 \_\_await\_\_ 方法获取一个异步迭代器。

- 如果对象是一个协程对象，直接执行该协程。
- 否则，尝试调用对象的**await**方法，获取一个迭代器对象用于执行异步操作。
- 如果对象既不是协程对象，也没有定义**await**方法，将引发 AttributeError 异常，表示对象不是一个有效的可等待对象。

代码示例：

```
import asyncio


class MyAwaitable:
    def __await__(self):
        return self._await_generator().__await__()

    async def _await_generator(self):
        # 执行异步操作
        await asyncio.sleep(1)
        # 返回结果作为可等待对象的结果
        return 'Hello, world!'


async def main():
    awaitable = MyAwaitable()
    result = await awaitable
    print(result)

asyncio.run(main())
```

# uvloop

Python 标准库中提供了 asyncio 模块，用于支持基于协程的异步编程。

uvloop 是 asyncio 中的事件循环的替代方案，替换后可以使得 asyncio 性能提高。事实上，uvloop 要比 nodejs、gevent 等其他 python 异步框架至少要快 2 倍，性能可以比肩 Go 语言。

安装 uvloop

```text
pip3 install uvloop
```

在项目中想要使用 uvloop 替换 asyncio 的事件循环也非常简单，只要在代码中这么做就行。

```python
import asyncio
import uvloop
asyncio.set_event_loop_policy(uvloop.EventLoopPolicy())

# 编写asyncio的代码，与之前写的代码一致。

# 内部的事件循环自动化会变为uvloop
asyncio.run(...)
```

注意：知名的 asgi uvicorn 内部就是使用的 uvloop 的事件循环。

# 官方文档

在程序中只要看到 async 和 await 关键字，其内部就是基于协程实现的异步编程，这种异步编程是通过一个线程在 I/O 等待时间去执行其他任务，从而实现并发。

以上就是异步编程的常见操作，其他具体的内容可参考官方文档。

- 中文版：[https://docs.python.org/zh-cn/3.8/library/asyncio.html](https://link.zhihu.com/?target=https%3A//docs.python.org/zh-cn/3.8/library/asyncio.html)
- 英文本：[https://docs.python.org/3.8/library/asyncio.html](https://link.zhihu.com/?target=https%3A//docs.python.org/3.8/library/asyncio.html)

此外，此篇文章基本借鉴于武 Sir，再次特别感谢。

- [原文地址](https://zhuanlan.zhihu.com/p/137057192)
