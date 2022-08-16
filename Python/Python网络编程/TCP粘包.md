# TCP/Socket与subprocess

我们准备做一个可以在Client端远程执行Server端的shell命令并拿到其执行结果的程序，而涉及到网络通信就必然会使用到socket模块，此外还需要subprocess模块拿到命令执行结果。

关于传输层协议的选择我们采用TCP协议，因为它是可靠传输协议且一次传输的数据量要比UDP协议更大。

以下是Server端的代码实现：

```
import subprocess
from socket import *

# 默认直接实例化socket是IPV4 + TCP协议
server = socket()
server.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)

server.bind(("localhost", 8888))
server.listen(5)

while 1:
    conn, clientAddr = server.accept()
    print("%s connect server" % clientAddr[0])
    
    while 1:
        try:
            command = conn.recv(1024)
            if not command:
                break

            result = subprocess.Popen(
                args=command.decode("u8"),
                shell=True,
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE
            )

            # get success result
            successOut = result.stdout.read()

            # get error result
            errorOut = result.stderr.read()

            # type == bytes
            dataBody = successOut or errorOut

            conn.send(dataBody)
        except ConnectionResetError as e:
            break

    print("%s close connect" % clientAddr[0])
    conn.close()
```

以下是Client端代码的实现：

```
from socket import *

client = socket()
client.connect(("localhost", 8888))

while 1:
    command = input(">>>").strip()
    if not command:
        continue

    if command == "exit":
        break

    client.send(command.encode("u8"))
    dataBody = client.recv(1024)

    # windows server : decode("gbk")
    # unix server : decode("u8")
    print(dataBody.decode("u8"))

print("client close")
client.close()
```

使用测试，Client端输入命令：

```
>>>ls
__pycache__
socketClient.py
socketServer.py

>>>pwd
/Users/yunya/PythonProject

>>>cal
      六月 2021         
日 一 二 三 四 五 六  
       1  2  3  4  5  
 6  7  8  9 10 11 12  
13 14 15 16 17 18 19  
20 21 22 23 24 25 26  
27 28 29 30           
                      

>>>
```



# 粘包现象

上面的测试看起来一切都非常完美，但如果Client端输入一条结果很长的命令时会出现一次性读取不完的Bug，如下所示：

```
>>>info vim
File: *manpages*,  Node: vim,  Up: (dir)

VIM(1)                                                                  VIM(1)

NAME
       vim - Vi IMproved, a programmer's text editor

SYNOPSIS
       vim [options] [file ..]
       vim [options] -
       vim [options] -t tag
       vim [options] -q [errorfile]

       ex
       view
       gvim gview evim eview
       rvim rview rgvim rgview

DESCRIPTION
       Vim  is a text editor that is upwards compatible to Vi.  It can be used
       to edit all kinds of plain text.  It is especially useful  for  editing
       programs.

       There  are a lot of enhancements above Vi: multi level undo, multi win-
       dows and buffers, syntax highlighting, command line  editing,  filename
       completion,   on-line   help,   visual  selection,  etc..   See  ":help
       vi_diff.txt" for a summary of the differences between Vim and Vi.

       While running Vim a lot of help can be obtained from the  on-line  help
       system,  with the ":help" command.  See the ON-
       
>>>ls
LINE HELP section below.

       Most often Vim is started to edit a single file with the command

            vim file

       More generally Vim is started with:

            vim [options] [filelist]

       If the filelist is missing, the editor will start with an empty buffer.
       Otherwise  exactly  one out of the following four may be used to choose
       one or more files to be edited.

       file ..     A list of filenames.  The first one  will  be  the  current
                   file  and  read  into the buffer.  The cursor will be posi-
                   tioned on the first line of the buffer.  You can get to the
                   other  files with the ":next" command.  To edit a file that
                   starts with a dash, precede the filelist with "--".

       -           The file to edit is read from  stdin.   Commands  are  read
                   from stderr, which should be a tty.

       -t {tag}    The file to edit and the initial cursor position depends on
                   a 
>>>
```

可以看到，第一次命令是info vim，第二次命令是ls，但是ls显示的依旧是info vim命令的执行结果。

这种现象就被称之为粘包现象。



# 产生原因

为什么会产生粘包现象呢？其实这与TCP底层传输有关，我们知道TCP是流式传输协议，故消息没有确切的边界。

