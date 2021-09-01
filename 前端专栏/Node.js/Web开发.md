# http模块

## 简单使用

Node.js中内置的http模块可用于搭建HTTP服务器。

如下所示：

```
import * as http from "http";

const hostname: string = "localhost";
const port: number = 3000;

const server = http.createServer((request: http.IncomingMessage, response: http.ServerResponse) => {
    response.setHeader('Content-Type', "text/http charset=utf-8");
    response.statusCode = 200;
    if (request.method === "GET") {
        response.end("/GET OK!");
    }
    else {
        response.end("/POST OK!");
    }
});

server.listen(port, hostname, 128, () => {
    console.log(`server running success! http://${hostname}:${port}`);
});
```

代码释义：

- http.createServer()：创建一个HTTP服务器，需要指定一个callback，它将在请求来临时自动调用，callback具有2个参数req和res分别代指请求对象和响应对象
- listen()：用于启动服务并监听指定端口，必须传入port、host，可选传入backlog和callback，callback会在服务器成功启动后自动调用





## 请求相关

以下是关于请求对象req的常用API，req是一个可读流的对象：

| api             | 描述                                            |
| --------------- | ----------------------------------------------- |
| req.headers     | 返回请求头对象                                  |
| req.method      | 返回本次请求的方式（GET、POST、DELETE、PUT 等） |
| req.url         | 返回本次请求的地址                              |
| req.httpVersion | 返回本次请求的HTTP版本                          |





## 响应相关

以下是关于请求对象响应对象res的常用API，res是一个可写流的对象：

| api                                                    | 描述                                                       |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| res.statusCode                                         | 用于设置响应的HTTP状态码                                   |
| res.setHeader(name, value)                             | 用于设置响应头                                             |
| res.writeHead(statusCode [, StatusMessage[, headers]]) | 发送响应首部，包含状态码、状态信息、响应头                 |
| res.getHeaders()                                       | 获取已设置的响应头副本                                     |
| res.getHeaderNames()                                   | 获取已设置的HTTP响应头名称的列表                           |
| res.getHeader(name)                                    | 获取已设置的响应头                                         |
| res.removeHeader(name)                                 | 删除已设置的响应头                                         |
| res.hasHeader(name)                                    | 如果响应已设置该消息头，则返回true                         |
| res.write(chunk)                                       | 用于向响应体中写入字符串或者buffer                         |
| res.end(chunk)                                         | 用于结束本次响应，若指定chunk则相当于同时调用了write()方法 |





## 返回文件

如下示例将演示如何返回一个普通的资源文件，如HTML、CSS、JS文件：

```
import * as http from "http";
import * as path from "path";
import * as fs from "fs";

const hostname: string = "localhost";
const port: number = 3000;

const baseDir: string = path.join(__dirname, "..");
const templatesDir: string = path.join(baseDir, "templates");

const server = http.createServer((request: http.IncomingMessage, response: http.ServerResponse) => {
    if (request.method === "GET") {
        response.writeHead(200, {
            "Content-Type": "text/html; charset=utf-8",
        });
        const indexPath: string = path.join(templatesDir, "index.html");
        fs.createReadStream(indexPath).pipe(response);
    } else {
        response.writeHead(405);
        response.end("Method Not Allowed");
    }
});

server.listen(port, hostname, 128, () => {
    console.log(`server running success! http://${hostname}:${port}`);
});
```



## 返回视频

如下示例将演示如何返回视频文件：

```
import * as http from "http";
import * as path from "path";
import * as fs from "fs";

const hostname: string = "localhost";
const port: number = 3000;

const baseDir: string = path.join(__dirname, "..");
const templatesDir: string = path.join(baseDir, "templates");
const staticsDir: string = path.join(baseDir, "static");


function index(request: http.IncomingMessage, response: http.ServerResponse) {
    console.log(request.httpVersion);
    response.writeHead(200, {
        "Content-Type": "text/html; charset=utf-8",
    });
    const indexPath: string = path.join(templatesDir, "index.html");
    fs.createReadStream(indexPath).pipe(response);
}

