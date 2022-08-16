# 基础介绍

## 什么是流

流是一种高效的信息传输手段，实际上早在几十年前的Unix操作系统中就已经存在了这一概念。

如下命令：

```
ls . | grep "test"
```

它有个最大的特点就能是对数据进行分段处理，第一个阶段该怎么做、第二个阶段该怎么做。

例如，在传统方式中，当告诉程序需要读取文件时，它会将文件从头到尾读入内存后再进行处理，而使用流的话则可以逐个分段的读取并处理数据，意味着不必一次读完所有数据后再进行处理。

一个通俗的例子，我们在线观看视频时，缓冲进度条总是随着当前观看进度不断的增加，它并不是一开始就立马请求完整个完整的视频后再进行播放，这就是一个非常典型的流式传输案例。

相较于使用其他的数据处理方法，流提供了2个主要优点：

- 内存效率：无需加载大量的数据到内存中即可进行处理
- 时间效率： 当获得数据之后即可立即开始处理数据，这样所需的时间更少，而不必等到整个数据有效负载可用才开始

Node.js 的 [stream模块](http://nodejs.cn/api/stream.html)提供了构建所有流 API 的基础。 所有的流都是[EventEmitter](http://nodejs.cn/api/events.html#events_class_eventemitter)的实例。

流可以分为4种类型：

- Readable：可读流，不可写入数据
- Writable：可写流，不可读取数据
- Duplex：双工流，可读可写
- Transform：类似于双工流、但其输出是其输入的转换的转换流



## 流的传输

流的数据传输必须依赖一个管道，例如，下面的示例将从可读流中读取数据，并通过管道将它交由HTTP客户端：

```
import * as http from 'http';
import * as fs from 'fs';

const hostname: string = "localhost";
const port: number = 3000;

const server: http.Server = http.createServer((request: http.IncomingMessage, response: http.ServerResponse): void => {
    console.log(request.url);
    response.setHeader('Content-Type', 'text/html; charset=utf-8');
    response.statusCode = 200;
    const stream = fs.createReadStream(__dirname + "/testFile.txt");
    stream.pipe(response);  // 通过管道发送给response
});

server.listen(port, hostname, (): void => {
    console.log(`server running success! http://${hostname}:${port}`);
})
```

这样做的好处就是如果要读取的文件太大，那么我们可以不必等待文件被完全读取后再进行返回，而是读取一部分返回一部分。

实际上pipe()方法可以进行链式操作，它将不断的对数据进行传输，如下所示：

```
src.pipe(dest1).pipe(dest2)
```

此构造相对于：

```javascript
src.pipe(dest1)
dest1.pipe(dest2)
```





# 可读流

## 基本创建

通过stream模块创建一个可读流，对其进行初始化的同时并实现\_read()方法即可。

this.push()方法用于为当前的可读流中新增数据，如下所示我们创建了一个不断产生随机数的可读流。

- this.push()的内容只能是字符串或者Buffer，不能是数字
- this.push()方法有第二参数encoding，用于第一个参数是字符串时指定encoding

如下所示：

```
import * as stream from "stream";

const readableStream: stream.Readable = new stream.Readable()
readableStream._read = function (): void {
    // 模拟数据的产生，每1s产生一个数据
    setTimeout(() => {
        this.push(String(Math.round(Math.random() * 20)) + "\n");
    }, 1000);
}

// 进行读取，将数据输出到stdout上
readableStream.pipe(process.stdout)
```

或者你也可以通过类继承的方式进行实现：

```
import * as stream from "stream";

class ReadableStream extends stream.Readable {
    constructor() {
        super();
    }
    _read(): void {
        setTimeout(() => {
            this.push(String(Math.round(Math.random() * 20)) + "\n");
        }, 1000);
    }
}

const readableStream = new ReadableStream();
readableStream.pipe(process.stdout);
```



## 停止读取

如何停止可读流呢？实际上只要this.push()一个null即可，如下所示只要随机数为20就停止读取：

```
import * as stream from "stream";

class ReadableStream extends stream.Readable {
    constructor() {
        super();
    }
    _read(): void {
        setTimeout(() => {
            let n: number = Math.round(Math.random() * 20);
            n != 20 ? this.push(String(n) + "\n") : this.push(null);
        }, 1000);
    }
}