上述代码中每次的recv()仅读取1024个字节，当消息超过1024字节后就会发生一次性读取不完整个内核缓冲区的情况，此时第二次recv()的读取会接着上次读取的位置继续进行读取，如下图所示：

![image-20210628135621029](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210628135621029.png)

由于我们的recv()只是按照固定的1024去读取数据，那么一旦整体内核缓冲区中所存储的数据量大于1024个字节，就会产生粘包现象。

产生粘包现象的原因主要还是因为接收方不知道消息之间的界限，不知道一次性读取多少字节的数据所造成的。





# Nagle算法

基于TCP协议的socket通信有一个特点：

- 即一方的send()与另一方的recv()可以没有任何关系
- 比如一方send()三次，另一方recv()一次就可以将数据全部取出来。

TCP协议的发送方有一个特征，他会采用了Nagle算法来对数据进行组包，如果一次发送的数据量很小，比如第一次发送10个字节，第二次发送2个字节，第三次发送3个字节，他可能会将这15个字节的数据凑到一块发送出去。

这么做有一个弊端就是接收方想要将这个大的数据包按照发送方的发送次数精确无误的接收拆分成10 2 3必须要有发送方提供的拆包机制才行。

Server端代码如下所示：

```
from socket import *


ip_port = ("localhost", 8000)
back_log = 5

server = socket(AF_INET, SOCK_STREAM)
server.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)

server.bind(ip_port)
server.listen(back_log)

conn, addr = server.accept()

conn.send("ABCDEFGHJK".encode("utf-8"))  # 第一次发送是10Bytes的数据
conn.send("LM".encode("utf-8"))     # 第二次也是2Bytes的数据
conn.send("NOP".encode("utf-8"))  # 第三次是3Bytes的数据
```

Client端代码如下所示：

```
from socket import *


ip_port = ("localhost", 8000)
buffer_size = 1024

client = socket(AF_INET, SOCK_STREAM)
client.connect(ip_port)

data_1 = client.recv(buffer_size)
print("first send of data:", data_1.decode("utf-8"))

data_2 = client.recv(buffer_size)
print("second send of data:", data_2.decode("utf-8"))

data_3 = client.recv(buffer_size)
print("last send of data:", data_3.decode("utf-8"))
```

最终接受结果：

```
first send of data: ABCDEFGHJK
second send of data: LMNOP
last send of data: 
```

可以看见发送方Server分三次发送了15bytes的数据，而接收方Client仅用两个recv()就接收完毕了所有的数据。

由Nagle算法产生的组包，会有极大可能导致粘包现象的发生，故我们需要思考如何让接收方知道发送方每一次发送的数据大小并准确进行拆分。



# 手动拆分

如果我们手动拆分接收方的recv()读取大小呢？是否还会发生粘包现象？

改进的Client端代码如下所示：

```
from socket import *


ip_port = ("localhost", 8000)
buffer_size = 1024

client = socket(AF_INET, SOCK_STREAM)
client.connect(ip_port)

# 由于预先知道对面第一次发送的数据包大小为10bytes，故这里也用10bytes进行读取
data_1 = client.recv(10)
print("first send of data:", data_1.decode("utf-8"))

# 由于预先知道对面第一次发送的数据包大小为2bytes，故这里也用2bytes进行读取
data_2 = client.recv(2)
print("second send of data:", data_2.decode("utf-8"))

# 由于预先知道对面第一次发送的数据包大小为3bytes，故这里也用3bytes进行读取
data_3 = client.recv(3)
print("last send of data:", data_3.decode("utf-8"))
```

最终接收结果：

```
first send of data: ABCDEFGHJK
second send of data: LM
last send of data: NOP
```

粘包被我们手动的计算字节数来精确的分割数据接受量的大小给解决了，但是这样做是不现实的，我们不可能知道对方发送的数据到底是怎么样的，更不用说手动计算。

所以有没有更好的解决方案呢？



# 预先发送消息长度

好了，其实上面关于解决粘包的思路已经出来了。

我们需要做的就是让接收方知道本次需要接收内容的大小才能够精确的将所有数据全部提取出来不产生遗漏。

实现方式很简单，可以尝试以下思路：

1. 发送方发送一个此次数据固定的长度
2. 接收方接收到该数据长度并且回应
3. 发送方收到回应并且发送真正的数据
4. 接收方不断的用默认的buffer_size值接收新的数据并存储起来直到超出整个数据的长度，代表此次数据全部接收完毕

