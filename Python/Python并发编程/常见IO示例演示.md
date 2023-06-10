# 阻塞I/O的socket服务端

使用socket模块与concurrent.futures实现阻塞式I/O的socket服务端。

开启多个子线程，每个子线程单独负责一个链接，这意味着该服务器最大的并发量取决于你CPU能够打开的最大有效线程数：

```
import socket
import threading
import concurrent.futures


class TcpServer:
    def __init__(self, bind_addr=("localhost", 8001), allow_reuse_address=False):

        self.socket = socket.socket()
        self.backlog = 5
        self.bufsize = 1024
        self.server_addr = bind_addr
        self.allow_reuse_address = allow_reuse_address

        # 默认创建的线程数为cpu核心数 * 5
        self.executor = concurrent.futures.ThreadPoolExecutor()

    def run_server(self):
        """运行服务"""

        self.server_bind()
        self.server_activate()
        self.handle_request()

    def server_bind(self):
        """绑定地址"""

        if self.allow_reuse_address:
            self.socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

        self.socket.bind(self.server_addr)

    def server_activate(self):
        """开启监听"""

        self.socket.listen(self.backlog)

    def handle_request(self):
        """处理链接"""

        while 1:
            # 阻塞点1：accept()函数会导致程序卡住，直至有新的链接请求到来
            conn, addr = self.socket.accept()
            self.executor.submit(
                self.handle_communicate, conn, addr)

    def handle_communicate(self, conn, addr):
        """多线程处理通信"""

        thName = threading.current_thread().name
        print(f"{addr} connect server, handle thread : {thName}")
        while 1:
            try:
                # 阻塞点2：recv()函数会导致程序卡住，直至有新的信息放入conn双向链接通道中
                data = conn.recv(self.bufsize)
                if not data:
                    raise Exception(
                        f"{addr} close connect, handle thread : {thName}")
                conn.send(data.upper())
            except Exception as e:
                self.close_connect(conn, e)
                break

    def close_connect(self, conn, msg):
        """关闭链接"""
        
        conn.close()
        print(msg)


if __name__ == "__main__":
    server = TcpServer(allow_reuse_address=True)
    server.run_server()
```





# 非阻塞I/O的socket服务端

将上述的socket服务端改为非阻塞的：

```
import socket
import threading
import concurrent.futures


class TcpServer:
    def __init__(self, bind_addr=("localhost", 8001), allow_reuse_address=False):
        
        self.socket = socket.socket()
        
        # 设置为非阻塞
        self.socket.setblocking(False)
        
        self.backlog = 5
        self.bufsize = 1024
        self.server_addr = bind_addr
        self.allow_reuse_address = allow_reuse_address
        
        # 默认创建的线程数为cpu核心数 * 5
        self.executor = concurrent.futures.ThreadPoolExecutor()

    def run_server(self):
        """运行服务"""
        
        self.server_bind()
        self.server_activate()
        self.handle_request()

    def server_bind(self):
        """绑定地址"""
        
        if self.allow_reuse_address:
            self.socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

        self.socket.bind(self.server_addr)

    def server_activate(self):
        """开启监听"""
        
        self.socket.listen(self.backlog)

    def handle_request(self):
        """处理链接"""
        
        while 1:
            
            # 设置非阻塞I/O后，accept()这个原本会阻塞的函数变的非阻塞了
            # 所以这里要不断的检测是否有新的链接请求
            try:
                conn, addr = self.socket.accept()
                self.executor.submit(
                    self.handle_communicate, conn, addr)
            except BlockingIOError as e:
                continue

    def handle_communicate(self, conn, addr):
        """多线程处理通信"""
        
        thName = threading.current_thread().name
        print(f"{addr} connect server, handle thread : {thName}")
        
        while 1:
            try:
                # 设置非阻塞I/O后，recv()这个原本会阻塞的函数变的非阻塞了
                # 所以这里要不断的检测是否有新的链接请求
                data = conn.recv(self.bufsize)
                if not data:
                    raise Exception(
                        f"{addr} close connect, handle thread : {thName}")
                conn.send(data.upper())
                
            except BlockingIOError as e:
                continue
                
            except Exception as e:
                self.close_connect(conn, e)
                break

    def close_connect(self, conn, msg):
        """关闭链接"""
        conn.close()
        print(msg)


if __name__ == "__main__":
    server = TcpServer(allow_reuse_address=True)
    server.run_server()
```







# I/O多路复用的socket服务端