function getVideo(request: http.IncomingMessage, response: http.ServerResponse) {
    // 前端代码： <video src="/static/video/apple.mp4" controls height="500" Auto></video>
    // 与本地的staticDir路径做拼接，获取static目录下的/video/apple.mp4
    const videoPath = path.join(staticsDir, (request.url || "").split("/static/")[1]);
    const stat: fs.Stats = fs.statSync(videoPath);
    const fileSize = stat.size;
    const range: string = request.headers.range || "";
    if (range) {
        // 后续的请求视频，浏览器会自动携带range请求头，包含当前播放的部分
        const parts = range.replace(/bytes=/, "").split("-");
        const start = parseInt(parts[0], 10);
        const end = parts[1]
            ? parseInt(parts[1], 10)
            : fileSize - 1;
        // 计算新返回的视频流长度
        const chunkSize: number = (end - start) + 1;
        // 根据视频流长度进行截取
        const file: fs.ReadStream = fs.createReadStream(videoPath, { start, end });
        // 返回当前响应的视频流长度，并且请求状态要置为206
        response.writeHead(206, {
            'Content-Type': 'video/mp4',
            'Content-Range': `bytes ${start}-${end}/${fileSize}`,
            'Accept-Ranges': 'bytes',
            'Content-Length': chunkSize
        });
        file.pipe(response);
    } else {
        // 第一次请求视频，请求头只返回请求视频的总长度，和一些数据
        response.writeHead(200, {
            'Accept-Ranges': 'bytes',
            'Content-Length': fileSize,
        });
        fs.createReadStream(videoPath).pipe(response);
    }
}

const server = http.createServer((request: http.IncomingMessage, response: http.ServerResponse) => {
    if (request.method === "GET") {
        const reqUrl: string = request.url || "";
        switch (true) {
            case reqUrl === "/":
                index(request, (response));
                break
            case reqUrl.includes("video"):
                getVideo(request, response);
                break
            default:
                response.writeHead(404);
                response.end("Not Found");
        }
    } else {
        response.writeHead(405);
        response.end("Method Not Allowed");
    }
});

server.listen(port, hostname, 128, () => {
    console.log(`server running success! http://${hostname}:${port}`);
});
```



# url模块

## GET请求参数

推荐使用内置模块url对GET请求参数做出解析，如下所示，我们需要将request.url转成一个url对象，然后获取其中的URLSearchParams对象：

```
import * as http from "http";
import * as url from "url";

const hostname: string = "localhost";
const port: number = 3000;

const server = http.createServer((request: http.IncomingMessage, response: http.ServerResponse) => {
    if (request.method === "GET") {
        const urlIns: url.URL = new url.URL(request.url || "", `http://${hostname}:${port}`);
        const getParams: url.URLSearchParams = urlIns.searchParams;
        console.log(getParams);
        response.writeHead(200, {
            "Content-Type": "text/plain;"
        });
        response.end("ok");
    } else {
        response.writeHead(405);
        response.end("Method Not Allowed");
    }
});

server.listen(port, hostname, 128, () => {
    console.log(`server running success! http://${hostname}:${port}`);
});
```

以下是urlIns对象的结构：

```
# requestURL : http://localhost:3000/?getParams1=v1&getParams2=v2

URL {
  href: 'http://localhost:3000/?getParams1=v1&getParams2=v2',
  origin: 'http://localhost:3000',
  protocol: 'http:',
  username: '',
  password: '',
  host: 'localhost:3000',
  hostname: 'localhost',
  port: '3000',
  pathname: '/',
  search: '?getParams1=v1&getParams2=v2',
  searchParams: URLSearchParams { 'getParams1' => 'v1', 'getParams2' => 'v2' },
  hash: ''
}
```



## POST请求参数

注意！如果前端发送的是POST请求可能请求体的内容一次性接受不完，此时后端需要监听可读流request对象进行接收。

请求体接收完成之后直接调用url模块下的URLSearchParams方法得到一个参数对象：

```
import * as http from "http";
import * as url from "url";

const hostname: string = "localhost";
const port: number = 3000;

