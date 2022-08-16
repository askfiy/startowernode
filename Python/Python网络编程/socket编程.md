# 前瞻知识

## C/S架构

C/S架构是一种由服务端（Server）和客户端（Client）组成的双层架构。

互联网中处处充满了C/S架构（Client/Server），比如我们需要玩英雄联盟，就必须连接至英雄联盟的服务器上，那么对于我们玩家来说它的英雄联盟服务器就是Server端，而我们必须要下载一个英雄联盟Client端才能够去和英雄联盟Server端进行数据交互。



## 五层协议

互联网的协议实际上就是为了让计算机之间互相进行通信而产生的，本身并没有层级之分。

为了便于理解，我们可以按照功能将它们划分成5层或者7层，如下表所示：

| **层级**   | **功能**                                                     | 相关协议            |
| ---------- | ------------------------------------------------------------ | ------------------- |
| 应用层     | 用于规定应用数据的格式，提供给各个应用程序以便于彼此之间进行通信 | HTTP、FTP           |
| 传输层     | 用于区分该系统上的唯一一个网络应用程序                       | TCP、UDP            |
| 网络层     | 用于区分广播域，防止网络风暴的发生                           | IP                  |
| 数据链路层 | 用于划分电信号以及提供IP地址和MAC地址相互转换的功能          | 以太网协议、ARP协议 |
| 物理层     | 用于传输电信号，它是网络传输数据的基石                       |                     |



## socket抽象层

计算机网络的核心就是一堆协议，想开发基于网络通信的软件就必须遵守这些协议。

但是由于学习协议的代价巨大，故我们需要一种高度抽象的中间层来承上启下便于我们进行快速开发。

此时，socket产生了，socket位于应用层和传输层之间，它向下封装了各种协议，用户只需要通过socket提供的接口就能快速的开发出基于网络通信的软件，而并不需要深入的去研究某些底层协议，如TCP、UDP等。

![image-20210627143602426](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210627143602426.png)

为什么要学习socket呢？因为它是各种与网络沾边的应用框架的底层实现，如Django、requests等只要与网络有关系的框架或模块都离不开socket。



# TCP协议简述

## 流式传输

TCP协议是一种基于字节流的形式，什么叫流呢？因为数据的传输就像是水龙头一样打开哗啦啦的没有确切的边界，如下图所示：

![image-20210627145205398](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210627145205398.png)



## 三次握手

Client端和Server端若想正常进行数据交互，必须先经历一次三次握手的过程，用于确定二者关系并建立双向链接通道：

![image-20210627151523421](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210627151523421.png)

状态释义，可通过netstat命令查看：

- SYN_SENT：Client端发送一次建立链接请求后会立刻进入该状态，并且在没有收到Server端的回应之前都会保持该状态，通常来说该状态持续时间非常短，几乎不可测
- ESTABLISHED：当某一方进入该状态后，则代表可以向另一方发送数据了
- LISTEN：Server端在等待Client端建立三次握手时的状态，即监听Client端的请求
- SYN_RCVD：Server端进入该状态代表已收到Client端的三次握手链接请求

信息释义：

- SYN：用于建立链接的标志位
- ACK：确认请求的标志位
- seq：可以理解为一段暗号，用于确认该信息未被修改



补充，SYN泛洪攻击：

- 当Server端长期进入SYN_RCVD状态时就要当心是否遭受了SYN洪水攻击。
- 因为TCP三次握手对于Server端来讲会无限的回复Client端发来的SYN请求，收到一条就回一条。
- 如果有黑客模拟成千上万台Client端对Server端发起SYN请求，在发送第一次握手后就溜溜球了那么服务器还傻乎乎的等待第三次的握手回信，这么做会让Server端的压力很大。所以TCP协议也被称为好人协议...



补充，半链接池backlog：