上述的2个示例中，每个线程都会负责监听双向链接通道和处理通信请求，所以这意味着你的系统最大有效线程数决定了你服务器的最高并发数，显然效率是十分低下的。

而如果有一种机制让一个线程来监听所有的双向链接通道的话那么并发量就会得到质的提升，它不再是单纯的一对一服务了，而是一对多服务。

下面将利用select模块实现select监听模式的I/O多路复用机制，仅仅利用单线程，最大并发数量就能提升到2048个：

```
import socket
import select


class TcpServer:
    def __init__(self, bind_addr=("localhost", 8001), allow_reuse_address=False):
        
        self.socket = socket.socket()
        
        # 设置为非阻塞（高性能的边缘触发必备条件）
        self.socket.setblocking(False)

        self.backlog = 5
        self.bufsize = 1024
        self.server_addr = bind_addr
        self.allow_reuse_address = allow_reuse_address

        # 1.存放注册读取事件描述符的列表，当有可读事件发生时意味着客户端请求建立与服务端的双向链接通道
        # 或者客户端有新的消息发送给服务端
        self.r_list = []
        # 2.存放注册可写事件描述符的列表，当有可写事件发生时意味着服务端可以主动向客户端发送信息
        self.w_list = []
        # 3.存放注册错误事件描述符的列表，...
        self.e_list = []
        # 4.存放被监听描述符回调函数的字典
        self.callback_dict = {}
        # 5.循环监听的时间间隔
        self.poll_interval = 0.5

    def run_server(self):
        """运行服务"""

        self.server_bind()
        self.server_activate()
        self.server_forever()

    def server_bind(self):
        """绑定地址"""

        if self.allow_reuse_address:
            self.socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

        self.socket.bind(self.server_addr)

    def server_activate(self):
        """开启监听"""
        
        self.socket.listen(self.backlog)

    def server_forever(self):
        """I/O多路复用"""
        
        # 注册self.sockt的可读事件并绑定回调函数为self.handle_request
        # 当客户端试图建立与服务端的双向链接通道时将触发self.socket的可读事件
        self.register(obj=self.socket, event_type="read",
                      callback=self.handle_request)

        # 开始循环监听描述符，每次while循环间隔0.5s
        while 1:
            # 当可读事件列表、可写事件列表、错误事件列表中任何一个fd的状态发生改变后
            # 都会添加到r、w、e中
            r, w, e = select.select(
                self.r_list, self.w_list, self.e_list, self.poll_interval)
            # 查看是否有可读事件发生，如果有就运行其回调函数
            for fd in r:
                # 运行回调函数
                fd_callback = self.callback_dict[fd]["r"]
                fd_callback(fd)

    def handle_request(self, fd_socket):
        """处理链接"""
        
        conn, addr = fd_socket.accept()
        print(f"{addr} connect server")

        # 注册conn的可读事件并绑定回调函数为self.handle_communicate
        # 当该客户端向服务端发送信息时将触会触发conn的可读事件
        self.register(obj=conn, event_type="read",
                      callback=self.handle_communicate)

    def handle_communicate(self, fd_conn):
        """处理通信"""
        
        # 实际上就是链接服务端的客户端地址
        addr = getattr(fd_conn, "getsockname")()

        try:
            data = fd_conn.recv(self.bufsize)

            # 针对Unix环境：客户端强制断开链接
            if not data:
                raise Exception(
                    f"{addr} close connect")
            fd_conn.send(data.upper())

        # 针对Windows环境：客户端强制断开链接
        except Exception as e:
            # 关闭双向链接通道并注销可读事件，这意味着不再监听该双向链接通道
            self.close_connect(fd_conn, e)
            self.unregister(obj=fd_conn, event_type="read")

    def close_connect(self, conn, msg):
        """关闭链接"""
        conn.close()
        print(msg)

    def register(self, obj, event_type, callback):
        """事件注册"""

        if event_type == "read":
            self.r_list.append(obj)
            self.callback_dict[obj] = {"r": callback}
        elif event_type == "write":
            self.w_list.append(obj)
            self.callback_dict[obj] = {"w": callback}
        elif event_type == "error":
            self.e_list.append(obj)
            self.callback_dict[obj] = {"e": callback}
        else:
            raise TypeError("unknown event type %r" % event_type)

    def unregister(self, obj, event_type):
        """取消注册"""

        if event_type == "read":
            self.r_list.remove(obj)
            del self.callback_dict[obj]["r"]
        elif event_type == "write":
            self.w_list.remove(obj)
            del self.callback_dict[obj]["w"]
        elif event_type == "error":
            self.e_list.remove(obj)
            del self.callback_dict[obj]["e"]
        else:
            raise TypeError("unknown event type %r" % event_type)


if __name__ == "__main__":
    server = TcpServer(allow_reuse_address=True)
    server.run_server()
```



