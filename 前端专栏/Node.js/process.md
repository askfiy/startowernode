# process

process对象是一个全局变量，同时它也是EventEmitter对象的实例，提供了当前Node.js进程的信息和操作方法。

由于process对象是EventEmitter对象的实例，故可以对其进行事件监听。



# 常用属性

## 属性一览

以下是process对象中常用的属性：

| 属性     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| title    | 返回进程名称，默认为node                                     |
| pid      | 返回进程pid号                                                |
| platform | 返回当前运行进程的操作系统名称                               |
| version  | 返回当前的Node.js版本号                                      |
| env      | 返回当前shell中所有的环境变量                                |
| execPath | 返回执行当前脚本的Node二进制文件的绝对路径                   |
| argv     | 返回从命令行接收到用户输入命令的数组，它包含了执行当前脚本的Node二进制文件的绝对路径以及当前被执行文件的绝对路径 |
| execArgv | 如果用户在命令行输入的命令携带了Node.js的内置参数，如--inspect时，那么--inspect将添加到该数组中 |



## argv

process.argv属性是一个比较常用的属性。

我们可以在命令行中输入一些参数，并且让JavaScript脚本进行接收。

如下所示：

```
$ npm start localhost:5700
```

脚本文件：

```
ts-node ./src/server.ts 127.0.0.1:5500
```

argv数组：

```
[
  '/Users/yunya/.nvm/versions/node/v14.17.5/bin/ts-node',
  '/Users/yunya/Project/nodeProject/src/server.ts',
  '127.0.0.1:5500'
]
```

它有什么作用呢？比如我们想由使用者来决定HTTP服务的port和hostname时，它就派上了用处：

```
import * as http from 'http';

const argv: string[] = process.argv.slice(2);
const hostname: string = argv.length ? argv[0].split(":")[0] : "localhost";
const port: number = argv.length ? Number(argv[0].split(":")[1]) : 3000;

const server: http.Server = http.createServer((request: http.IncomingMessage, response: http.ServerResponse): void => {
    response.setHeader('Content-Type', 'application/json');
    response.statusCode = 200;
    response.end(JSON.stringify(request.method === "GET" ? "GET OK!!!" : "POST OK!!!"));
});

server.listen(port, hostname, (): void => {
    console.log(`server running success! http://${hostname}:${port}`);
})
```



# 常用方法

## 方法一览

以下是process对象中常用的方法：

| 方法          | 描述                          |
| ------------- | ----------------------------- |
| chidr()       | 切换工作目录到指定目录        |
| cwd()         | 返回当前工作目录              |
| exit()        | 退出当前进程                  |
| memoryUsage() | 返回Node.js进程的内存使用情况 |





# 进程事件

## exit

当Node.js进程因以下原因之一即将退出时，则会触发exit事件：

- 显式调用process.exit()方法
- Node.js事件循环不再需要执行任何其他工作

此时无法阻止退出事件循环，并且一旦所有exit事件的监听器都已完成运行时，Node.js进程将终止

```
process.on('exit', (code: number) => {
  console.log(`退出码: ${code}`);
});
```

## uncaughtException

当前进程抛出一个没有被捕捉的错误时，会触发uncaughtException事件：

```
process.on('uncaughtException', function (err: NodeJs.ErrnoException) {
  console.error(err.stack);
});
```

## beforeExit

当Node.js清空其事件循环并且没有其他工作要安排时，会触发beforeExit事件。通常Node.js进程将在没有调度工作时退出，但是在beforeExit事件上注册的监听器可以进行异步调用使Node.js进程继续：

```
process.on('beforeExit', (code: number) => {
  console.log('进程 beforeExit 事件的代码: ', code);
});

process.on('exit', (code: number) => {
  console.log('进程 exit 事件的代码: ', code);
});

console.log('此消息最新显示');

// 打印:
// 此消息最新显示
// 进程 beforeExit 事件的代码: 0
// 进程 exit 事件的代码: 0
```

## message

如果使用IPC通道fork Node.js进程，子进程收到父进程使用childprocess.send()发送的消息，就会触发message事件：

```
process.on('message', (m: string) => {
  console.log('子进程收到消息', m);
});
```





# process.nextTick(callback)

process.nextTick()方法将callback添加到下一个时间点的队列执行。

也就是说它会将一个任务提升为异步微任务。