- Server端如果不能一时之间全部处理完成所有的请求时会将后到的请求放入半链接池中进行排队
- 防止SYN 洪水攻击的有效策略其中一点就是：增大backlog链接池的最大数量，但是一般不会采用该策略
- 另一个策略是缩小Server端对每个请求的返回次数（如果Server端发现Client端没理自己，就会不断的回应上次的信息。初始值为5s，过5s发一次，然后变成3s，再过3s发一次，变成1s，再发一次...直到不想发了就不会理睬这个请求了）
- 平常打开一个网页打不开的时候，有一种可能性就是人家的backlog满了，你就只能排在外边儿等



## 双向链接通道

当Client端和Server端三次握手完成后，TCP协议会创建一个双向链接通道，用于Client端和Server端之间的数据交互，如图所示：

![image-20210627152355048](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210627152355048.png)





## 可靠传输

TCP协议也被称为可靠传输协议，即发送方向接收方发送了数据后，接收方必须回应发送方一个收到了的信息的ACK确认，若发送方没收到该ACK确认，则会重新发送一次数据，如下图所示。

注意：三次握手时的数据交互并不是走双向链接通道，而对于下图的数据传输来说则是走的双向链接通道了

![image-20210627152601873](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210627152601873.png)



## 四次挥手

当Client端要断开与Server端的链接时，必须要经历一个四次挥手的过程。

为什么创建链接仅需要三次，而断开链接则需要四次呢？

可以看到，三次握手之前是没有数据传输的，并且其中第二次是一次性发送了一个请求和一个确认。所以减少了一次操作。

而四次挥手涉及到数据的传输，所以不可能简化成三次挥手。

此外，四次挥手也是不同于三次握手，四次挥手也是建立在双向链接通道的基础之上的，而三次握手的时候该双向通道还未建立成功：

![image-20210627153359574](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210627153359574.png)



状态释义：

- FIN_WAIT_1：代表主动发起断开链接请求
- FIN_WAIT_2：代表此时的Client端不会再主动向Server端发送数据
- TIME_WAIT：代表Client端还要回复最后一条确认消息，回复完毕后双向链接正式关闭
- CLOSE_WAIT：代表关闭等待
- LAST_ACK：代表持续的确认（即只要Client端没有回复第4条信息，Server端就不断尝试发送断开链接的FIN请求）



另外，在实际生活场景中，服务端主动断开链接的情况比较多，因为它涉及到了和很多客户端的通信，还有的客户端还在排队，所以不可能对一个客户端浪费太多时间。

这句话你可以理解为：

- 服务器是个渣男 ，很多女孩子（Client端）都喜欢他，都给他写情书，他回复完了一个女孩子的情书后立马会拆开下一封情书，并不会只留恋于一封。

# UDP协议简述

## 数据报传输

UDP协议是一种基于数据报的格式（也被称为基于消息的传输），不同于TCP的字节流格式。UDP的数据报格式是有头有尾的，这一点很重要。

如图所示：

![image-20210627154335835](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210627154335835.png)





## 不可靠传输

UDP协议在数据传输时并不需要建立双向链接通道。

且UDP发消息与TCP不太一样，发送发只管发送消息，不管接收方有没有收到消息都不会再发，所以这也是UDP协议被称为不可靠传输协议的由来。

因为UDP协议没有这种ACK确认的机制，虽然对于数据可靠性来说下降了不少但是对于数据传输性上则有了明显的提示。

故DHCP服务以及DNS域名解析服务都是使用UDP协议，因为它速度更快，此外早起的QQ也是使用的UDP协议进行通信。

# 套接字发展史

## 套接字起源

套接字起源于20世纪70年代加利福尼亚大学伯克利分校版本的Unix，它最初的设计是为了让同一台主机上的多个应用程序之间进行通信，也就是进程通信或者被称为IPC，套接字有两种：

- 基于文件的套接字
- 基于网络的套接字

同机的不同进程之间本身是不允许通信的，但是可以通过套接字来进行数据交互。