实现代码如下，Server端：

```
import subprocess
from socket import *

# 默认直接实例化socket是IPV4 + TCP协议
server = socket()
server.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)

server.bind(("localhost", 8888))
server.listen(5)

while 1:
    conn, clientAddr = server.accept()
    print("%s connect server" % clientAddr[0])

    while 1:
        try:
            command = conn.recv(1024)

            if not command:
                break

            result = subprocess.Popen(
                args=command.decode("u8"),
                shell=True,
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE
            )

            # get success result
            successOut = result.stdout.read()

            # get error result
            errorOut = result.stderr.read()

            # type == bytes
            dataBody = successOut or errorOut

            # this is message head
            # tell client next receive of buffsize how many
            dataHead = len(dataBody)
            conn.send(str(dataHead).encode("u8"))

            # server receive result is ready
            # representative client it is already possible to receive real data body
            if conn.recv(1024) == b"ready":
                conn.send(dataBody)

        except ConnectionResetError as e:
            break

    print("%s close connect" % clientAddr[0])
    conn.close()
```



Client端：

```
from socket import *

client = socket()
client.connect(("localhost", 8888))

while 1:
    command = input(">>>").strip()
    if not command:
        continue

    if command == "exit":
        break

    client.send(command.encode("u8"))
    dataHead = client.recv(1024)
    dataBodyLength = int(dataHead.decode("u8"))
    currentRecvDataBodyLength = 0
    dataBody = b""

    # Can receive data body
    client.send(b"ready")
    while currentRecvDataBodyLength < dataBodyLength:
        currentRecvResult = client.recv(1024)
        dataBody += currentRecvResult
        currentRecvDataBodyLength += len(currentRecvResult)

    # all data body receive end
    else:
        print(dataBody.decode("u8"))


print("client close")
client.close()
```

经过实测后发现输入任何命令都不会发生粘包了。



# json+struct增加消息头

上面的解决方案还是有一些弊端，因为Server端是发送了2次send()，第1次发送数据整体长度，第2次发送数据内容主体，这样是不太好的（Server端可能同时处理多个链接，所以send()次数越少越好）。

而且如果Server端传的是一个文件的话那么局限性就太强了。因为我们只能将整体的消息长度发送过去而诸如文件名，文件大小之内的信息就发送不过去。

所以我们需要一个更加完美的解决方案，即Server端发送一次send()就将本次的数据整体长度发送过去（还可以包括文件姓名，文件大小等信息）。

那么这里就要使用到一个内置模块，struct。

struct模块可以将其某一种数据格式序列化为固定长度的bytes类型，其中最重要的两个方法就是pack()、unpack()。

- pcak(fmt, \*args)：根据格式将数据转换为固定长度的bytes类型
- unpack(fmt, string)：根据格式将bytes类型转换为原本的数据

以下是常用fmt格式类型：

|      |                    |             |            |
| ---- | ------------------ | ----------- | ---------- |
| 格式 | C语言类型          | Python类型  | 字节数大小 |
| x    | 填充字节           | 没有值      |            |
| c    | char               | 字节长度为1 | 1          |
| b    | signed char        | 整数        | 1          |
| B    | unsigned char      | 整数        | 1          |
| ?    | _Bool              | bool        | 1          |
| h    | short              | 整数        | 2          |
| H    | unsigned short     | 整数        | 2          |
| i    | int                | 整数        | 4          |
| I    | unsigned int       | 整数        | 4          |
| l    | long               | 整数        | 4          |
| L    | unsigned long      | 整数        | 4          |
| q    | long long          | 整数        | 8          |
| Q    | unsigned long long | 整数        | 8          |
| n    | ssize_t            | 整数        |            |
| N    | size_t             | 整数        |            |
| f    | float              | 浮点数      | 4          |
| d    | double             | 浮点数      | 8          |
| s    | char[]             | 字节        |            |
| p    | char[]             | 字节        |            |
| P    | void *             | 整数        |            |

简单的使用示例，将数值转换为固定长度的bytes：

```
>>> import struct
>>> binInt = struct.pack("i", len("this is message"))
>>> binInt
b'\x0f\x00\x00\x00'
>>> struct.unpack("i", binInt)
(15,)
>>> 
```

现在利用该struct模块 + json模块，我们就可以完美的解决粘包现象。