const server = http.createServer((request: http.IncomingMessage, response: http.ServerResponse) => {
    if (request.method === "POST") {
        let data: string = "";
        request.on("data", (chunk: Buffer) => {
            data += chunk.toString();
        })
        request.on("end", () => {
            const postParams: url.URLSearchParams = new url.URLSearchParams(data);
            console.log(postParams);
            response.writeHead(200, {
                "Content-Type": "text/plain;"
            });
            response.end("ok");
        })
    } else {
        response.writeHead(405);
        response.end("Method Not Allowed");
    }
});

server.listen(port, hostname, 128, () => {
    console.log(`server running success! http://${hostname}:${port}`);
});
```

以下是postParams对象的结构：

```
# requestURL : http://localhost:3000/
# requestBody : postParams1=v1 postParams2=v2

URLSearchParams { 'postParams1' => 'v1', 'postParams2' => 'v2' }
```



## 合并两者请求

有的时候在发送POST请求时，可能也会在地址栏中带入一些GET请求的参数。

此时我们就需要获取POST+GET的所有参数，可以将2个URLSearchParams对象进行合并，如下所示：

```
import * as http from "http";
import * as url from "url";

const hostname: string = "localhost";
const port: number = 3000;

const server = http.createServer((request: http.IncomingMessage, response: http.ServerResponse) => {
    let data: string = "";
    request.on("data", (chunk: Buffer) => {
        data += chunk.toString();
    })
    request.on("end", () => {
        const getParams: url.URLSearchParams = new url.URL(request.url || "", `http://${hostname}:${port}`).searchParams;
        const postParams: url.URLSearchParams = new url.URLSearchParams(data);
        const queryParams: { [key: string]: any } = {};

        getParams.forEach((v: string, k: string) => {
            queryParams[k] = v;
        })
        postParams.forEach((v: string, k: string) => {
            queryParams[k] = v;
        })

        console.log(queryParams);

        response.writeHead(200, {
            "Content-Type": "text/plain;"
        });
        response.end("ok");
    })
});

server.listen(port, hostname, 128, () => {
    console.log(`server running success! http://${hostname}:${port}`);
});
```

如下，发送一次POST请求，并在地址栏中携带参数：

```
# requestURL : http://localhost:3000/?getParams1=v1&getParams2=v2
# requestBody : postParams1=v1 postParams2=v2

{
  getParams1: 'v1',
  getParams2: 'v2',
  postParams1: 'v1',
  postParams2: 'v2'
}
```



# mime模块

## 基本使用

我们可以安装一个mime模块，用于动态的获取请求中的MIME类型，判断所请求的是静态资源还是动态资源。

也可以自动的书写响应头Content-Type：

```
$ npm install mime --save
$ npm install --save @types/mime  // 适用于TypeScript导包
```

示例演示：

```
import * as http from "http";
import * as url from "url";
import * as mime from "mime";

const hostname: string = "localhost";
const port: number = 3000;

const server = http.createServer((request: http.IncomingMessage, response: http.ServerResponse) => {
    // 获取请求的MIME类型
    let urlIns: url.URL = new url.URL(request.url || "", `http://${hostname}:${port}`);
    console.log(mime.getType(urlIns.pathname));
    let mimeType: string = mime.getType(urlIns.pathname) || "";
    response.writeHead(200, { "Content-Type": mimeType });
    response.end("ok");
});

server.listen(port, hostname, 128, () => {
    console.log(`server running success! http://${hostname}:${port}`);
});
```



## 常见mime类型

以下例举常见的mime类型：

|       类型/字类型        |               扩展名                |
| :----------------------: | :---------------------------------: |
|        text/plain        |        *.txt 或 其他文本文件        |
|        text/html         |           *.html 或 *.htm           |
|         text/css         |                *.css                |
|     text/javascript      |                *.js                 |
|     application/json     |               *.json                |
|        image/gif         |                *.gif                |
|        image/png         |                *.png                |
|        image/jpeg        |           *.jpg 或 *.jpeg           |
|        image/bmp         |                *.bmp                |
|        image/webp        |               *.webp                |
|      image/svg+xml       |            *.svg(矢量图)            |
|       image/x-icon       |                *.ico                |
|        audio/wav         |                *.wav                |
|        audio/webm        |               *.webm                |
|        audio/ogg         |                *.ogg                |
|        audio/mpeg        |                *.mp3                |
|        video/webm        |               *.webm                |
|        video/ogg         |                *.ogg                |
|        video/mp4         |                *.mp4                |
| application/octet-stream | .*（ 二进制流，不知道下载文件类型） |





# EJS模块

## 模板渲染

下载EJS模块，生成模板文件：

```
$ npm install ejs --save
$ npm install @types/ejs  // 适用于TypeScript导包
```

官方文档：[点我跳转](https://ejs.bootcss.com/#docs)

模板文件如下，注意模板文件用ejs进行结尾，在模板文件中你可以输入任何JavaScript代码：

```
// index.ejs

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <ul>
        <% unOrderList.forEach(value => { %>
            <li><%= value %></li>
        <% }) %>
    </ul>
    <video src="static/video/apple.mp4" controls height="500" Auto></video>