此外，套接字也允许应用程序将I/O（input / output）插入到网络中，并与网络中的其他应用程序进行通信，基于网络的套接字就是IP地址加端口的组合。

故socket也被称为：IP+PORT。

## 基于文件的套接字家族



名称：AF_UNIX

作用：Unix一切皆文件，基于文件的套接字调用的就是底层的文件系统来存取数据，两个套接字进程运行在同一台机器上，可以通过访问同一个文件系统间接完成通信。



## 基于网络的套接字家族

名称：AF_INET

作用：IPV4协议套接字，有了IP + PORT我们可以与互联网上的任何应用程序进行通信。

除此之外还有一个叫AF_INET6的套接字，也就是基于IPV6的套接字。

# 套接字工作流程

## 基于TCP协议的套接字工作流程

由于TCP协议本身比较复杂，故使用基于TCP协议的套接字编写程序整体流程也较为复杂：

![image-20200626175328466](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/1881426-20200627145212675-1294046456.png)

## 基于UDP协议的套接字工作流程

基于UDP协议的套接字工作流程相比于基于TCP协议的套接字工作流程来说简单一些，因为不用建立双向链接通道：

![image-20200626175733838](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/1881426-20200627145212187-687025277.png)





# TCP/Socket简单通信

## 基础实现

由于TCP协议需要建立双向链接通道，故必须先开启Server端后再开启Client端，否则会发生异常。

 Server端代码如下：

```
from socket import *

# 1. 实例化socket对象，并添加端口复用
# AF_INET：IPV4
# SOCK_STREAM：TCP协议
server = socket(family=AF_INET, type=SOCK_STREAM)
server.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)

# 2.绑定IP地址与端口号
# localhost：仅允许本机使用
# 127.0.0.1：仅允许本机使用
# 0.0.0.0：允许任意client链接
server.bind(("localhost", 8888))

# 3.设置半链接池大小
server.listen(5)

# 4.阻塞等待三次握手请求
# conn：双向链接通道
# clientAddr：客户端链接信息
conn, clientAddr = server.accept()

# 5.接收client发送的信息，单位：字节
data = conn.recv(1024)

# 6.给client发送信息
conn.send(data.upper())

# 7.关闭双向链接通道，释放底层端口占用的系统资源
conn.close()

# 8.关闭服务器，释放Python应用程序所占据的内存资源
server.close()
```

Client端代码如下：

```
from socket import *

# 1. 实例化socket对象
# AF_INET：IPV4
# SOCK_STREAM：TCP协议
client = socket(family=AF_INET, type=SOCK_STREAM)

# 2.向服务端发送请求
# 开始进行三次握手并创建双向链接通道
client.connect(("localhost", 8888))

# 3.通信开始，发送消息
client.send("hello world".encode("u8"))

# 4.接收server端的消息
msg = client.recv(1024)
print(msg.decode("u8"))

# 5.关闭客户端
client.close()
```



## 双层循环

上面的例子中，Server端会接受Client端的信息，并将其进行upper()后返回。

也就是说Server端和Client端的交互仅有一次，这显然不符合常理。

所以我们需要为Server端的代码做一些小小的改动，让其能够不断的处理Client端的请求，而非只处理一次。

具体逻辑是：

- 增加链接循环，接收不同Client端的三次握手请求
- 增加通信循环，让Client端和Server端能够长时间通信

Server端代码如下所示：

```
from socket import *

server = socket(family=AF_INET, type=SOCK_STREAM)
server.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)

server.bind(("localhost", 8888))

server.listen(5)

while 1:
    # 不断接受新请求， 即服务端永不停止运（链接循环）
    conn, clientAddr = server.accept()
    print("%s connect server" % clientAddr[0])

    # 服务端能一直保持和客户端的通信，当客户端输入exit后将停止对当前客户端的服务（通信循环）
    while 1:
        data = conn.recv(1024)

        if data.decode("u8") == "exit":
            break

        conn.send(data.upper())

    print("%s close connect" % clientAddr[0])
    conn.close()
```