Server端代码如下：

```
import subprocess
import json
import struct

from socket import *

# 默认直接实例化socket是IPV4 + TCP协议
server = socket()
server.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)

server.bind(("localhost", 8888))
server.listen(5)

while 1:
    conn, clientAddr = server.accept()
    print("%s connect server" % clientAddr[0])

    while 1:
        try:
            command = conn.recv(1024)

            if not command:
                break

            result = subprocess.Popen(
                args=command.decode("u8"),
                shell=True,
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE
            )

            # get success result
            successOut = result.stdout.read()

            # get error result
            errorOut = result.stderr.read()

            # type == bytes
            dataBody = successOut or errorOut

            # 如果请求的是文件，还可以添加诸如 fileName fileSize
            # 等属性
            dataHeadDict = {
                "dataBodyLength": len(dataBody),
                "dataBodyType": str(type(dataBody)),
            }

            # 将字典序列化为字节
            dataHead = json.dumps(dataHeadDict).encode("u8")
            dataHeadLength = struct.pack("i", len(dataHead))

            # 发送数据头长度（固定为4字节）， 数据头字典，数据体信息
            conn.send(
                dataHeadLength + \
                dataHead + \
                dataBody
            )


        except ConnectionResetError as e:
            break

    print("%s close connect" % clientAddr[0])
    conn.close()
```

Client端代码如下：

```
import json
import struct

from socket import *

client = socket()
client.connect(("localhost", 8888))

while 1:
    command = input(">>>").strip()
    if not command:
        continue

    if command == "exit":
        break


    client.send(command.encode("u8"))

    # step01：接收数据头长度，也就是数据头字典的bufsize
    dataHeadLength = struct.unpack("i", client.recv(4))[0]

    # step02：提取数据头字典
    dataHeadDict = json.loads(client.recv(dataHeadLength))

    # step03：提取数据体长度和数据体类型
    dataBodyLength, dataBodyType = dataHeadDict.values()

    # step04：提取数据体
    dataBody = b""
    currentRecvDataBodyLength = 0
    while currentRecvDataBodyLength < dataBodyLength:
        currentRecvResult = client.recv(1024)
        dataBody += currentRecvResult
        currentRecvDataBodyLength += len(currentRecvResult)

    else:
        print(dataBody.decode("u8"))


print("client close")
client.close()
```

# 通过占位符拆分消息

规定一个特定的占位符作为消息的结束符。这是一个简单的解决方案，总体来说并不如上述方案优秀。

Server 端代码如下：

```
import subprocess
from socket import *

server = socket()
server.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)

server.bind(("localhost", 8888))
server.listen(5)

placeholder = "{{$E$}}"


def recv_msg(conn, size=1024) -> str:
    data = bytes.decode(conn.recv(size), "utf-8")

    # 客户端强制退出，会无限发送 ""
    if not data:
        return ""

    while not str.endswith(data, placeholder):
        data += bytes.decode(conn.recv(size), "utf-8")
    return data[:-len(placeholder)]


def send_msg(conn, msg: str) -> None:
    conn.send(str.encode(msg + placeholder, "utf-8"))


while 1:
    conn, client_addr = server.accept()
    print("%s connect server" % client_addr[0])

    while 1:
        try:
            command = recv_msg(conn, 1024)

            if not command:
                break

            result = subprocess.Popen(
                args=command,
                shell=True,
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE
            )

            success_out = result.stdout.read()

            error_out = result.stderr.read()

            send_msg(conn, bytes(success_out or error_out).decode("utf-8"))

        except ConnectionResetError as e:
            break

    print("%s close connect" % client_addr[0])
    conn.close()
```

Client端代码如下：

```
from socket import *

client = socket()
client.connect(("localhost", 8888))

placeholder = "{{$E$}}"


def recv_msg(conn, size=1024) -> str:
    data = bytes.decode(conn.recv(size), "utf-8")
    while not str.endswith(data, placeholder):
        data += bytes.decode(conn.recv(size), "utf-8")
    return data[:-len(placeholder)]


def send_msg(conn, msg: str) -> None:
    conn.send(str.encode(msg + placeholder, "utf-8"))


while 1:
    command = input(">>>").strip()
    if not command:
        continue

    if command == "exit":
        break

    send_msg(client, command)

    print(recv_msg(client))


print("client close")
client.close()
```