</body>

</html>
```

后端渲染的视图函数如下：

```
import * as http from "http";
import * as path from "path";
import * as ejs from "ejs";

const hostname: string = "localhost";
const port: number = 3000;

const baseDir: string = path.join(__dirname, "..");
const templatesDir: string = path.join(baseDir, "templates");

const server = http.createServer((request: http.IncomingMessage, response: http.ServerResponse) => {
    if (request.method === "GET") {
        response.writeHead(200, {
            "Content-Type": "text/html; charset=utf-8",
        });
        const indexPath: string = path.join(templatesDir, "index.ejs");
        ejs.renderFile(indexPath, { unOrderList: ["热门新闻", "热门评论", "实时资讯"] }, (err: Error | null, str: string) => {
            if (err) throw err;
            response.end(str);
        })
    } else {
        response.writeHead(405);
        response.end("Method Not Allowed");
    }
});

server.listen(port, hostname, 128, () => {
    console.log(`server running success! http://${hostname}:${port}`);
});
```



## 相关语法

脚本标签：

```
<% JavaScript代码 %>  # 它将作用在后端服务器上
```

输出标签：

```
<%= 变量 >
```

导入模板：

```
<%- include('模板') -%>
```

自定义分隔符：

```
// 单个模板文件，将 <% %> 改为 <? ?>
ejs.render('<?= user ?>', {user: 'Jack'}, {delimiter: '?'});

// 全局，将 <% %> 改为 <? ?>
ejs.delimiter = '$';

ejs.render('<?= user ?>', {user: 'Jack'});
```



# mysql模块

## 基本使用

如果想操纵mysql数据库，需要依赖第三方模块mysql：

```
$ npm install mysql --save
$ npm install @types/mysql --save  // 适用于TypeScript导包
```

然后我们需要链接数据库：

```
import * as mysql from "mysql";

const connection: mysql.Connection = mysql.createConnection({
    host: "localhost",
    user: "root",
    password: "",
    database: "db1"
})

connection.connect();
console.log("connection success");

connection.end();
console.log("close connection");
```



## 新增记录

示例如下，我们的数据库db1中有一张空的userInfo表，在此基础上做操作。

```
import * as mysql from "mysql";

const connection: mysql.Connection = mysql.createConnection({
    host: "localhost",
    user: "root",
    password: "",
    database: "db1"
})

connection.connect();
console.log("connection success");

// 使用 ? 进行占位
const sql: string = "INSERT INTO db1.userInfo (name, age, gender) VALUES (?, ?, ?);";
// 与 ? 一一对应，防止SQL注入
const params = ["Jack", "18", "male"];

connection.query(sql, params, (err: mysql.MysqlError | null, result: any) => {
    if (err) {
        console.log(err.message);
        throw err;
    }
    console.log(`Affected line: ${result.affectedRows}`);
    if (result.affectedRows > 0) {
        console.log(`Insert success, id: ${result.insertId}`);
    } else {
        console.log(`Insert fail`);
    }
})

connection.end();
console.log("close connection");
```



## 删除记录

删除记录示例：

```
import * as mysql from "mysql";

const connection: mysql.Connection = mysql.createConnection({
    host: "localhost",
    user: "root",
    password: "",
    database: "db1"
})

connection.connect();
console.log("connection success");

// 使用 ? 进行占位
const sql: string = "DELETE FROM db1.userInfo WHERE id=?";
// 与 ? 一一对应，防止SQL注入
const params = 1;