Client端代码如下所示：

```
from socket import *

client = socket(family=AF_INET, type=SOCK_STREAM)

client.connect(("localhost", 8888))

# 不断的与服务器进行交互
while 1:
    sendMsg = input(">>>")
    client.send(sendMsg.encode("u8"))

    # 如果发送的是exit，则断开链接
    if sendMsg == "exit":
        break

    recvMsg = client.recv(1024)
    print(recvMsg.decode("u8"))

print("client close")
client.close()
```



## Client端发送空信息导致的异常

现在Server端已经全部做好了，但是Client端还有一个问题。

当Client端出现 >>> 时直接敲击回车后会Server端会卡住，只有重启Server端才能解决该问题。



若进行代码调试，可观察到此时Client端处于recv()状态，而Server端也处于recv()状态，这代表Client端发送的回车“空消息”根本没有被Server端所接收到。

为什么会出现这样的情况？还需要从底层原理说起。



其实不管是send()还是recv()都是socket应用程序对操作系统发出的一次系统调用，再此期间CPU工作状态会从用户态转变至内核态。

而用户态的内存数据是不能直接与内核态的内存数据发生交互的，所以只能靠一种映射关系（可以理解为拷贝，但是并不准确）来映射出需要发送的内容。

如果Client端输入一个回车，且发送给了Server端后，Server端是接收不到该信息的，因为recv()的映射是读取不到“空消息”的：

- Client端将自己的回车“空消息”进行发送
- Server端的recv()由于读取不到“空消息”，故会直接卡住

如下图所示：

![image-20210628115821081](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210628115821081.png)

如果要解决这个问题，我们只需对Client端发送的消息做出限制，让其不为空即可：

```
from socket import *

client = socket(family=AF_INET, type=SOCK_STREAM)

client.connect(("localhost", 8888))

# 不断的与服务器进行交互
while 1:
    sendMsg = input(">>>")

    # 不让客户端发送空消息
    if not sendMsg:
        continue
        
    client.send(sendMsg.encode("u8"))
    recvMsg = client.recv(1024)
    print(recvMsg.decode("u8"))

    # 如果发送的是exit，则断开链接
    if sendMsg == "exit":
        break

print("client close")
client.close()
```



## 强行关闭Client端导致的异常

当Server端正在与Client端链接时，如果此时强行关闭Client端，将会导致Server端出现异常。

- Windows平台下Server端会直接抛出ConnectionResetError异常
- Unix平台下Server端的recv()会无限制收到空信息

如下所示：

```
ConnectionResetError: [WinError 10054] 远程主机强迫关闭了一个现有的连接。
```

为什么会出现这种原因呢？因为Server端和Client端的链接是双向的，一旦一方关闭链接通道后这个链接通道就会崩塌，从而导致Server端发生此异常。

如何解决该异常？需要用到try和except以及if判断，如下所示：

```
from socket import *

server = socket(family=AF_INET, type=SOCK_STREAM)
server.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)

server.bind(("localhost", 8888))

server.listen(5)

while 1:

    conn, clientAddr = server.accept()
    print("%s connect server" % clientAddr[0])

    while 1:

        # try： 针对Windows环境
        try:

            data = conn.recv(1024)
            
            # if：针对Unix环境
            if not data:
                break

            conn.send(data.upper())

        except ConnectionResetError as e:
            break

    print("%s close connect" % clientAddr[0])
    conn.close()
```





## 最终代码

最终代码如下。

Server端：

```
from socket import *

server = socket(family=AF_INET, type=SOCK_STREAM)
server.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)

server.bind(("localhost", 8888))

server.listen(5)

while 1:

    conn, clientAddr = server.accept()
    print("%s connect server" % clientAddr[0])

    while 1:

        # try： 针对Windows环境
        try:

            data = conn.recv(1024)
            
            # if：针对Unix环境
            if not data:
                break

            conn.send(data.upper())

        except ConnectionResetError as e:
            break

    print("%s close connect" % clientAddr[0])
    conn.close()
```