const readableStream = new ReadableStream();
readableStream.pipe(process.stdout);
```



## 工作模式

细心的朋友可能会发现，我们的\_read()方法中关于随机数的生成使用了setTimeout()，而没有使用setInterval()，但是当readableStream.pipe()时依旧会源源不断的产生随机数，这是为什么呢？

其实流分为2种工作模式：

- 流动模式：由pipe()方法进行触发
- 暂停模式：由read()方法进行触发（默认的）

可以这么理解，当我们调用read()方法时，相当于把水龙头拧开之后马上关掉，每次只会流出一股水。

而当调用pipe()方法时，它内部会源源不断的调用readableStream._read()方法，相当于直接把水龙头拧开，直至蓄水池中的水被全部放掉后才会关闭水龙头，也就是说只有可读流不再产生新的数据后，pipe()方法才算运行完成，否则便会一直运行。



## 相关事件

我们可以对可读流进行事件监听：

- data事件：当可读流已经产生了数据时，将自动触发该事件并启用流动模式读取产生的数据
- end事件：数据已经读取完成
- error事件：数据读取失败了

如下示例演示：

```
import * as stream from "stream";

class ReadableStream extends stream.Readable {
    constructor() {
        super();
    }
    _read(): void {
        setTimeout(() => {
            let n: number = Math.round(Math.random() * 20);
            n != 20 ? this.push(String(n) + "\n") : this.push(null);
        }, 1000);
    }
}

const readableStream = new ReadableStream();

readableStream.on("data", (chunk: Buffer) => {
    // chunk = 本次被读取的数据
    console.log(chunk.toString());
})

readableStream.on("end", () => {
    console.log("读取完毕");
})

readableStream.on("error", () => {
    console.log("读取失败");
})
```

此外还有一个readable事件，它代表可读流已经产生了新的数据。

常用于和read()方法搭配使用，用于启用暂停模式的读取数据：

```
import * as stream from "stream";

class ReadableStream extends stream.Readable {
    constructor() {
        super();
    }
    _read(): void {
        setTimeout(() => {
            let n: number = Math.round(Math.random() * 20);
            n != 20 ? this.push(String(n) + "\n") : this.push(null);
        }, 1000);
    }
}

const readableStream = new ReadableStream();

readableStream.on("readable", () => {
    console.log("有数据了");
    let chunk: Buffer = readableStream.read();
    if(chunk !== null){
        console.log(chunk.toString());
    }
})
```





# 可写流

## 基本创建

通过stream模块创建一个可写流，对其进行初始化的同时并实现_write()方法即可。

在进行可写流初始化时，可指定options，它是一个对象，以下是常用的设置项：

- objectMode：Boolean类型，默认是 false， 设置成true后writable.write()方法除了写入string和buffer外，还可以写入任意 JavaScript对象。
- highWaterMark：Number类型，每次最多写入的数据量， Buffer的时候默认值16kb
- decodeStrings：Boolean类型，是否把传入的数据转成Buffer，默认是true

若想使用可写流，则需要调用write()方法，它指向了内部的\_write()，共接收3个参数：

- chunk：写入的数据，默认是Buffer类型
- encoding：如果数据是字符串，可以设置编码，buffer或者object模式会忽略
- done：回调函数

如下所示，我们往自己的可写流中写入数据，然后将数据再写入到process.stdout的可写流中：

```
import * as stream from "stream";

class WriteableStream extends stream.Writable {

    constructor(options?: { [key: string]: any }) {
        super(options);
    }

    _write(chunk: string | Buffer, encoding?: Buffer | string, done?: (err?: NodeJS.ErrnoException) => void): void {
        try {
            // Ts避免错误
            encoding;
            process.stdout.write(chunk.toString());
            if (done !== undefined) {
                done();
            }
        } catch (e) {
            if (done !== undefined) {
                done(e);
            }
        }
    }
}

const writeableStream = new WriteableStream({
    objectMode: false,
    highWaterMark: 16,
    decodeStrings: true
});

writeableStream.write("hello");

// buffer
// hello
```



## 停止写入

数据写入完成后，应当调用end()方法，表示数据已经写入完成了：

```
writeableStream.write("hello");
writeableStream.end();
```

它将会触发一系列的事件完成操作。



## 相关事件

我们可以对可写流进行事件监听：

- finish事件：数据已经写入完成
- error事件：数据写入失败了

如下示例演示：

```
import * as stream from "stream";

class WriteableStream extends stream.Writable {

    constructor(options?: { [key: string]: any }) {
        super(options);
    }

    _write(chunk: string | Buffer, encoding?: Buffer | string, done?: (err?: NodeJS.ErrnoException) => void): void {
        try {
            // Ts避免错误
            encoding;
            process.stdout.write(chunk.toString());
            if (done !== undefined) {
                done();
            }
        } catch (e) {
            if (done !== undefined) {
                done(e);
            }
        }
    }
}

const writeableStream = new WriteableStream({
    objectMode: false,
    highWaterMark: 16,
    decodeStrings: true
});

writeableStream.write("hello world");
writeableStream.end();

writeableStream.on("finish", () => {
    console.log("写入完成");
})

writeableStream.on("error", () => {
    console.log("写入失败");
})
```