connection.query(sql, params, (err: mysql.MysqlError | null, result: any) => {
    if (err) {
        console.log(err.message);
        throw err;
    }
    console.log(`Affected line: ${result.affectedRows}`);
    if (result.affectedRows > 0) {
        console.log(`delete success`);
    } else {
        console.log(`delete fail`);
    }
})

connection.end();
console.log("close connection");
```



## 更新记录

更新记录示例，以下是原有的数据：

```
mysql> select * from userInfo;
+----+------+-----+--------+---------------------+
| id | name | age | gender | create_time         |
+----+------+-----+--------+---------------------+
|  1 | Jack |  18 | male   | 2021-08-30 17:23:44 |
|  2 | Mary |  21 | female | 2021-08-30 17:24:53 |
|  3 | Tom  |  17 | male   | 2021-08-30 17:24:53 |
+----+------+-----+--------+---------------------+
```

进行更新：

```
import * as mysql from "mysql";

const connection: mysql.Connection = mysql.createConnection({
    host: "localhost",
    user: "root",
    password: "",
    database: "db1"
})

connection.connect();
console.log("connection success");

// 使用 ? 进行占位
const sql: string = `UPDATE db1.userInfo SET name = ?, age = ? WHERE id = ?; `;
// 与 ? 一一对应，防止SQL注入
const params = ["Ken", 22, 1];

connection.query(sql, params, (err: mysql.MysqlError | null, result: any) => {
    if (err) {
        console.log(err.message);
        throw err;
    }
    console.log(`Affected line: ${result.affectedRows}`);
    if (result.affectedRows > 0) {
        console.log(`update success`);
    } else {
        console.log(`update fail`);
    }
})

connection.end();
console.log("close connection");
```



## 查询记录

查询记录示例：

```
import * as mysql from "mysql";

const connection: mysql.Connection = mysql.createConnection({
    host: "localhost",
    user: "root",
    password: "",
    database: "db1"
})

connection.connect();
console.log("connection success");

// 使用 ? 进行占位
const sql: string = `SELECT * FROM db1.userInfo WHERE id > ? `;
// 与 ? 一一对应，防止SQL注入
const params = 1;

connection.query(sql, params, (err: mysql.MysqlError | null, result: any) => {
    if (err) {
        console.log(err.message);
        throw err;
    }

    console.log(`select success! a total of ${result.length} records`);

    for (let row of result) {
        console.log(row.id, row.name, row.age, row.gender);
    }
})

connection.end();
console.log("close connection");
```



# 模块封装

## 项目前瞻

http模块最大的缺点是在路由系统以及静态文件管理系统上。

它只提供了最基本的HTTP服务器所需要的功能，并未对此进行更高层次的封装，所以在此我们可以对其进行更高程度的封装，着重以下3个方面：

- 路由系统：用户可自己定义路由匹配规则，并限制视图函数处理的请求方式
- 参数获取：用户可以直接从req.queryParams中获取GET请求以及POST请求所发送的所有参数
- 静态资源响应：自动响应所请求的静态资源，并且如果请求的是一个视频文件，那么它会自动使用流返回该视频文件

以下是整个项目的目录结构：

```
.
├── dist                      # 编译代码目录
│   ├── ...
├── node_modules
│   ├── ...
├── package-lock.json
├── package.json
├── src                       # 源代码目录
│   └── server.ts
├── common                    # 公用组件，自定义模块
│   ├── advancedHttp.ts        
├── static                    # 静态文件目录
│   ├── css
│   │   └── index.css
│   ├── img
│   │   └── avatar.jpg
│   ├── js
│   │   └── index.ts
│   └── video
│       └── apple.mp4
├── templates                # 模板文件目录
│   └── index.ejs
└── tsconfig.json
```

其次我们需要修改一下tsconfig.json文件：

```
{
  "include": [
    "./src/**/*",            // 编译src目录下的所有ts文件
    "./static/js/**/*",      // 编译static/js/目录下的所有ts文件
    "./common/**/*"          // 编译common目录下的所有ts文件
  ],
  "exclude": [
    "./src/test/**/*",       // 不编译src目录中test目录下所有的ts文件
  ],
  "compilerOptions": {
    "target": "ES6",         // 编译后生成的js版本为es6
    "module": "CommonJS",    // 模块使用规范为es6
    "lib": [                 // node环境中测试ts代码所需要使用的库
      "ES6"
    ],
    "outDir": "./dist",       // 编译后生成的js文件存放路径
    "allowJs": true,          // 二次编译js文件
    "checkJs": true,          // 验证js文件语法
    "removeComments": true,  // 编译后的js文件删除注释信息
    "noEmitOnError": true,    // 如果编译时出现错误，编译将终止
    "strict": true,           // 启用TypeScript的严格模式
    "alwaysStrict": true,     // 启用JavaScript的严格模式
    "noFallthroughCasesInSwitch": true,  // 检测switch语句块是否正确的使用了break
    "noImplicitReturns": true,           // 检测函数是否具有隐式的返回值
    "noUnusedLocals": true,              // 检测是否具有未使用的局部变量
    "noUnusedParameters": true,          // 检测是否具有未使用的函数参数
    "allowUnreachableCode": true,        // 检测是否具有永远不会运行的代码
    "allowSyntheticDefaultImports": false, // 允许从没有设置默认导出的模块中默认导入，仅用于提示，不影响编译结果
    "esModuleInterop": false               // 允许编译生成文件时，在代码中注入工具类
  }
}
```





## 具体实现

我们将所有的代码都写在./common/advancedHttp.ts文件中，为了尽量的简单，仅定义一个类：

```
import * as http from 'http';
import * as url from 'url';
import * as mime from 'mime';
import * as path from 'path';
import * as fs from 'fs';