Client端：

```
from socket import *

client = socket(family=AF_INET, type=SOCK_STREAM)

client.connect(("localhost", 8888))

# 不断的与服务器进行交互
while 1:
    sendMsg = input(">>>")

    # 不让客户端发送空消息
    if not sendMsg:
        continue

    # 如果发送的是exit，则断开链接
    if sendMsg == "exit":
        break

    client.send(sendMsg.encode("u8"))
    recvMsg = client.recv(1024)
    print(recvMsg.decode("u8"))


print("client close")
client.close()
```



# UDP/Socket简单通信

## 基础实现

下面是基于UDP协议的socket通信。由于UDP协议是没有双向链接通道的，故首先启动任意一端都不会报错。

Server端代码如下：

```
from socket import *

# 1. 实例化socket对象，并添加端口复用
# AF_INET：IPV4
# SOCK_DGRAM：UDP协议
server = socket(family=AF_INET, type=SOCK_DGRAM)
server.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)

# 2.绑定IP地址与端口号
# localhost：仅允许本机使用
# 127.0.0.1：仅允许本机使用
# 0.0.0.0：允许任意client链接
server.bind(("localhost", 8888))

# 3.接收client端的数据
data, clientAddr = server.recvfrom(1024)

# 4.回复client的信息
server.sendto(data.upper(), clientAddr)

# 5.关闭服务器，释放Python应用程序所占据的内存资源
server.close()
```

Client端代码如下：

```
from socket import *

# 1. 实例化socket对象
# AF_INET：IPV4
# SOCK_DGRAM：UDP协议
client = socket(family=AF_INET, type=SOCK_DGRAM)

# 2.向服务端发送数据
client.sendto("hello world".encode("u8"), ("localhost", 8888))

# 3.接收服务端的数据
msg, serverAddr = client.recvfrom(1024)
print(msg.decode("u8"))

# 4.关闭客户端
client.close()
```



## 增加单层循环

由于基于UDP协议通信不会建立双向链接通道，所以我们只需要增加一个“通信循环”即可。

Server端改进代码如下所示：

```
from socket import *

server = socket(family=AF_INET, type=SOCK_DGRAM)
server.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)

server.bind(("localhost", 8888))

while 1:
    data, clientAddr = server.recvfrom(1024)
    server.sendto(data.upper(), clientAddr)
```

Client端改进代码如下所示：

```
from socket import *

client = socket(family=AF_INET, type=SOCK_DGRAM)

while 1:
    sendMsg = input(">>>")

    if sendMsg == "exit":
        break

    client.sendto(sendMsg.encode("u8"), ("localhost", 8888))

    msg, serverAddr = client.recvfrom(1024)
    print(msg.decode("u8"))

client.close()
```



## Bug测试

我们对该两段代码进行BUG测试均未发现异常。

1）强制停止Client端是否会导致Server端异常崩溃？

- 没有导致，原因是因为UDP协议的通信不基于双向链接通道

2）客户端发送回车或者任意空消息是否会导致recvfrom()卡住？

- 没有导致，这个还是要从UDP的数据格式说起，因为UDP是数据报格式的发送，所以即便消息体是空，也还有一个消息头在里面

    所以UDP的整段数据是不可能为空的，也就不会导致内核缓冲区读不到数据而卡住



# 解决端口占用问题

在进行socket编程中肯定会遇到端口被占用的情况，实际上就是服务器再向客户端发送最后一条ACK回应，也就是四次挥手中的第四步。

此时服务器的状态应该处于：TIME_WAIT（等待一段时间确保双向链接通道中的信息全部读取完毕）。这是属于正常情况，请勿惊慌。

解决方式如下：