# 异步的socket服务端

selectors模块是select模块的升级版，他还会根据平台自动选择监听方式，是十分方便的。

　[官方文档](https://docs.python.org/zh-cn/3.6/library/selectors.html)

> 注意，Windows平台下不支持poll或者epoll模式
>
> epoll模式仅有Linux平台支持



现在很多框架的内部，如著名的异步框架tornado，Twisted ，等等都是通过epoll实现的异步，其实epoll到底属于不属于异步在网络上有很大的争议，下面会有一节会和大家一起讨论一下epoll和异步之间的关系。

下面是它的使用案例，基本步骤和select模块使用类似，也是注册事件、绑定回调、监听描述符。

相较于使用select模式的I/O多路复用机制来说，epoll模式的I/O多路复用机制单线程下的效率和最大支持并发量提升了不少：

```
import socket
import selectors


class TcpServer:
    def __init__(self, bind_addr=("localhost", 8001), allow_reuse_address=False):
    
        self.socket = socket.socket()
        
        # 设置为非阻塞（高性能的边缘触发必备条件）
        self.socket.setblocking(False)

        self.backlog = 5
        self.bufsize = 1024
        self.server_addr = bind_addr
        self.allow_reuse_address = allow_reuse_address

        # 1.自动选择监听模式:select、poll、epoll
        self.sel = selectors.DefaultSelector()

        # 2.循环监听的时间间隔
        self.poll_interval = 0.5

    def run_server(self):
        """运行服务"""

        self.server_bind()
        self.server_activate()
        self.server_forever()

    def server_bind(self):
        """绑定地址"""

        if self.allow_reuse_address:
            self.socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

        self.socket.bind(self.server_addr)

    def server_activate(self):
        """开启监听"""
        
        self.socket.listen(self.backlog)

    def server_forever(self):
        """I/O多路复用"""
        
        # 注册self.sockt的可读事件并绑定回调函数为self.handle_request
        # 当客户端试图建立与服务端的双向链接通道时将触发self.socket的可读事件
        # 如果是要注册可读写事件，可以传入 events = selectors.EVENT_READ + selectors.EVENT_WRITE,
        # 或者传入 3 即可
        self.sel.register(self.socket, selectors.EVENT_READ, self.handle_request)

        # 开始循环监听描述符，每次while循环间隔0.5s, 如果这里是 0, 则会一直阻塞
        # 直到有一个 fd 的事件出现了
        while 1:
            # 当有已注册描述符状态发生改变后，都会添加至fd_list
            # fd_list = [(fd, event), ...]
            fd_list = self.sel.select(self.poll_interval)
            for fd, event in fd_list:
                # 运行回调函数
                fd_callback = fd.data
                fd_callback(fd.fileobj)

    def handle_request(self, fd_socket):
        """处理链接"""
        
        conn, addr = fd_socket.accept()
        print(f"{addr} connect server")

        # 注册conn的可读事件并绑定回调函数为self.handle_communicate
        # 当该客户端向服务端发送信息时将触会触发conn的可读事件
        self.sel.register(conn, selectors.EVENT_READ, self.handle_communicate)

    def handle_communicate(self, fd_conn):
        """处理通信"""
        
        # 实际上就是链接服务端的客户端地址
        addr = getattr(fd_conn, "getsockname")()

        try:
            data = fd_conn.recv(self.bufsize)

            # 针对Unix环境：客户端强制断开链接
            if not data:
                raise Exception(
                    f"{addr} close connect")
            fd_conn.send(data.upper())

        # 针对Windows环境：客户端强制断开链接
        except Exception as e:
            # 关闭双向链接通道并注销可读事件，这意味着不再监听该双向链接通道
            self.close_connect(fd_conn, e)
            self.sel.unregister(fd_conn)

    def close_connect(self, conn, msg):
        """关闭链接"""
        
        conn.close()
        print(msg)


if __name__ == "__main__":
    server = TcpServer(allow_reuse_address=True)
    server.run_server()
```



# 更高性能的提升

上述的I/O多路复用服务端都是用单线程来实现对于请求的处理，如果加上了多线程进行处理请求，那么效率上又会有很大的提升。

如下所示：

```
import socket
import selectors
import concurrent.futures


class TcpServer:
    def __init__(self, bind_addr=("localhost", 8001), allow_reuse_address=False):
        self.socket = socket.socket()

        # 设置为非阻塞（高性能的边缘触发必备条件）
        self.socket.setblocking(False)

        self.backlog = 5
        self.bufsize = 1024
        self.server_addr = bind_addr
        self.allow_reuse_address = allow_reuse_address

        # 默认创建的线程数为cpu核心数 * 5
        self.executor = concurrent.futures.ThreadPoolExecutor()

        # 1.自动选择监听模式:select、poll、epoll
        self.sel = selectors.DefaultSelector()

        # 2.循环监听的时间间隔
        self.poll_interval = 0.5

    def run_server(self):
        """运行服务"""

        self.server_bind()
        self.server_activate()
        self.server_forever()

    def server_bind(self):
        """绑定地址"""

        if self.allow_reuse_address:
            self.socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

        self.socket.bind(self.server_addr)

    def server_activate(self):
        """开启监听"""

        self.socket.listen(self.backlog)

    def server_forever(self):
        """I/O多路复用"""

        # 注册self.sockt的可读事件并绑定回调函数为self.handle_request
        # 当客户端试图建立与服务端的双向链接通道时将触发self.socket的可读事件
        self.sel.register(self.socket, selectors.EVENT_READ, self.handle_request)

        # 开始循环监听描述符，每次while循环间隔0.5s
        while 1:
            # 当有已注册描述符状态发生改变后，都会添加至fd_list
            # fd_list = [(fd, event), ...]
            fd_list = self.sel.select(self.poll_interval)
            for fd, event in fd_list:
                # 多线程运行回调函数
                fd_callback = fd.data
                self.executor.submit(fd_callback, fd.fileobj)

    def handle_request(self, fd_socket):
        """处理链接"""

        conn, addr = fd_socket.accept()
        print(f"{addr} connect server")

        # 注册conn的可读事件并绑定回调函数为self.handle_communicate
        # 当该客户端向服务端发送信息时将触会触发conn的可读事件
        self.sel.register(conn, selectors.EVENT_READ, self.handle_communicate)

    def handle_communicate(self, fd_conn):
        """处理通信"""

        # 实际上就是链接服务端的客户端地址
        addr = getattr(fd_conn, "getsockname")()

        try:
            data = fd_conn.recv(self.bufsize)

            # 针对Unix环境：客户端强制断开链接
            if not data:
                raise Exception(
                    f"{addr} close connect")
            fd_conn.send(data.upper())

        # 针对Windows环境：客户端强制断开链接
        except Exception as e:
            # 关闭双向链接通道并注销可读事件，这意味着不再监听该双向链接通道
            self.close_connect(fd_conn, e)
            self.sel.unregister(fd_conn)

    def close_connect(self, conn, msg):
        """关闭链接"""

        conn.close()
        print(msg)


if __name__ == "__main__":
    server = TcpServer(allow_reuse_address=True)
    server.run_server()
```





# epoll到底是同步还是异步

摘自知乎：

学习tornado、asyncio这些异步网络库时，遇到了同样的问题，网上查到的也都说不明白，原来几个概念没搞清楚。

- I/O操作有多种，处理socket是一种，磁盘读写也是一种，暂时分为网络I/O和文件I/O、
- I/O多路复用是操作系统级别的，属于linux操作系统的五种I/O模型中的一种，是操作系统级别同步非阻塞的
- 异步网络库 twisted、tornado、asyncio所谓的异步，是应用级别的异步，底层确实是基于epoll实现，基本上都是处理网络 I/O，而且都是基于事件驱动的，使用时划分事件也大多是根据网络请求。

- 操作系统级别的异步I/O才是真正异步非阻塞的，然后并没有很多应用，貌似unix平台没有，windows NT平台有也很少，而且基本都是文件I/O
- I/O多路复用的实现用的比较多，linux平台的epoll，windows平台的select等，基于epoll，select的应用大多是实现网络I/O。

所以，遇到异步框架，异步网络库都应该知道是应用级别的异步，而且基本上都是基于epoll/select实现的。

已知tornado会根据系统平台，选择epoll还是select。

实现高并发有多种方式，python多进程可以利用多核优势，协程(gevent、asyncio)可以实现应用级别的异步，celery实现任务异步，消息队列实现服务解耦等等，项目中可以根据实际情况选择或组合不同的方式。

- tornado(web框架/异步网络库)：进程+异步+epoll

- asyncio：协程+epoll，使用中需要相应的异步库，如常用的aiohttp

- gevent：greenlet+猴子补丁，猴子补丁把socket相关库改为非阻塞