interface ServerRequest extends http.IncomingMessage {
    queryParams: { [key: string]: any },
}

interface ServerResponse extends http.ServerResponse {
    mimeType: string;
}

type ViewType = (req: ServerRequest, res: ServerResponse) => void;

interface RouterStructure {
    method: string[];
    rule: string;
    view: ViewType;
}

type RequestRouter = RouterStructure[];

class HttpServer {

    public hostname: string;
    public port: number;
    public backlog: number;
    public router: RequestRouter;
    public baseDir: string;
    public templatesDir: string;
    public staticsDir: string;
    public server: http.Server;
    public addr: string;
    public request: ServerRequest | null;
    public response: ServerResponse | null;
    public pathName: string;

    constructor(hostname: string, port: number, router: RequestRouter, baseDir: string = ".", templatesDir: string = "./templates", staticsDir: string = "./static", backlog?: number) {
        this.hostname = hostname;
        this.port = port;
        this.router = router;
        this.backlog = backlog || 128;
        this.baseDir = baseDir;
        this.templatesDir = templatesDir;
        this.staticsDir = staticsDir;
        this.server = http.createServer(this.handleRequestParams.bind(this));
        this.addr = `http://${this.hostname}:${this.port}`;
        this.request = null;
        this.response = null;
        this.pathName = "";
    }

    // 参数处理
    handleRequestParams(req: http.IncomingMessage, res: http.ServerResponse): void {

        let data: string = "";

        req.on("data", (chunk: Buffer) => {
            data += chunk.toString();
        })

        req.on("end", () => {
            let urlIns: url.URL = new url.URL(req.url || "", this.addr);
            const postParams: url.URLSearchParams = new url.URLSearchParams(data);
            const getParams: url.URLSearchParams = urlIns.searchParams;
            const queryParams: { [key: string]: any } = {};
            getParams.forEach((v: string, k: string) => {
                queryParams[k] = v;
            })
            postParams.forEach((v: string, k: string) => {
                queryParams[k] = v;
            })
            // 封装request对象和response对象
            this.pathName = urlIns.pathname;
            this.request = Object.assign(req, { queryParams });
            this.response = Object.assign(res, { mimeType: mime.getType(this.pathName) || "" });
            this.handleRequestRouter(this.request, this.response)
        })
    }

    // 路由解析
    handleRequestRouter(req: ServerRequest, res: ServerResponse) {
        let tag: boolean = false;
        // 匹配静态资源
        if (this.pathName.includes("static")) {
            tag = this.handleStaticRequest(req, res);
        }
        // 匹配动态资源
        else {
            tag = this.handleDynamicRequest(req, res);
        }
        // 没有匹配到
        if (!tag) {
            res.writeHead(404);
            res.end("Not Fund");
        }
    }