1）加入一条socket配置，重用ip和端口：

```
# 这条代码放在bind的前面

server = socket(family=AF_INET, type=SOCK_STREAM)
server.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)  # *
server.bind(("localhost", 8888))
```

2）针对Linux平台：

```
发现系统存在大量TIME_WAIT状态的连接，通过调整linux内核参数解决，
vi / etc / sysctl.conf

编辑文件，加入以下内容：
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_fin_timeout = 30

然后执行 / sbin / sysctl - p
让参数生效。

net.ipv4.tcp_syncookies = 1
表示开启SYN
Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；

net.ipv4.tcp_tw_reuse = 1
表示开启重用。允许将TIME - WAIT
sockets重新用于新的TCP连接，默认为0，表示关闭；

net.ipv4.tcp_tw_recycle = 1
表示开启TCP连接中TIME - WAIT
sockets的快速回收，默认为0，表示关闭。

net.ipv4.tcp_fin_timeout
修改系統默认的
TIMEOUT
时间
```



# 验证链接合法性

在很多时候，我们的TCP服务端为了防止网络泛洪可以设置一个客户端链接的验证机制。

这个验证机制的实现其实也是非常简单，思路在于进入通信循环之前，客户端和服务端先走一次链接认证，只有通过认证的客户端才能够继续和服务端进行链接，核心点在于Server端和Client端都必须具有1个相同的验证对比盐值。

实现如图所示：

![image-20210628223106797](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210628223106797.png)

Server端代码如下：

```
from socket import *
import json
import os
import hmac


class TcpServer:
    def __init__(self, ip_port=("localhost", 8888), backlog=5, bufsize=1024) -> None:
        self.salt = b"SOCKET SERVER"
        self.verificationString = None

        self.bufsize = bufsize
        self.backlog = backlog
        self.ip_port = ip_port
        self.socket = None
        self.conn = None
        self.clientAddr = None

    def run(self):
        self.initialization()
        while 1:
            self.requestHandler()

            if not self.verification():
                print("verification fail")
                continue

            while 1:
                try:
                    self.communicateHandler()

                except ConnectionResetError as e:
                    self.conn.close()
                    break

    def initialization(self):
        """初始化数据"""
        self.socket = socket(family=AF_INET, type=SOCK_STREAM)
        self.socket.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)
        self.socket.bind(self.ip_port)
        self.socket.listen(5)
        self.socket.listen(self.backlog)

    def requestHandler(self):
        """处理链接请求"""
        self.conn, self.clientAddr = self.socket.accept()

    def verification(self):
        """验证链接合法性"""

        # 生成32位bytes类型的随机值
        self.verificationString = os.urandom(32)

        # 将随机值发送给client端
        self.conn.send(self.verificationString)

        # 接收client端的信息
        recvMsg = self.conn.recv(self.bufsize)

        # 将随机值和盐进行hash加密，与recvMsg进行对比，如果一样则验证通过
        # 否则认证失败，关闭双向通道
        h = hmac.new(self.salt, self.verificationString)
        digest = h.digest()

        print(digest, "\n", recvMsg)
        return hmac.compare_digest(digest, recvMsg)

    def communicateHandler(self):
        """处理通信请求"""

        data = self.conn.recv(self.bufsize)

        if not data:
            raise ConnectionResetError(
                "client %s close" % str(self.clientAddr))

        data = json.loads(data.decode("u8"))
        self.conn.send(
            json.dumps(data).encode("u8")
        )


if __name__ == "__main__":
    server = TcpServer()
    server.run()
```

Client端代码如下：

```
import json
import hmac

from socket import *


class TcpClient:
    def __init__(self, server_ip_port=("localhost", 8888), backlog=5, bufsize=1024) -> None:
        self.salt = b"SOCKET SERVER"
        self.bufsize = bufsize
        self.backlog = backlog
        self.server_ip_port = server_ip_port

        self.socket = None

    def run(self):
        """初始化数据"""
        self.socket = socket()
        self.socket.connect(self.server_ip_port)
        self.verification()
        self.communicateHandler()

    def verification(self):
        """验证链接合法性"""

        # 获取server端发送的验证字符串
        recvVerificationString = self.socket.recv(self.bufsize)

        # 将验证字符串与本地的盐进行混合，得到新的结果
        h = hmac.new(self.salt, recvVerificationString)
        digest = h.digest()

        # 将新结果发送给server端做比对
        self.socket.send(digest)

    def communicateHandler(self):
        """处理通信请求"""
        while 1:
            sendMsg = input(">>>")

            # 不让客户端发送空消息
            if not sendMsg:
                continue

            # 如果发送的是exit，则断开链接
            if sendMsg == "exit":
                break

            self.socket.send(
                json.dumps(sendMsg).encode("u8")
            )

            recvMsg = json.loads(
                self.socket.recv(1024).decode("u8")
            )

            print(recvMsg)

        self.socket.close()

if __name__ == "__main__":
    client = TcpClient()
    client.run()
```



# socket模块方法大全

以下是socket模块所提供的方法：

| 方法                                 | 描述                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| s.bind()                             | 绑定地址（host, port）到套接字， 在AF_INET下,以元组（host, port）的形式表示地址 |
| s.listen()                           | 开始TCP监听。backlog指定在拒绝连接之前，操作系统可以挂起的最大连接数量。该值至少为1，大部分应用程序设为5就可以了 |
| s.accept()                           | 被动接受TCP客户端连接，(阻塞式)等待连接的到来                |
| s.connect()                          | 主动初始化TCP服务器连接。一般address的格式为元组（hostname, port），如果链接出错会抛出socket.error异常 |
| s.connect_ex()                       | connect()函数的扩展版本，出错时返回出错码，而不是抛出异常    |
| s.recv()                             | 接收TCP数据，数据以字符串形式返回，bufsize指定要接收的最大数据量。flag提供有关消息的其他信息，通常可以忽略 |
| s.send()                             | 发送TCP数据，将string中的数据发送到连接的套接字。返回值是要发送的字节数量，该数量可能小于string的字节大小 |
| s.sendall()                          | 完整发送TCP数据，将string中的数据发送到连接的套接字，但在返回之前会尝试发送所有数据。成功返回None，失败则抛出异常 |
| s.recvfrom()                         | 接收UDP数据，与recv()类似，但返回值是（data, address）。其中data是包含接收数据的字符串，address是发送数据的套接字地址 |
| s.sendto()                           | 发送UDP数据，将数据发送到套接字，address是形式为（ipaddr，port）的元组，指定接收方的ip地址和端口号。返回值是发送的字节数 |
| s.close()                            | 关闭套接字                                                   |
| s.getpeername()                      | 返回连接套接字的远程地址。返回值通常是元组（ipaddr, port）   |
| s.getsockname()                      | 返回套接字自己的地址。通常是一个元组(ipaddr, port)           |
| s.setsockopt(level,optname,value)    | 设置给定套接字选项的值                                       |
| s.getsockopt(level,optname[.buflen]) | 返回套接字选项的值                                           |
| s.settimeout(timeout)                | 设置链接超时时间，默认为None，即永不超时                     |
| s.gettimeout()                       | 返回当前超时期的值，单位是秒，如果没有设置超时期，则返回None |
| s.fileno()                           | 返回套接字的文件描述符                                       |
| s.setblocking(flag)                  | 如果flag为0，则将套接字设为非阻塞模式，否则将套接字设为阻塞模式（默认值）。非阻塞模式下，如果调用recv()没有发现任何数据，或send()调用无法立即发送数据，那么将引起socket.error异常 |
| s.makefile()                         | 创建一个与该套接字相关连的文件                               |

更多方法，参考官方文档：[点我跳转](https://docs.python.org/3.6/library/socket.html)