    // 静态请求处理
    handleStaticRequest(req: ServerRequest, res: ServerResponse): boolean {
        const requestPath: string = this.pathName.split("/static/")[1];
        // 注意！如果是js文件，那么它存放的目录是./dist下，因为ts文件会先进行编译，不能直接在网页中进行使用
        const realPath = res.mimeType.includes("application/javascript") ?
            path.join(this.baseDir, "dist", path.basename(this.staticsDir), requestPath) :
            path.join(this.staticsDir, requestPath);

        if (fs.existsSync(realPath)) {
            switch (true) {
                // 请求的是视频文件
                case res.mimeType.includes("audio"):
                case res.mimeType.includes("video"):
                    const stat: fs.Stats = fs.statSync(realPath);
                    const fileSize = stat.size;
                    const range: string = req.headers.range || "";
                    if (range) {
                        // 后续的请求视频，浏览器会自动携带range请求头，包含当前播放的部分
                        const parts = range.replace(/bytes=/, "").split("-");
                        const start = parseInt(parts[0], 10);
                        const end = parts[1]
                            ? parseInt(parts[1], 10)
                            : fileSize - 1;
                        // 计算新返回的视频流长度
                        const chunkSize: number = (end - start) + 1;
                        // 根据视频流长度进行截取
                        const file: fs.ReadStream = fs.createReadStream(realPath, { start, end });
                        // 返回当前响应的视频流长度，并且请求状态要置为206
                        res.writeHead(206, {
                            'Content-Type': 'video/mp4',
                            'Content-Range': `bytes ${start}-${end}/${fileSize}`,
                            'Accept-Ranges': 'bytes',
                            'Content-Length': chunkSize
                        });
                        file.pipe(res);
                    }
                    else {
                        // 第一次请求视频，请求头只返回请求视频的总长度，和一些数据
                        res.writeHead(200, {
                            'Accept-Ranges': 'bytes',
                            'Content-Length': fileSize,
                        });
                        fs.createReadStream(realPath).pipe(res);
                    }
                    break

                // 请求的是其他文件，CSS、img、js等
                default:
                    res.writeHead(200, {
                        "Content-type": res.mimeType,
                    });
                    fs.createReadStream(realPath).pipe(res);
            }
            return true;
        }
        // 没找到静态文件
        return false;
    }

    // 动态请求处理
    handleDynamicRequest(req: ServerRequest, res: ServerResponse): boolean {
        for (let match of this.router) {
            if (this.pathName === match.rule) {
                if (match.method.includes(req.method || "")) {
                    match.view(req, res);
                } else {
                    res.writeHead(406);
                    res.end("Method Not Allowed");
                }
                return true;
            }
        }
        return false;
    }

    // 启动服务
    start(callback: () => void) {
        this.server.listen(this.port, this.hostname, this.backlog, callback.bind(this));
    }

}

export {
    ServerResponse,
    ServerRequest,
    ViewType,
    RouterStructure,
    RequestRouter,
    HttpServer
}
```

使用案例：

```
import * as path from 'path';
import * as ejs from "ejs";

import * as advHttp from "../common/advancedHttp";


const baseDir: string = path.join(__dirname, "..");
const templateDir: string = path.join(baseDir, "templates");
const staticDir: string = path.join(baseDir, "static");

const host: string = "localhost";
const port: number = 3000;

function index(req: advHttp.ServerRequest, res: advHttp.ServerResponse) {
    console.log(req.queryParams);
    console.log(res.statusCode);
    res.writeHead(200, { 'Content-Type': 'text/html charset=utf-8' });
    ejs.renderFile(path.join(templateDir, "index.ejs"), { unOrderList: ["first", "second", "third"] }, (err: Error | null, str: string) => {
        if (err) throw err;
        res.end(str);
    })
}

const router: advHttp.RequestRouter = [
    { method: ["GET", "POST"], rule: "/", view: index },
]

const server: advHttp.HttpServer = new advHttp.HttpServer(host, port, router, baseDir, templateDir, staticDir);
server.start(() => {
    console.log(`server running success! http://${host}:${port}`);
})
```







