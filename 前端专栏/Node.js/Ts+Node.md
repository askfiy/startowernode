# 基本介绍

TypeScript现在已经开始逐渐的取代JavaScript，因此在我们的后端上也推荐使用TypeScript进行代码编写。

它拥有严格的类型检查，让你的错误能够及时发现，最大限度的规范你的代码。

关于TypeScript的使用，可以查看之前关于TypeScript内容进行学习。



# 下载安装

下载安装TypeScript：

```
$ npm install typescript
```

在项目根目录中运行以下命令，生成tsconfig.json：

```
$ tsc --init
```

填入TypeScript的配置文件：

```
{
  "include": [
    "./src/**/*" // 仅编译src目录下的所有ts文件
  ],
  "exclude": [
    "./src/test/**/*" // 不编译src目录中test目录下所有的ts文件
  ],
  "compilerOptions": {
    "target": "ES6", // 编译后生成的js版本为es6
    "module": "CommonJS", // 编译后的模块使用规范为CommonJs
    "lib": [
      // node环境中测试ts代码所需要使用的库
      "ES6"
    ],
    "outDir": "./dist", // 编译后生成的js文件存放路径
    "allowJs": true, // 二次编译js文件
    "checkJs": true, // 验证js文件语法
    "esModuleInterop": true,
    "removeComments": false, // 编译后的js文件删除注释信息
    "noEmitOnError": true, // 如果编译时出现错误，编译将终止
    "strict": true, // 启用TypeScript的严格模式
    "alwaysStrict": true, // 启用JavaScript的严格模式
    "noFallthroughCasesInSwitch": true, // 检测switch语句块是否正确的使用了break
    "noImplicitReturns": true, // 检测函数是否具有隐式的返回值
    "noUnusedLocals": true, // 检测是否具有未使用的局部变量
    "noUnusedParameters": true, // 检测是否具有未使用的函数参数
    "allowUnreachableCode": true // 检测是否具有永远不会运行的代码
  }
}
```



# 相关依赖

node解释器并不认识TypeScript，我们可以下载一个ts-node，让我们能够更加方便的调试TypeScript代码。

```
$ npm install ts-node
```

其次，需要下载一个@types/node，让我们的Node.js在TypeScript中能够支持ES Module语法：

```
$ npm install @types/node
```

还需要下载3个依赖包：

```
$ npm install peer dependencies yourself
```



# 简单使用

项目目录如下：

```
.
├── dist                # 存放编译后的JavaScript代码
├── node_modules        
│   └── ...
├── package-lock.json
├── package.json
├── src                 # ts源代码存放目录
│   └── server.ts
└── tsconfig.json
```

此外你还需要对package.json做修改，主要是main和scripts属性：

```
{
  ...
  "main": "./dist/server.js",
  "scripts": {
    "serve": "node ./dist/server.js"
  },
  ...
}

```

下面编写一个HTTP服务，注意，我们在利用TypeScript编写代码的时候也将采用ES Module标准，而不是使用CommonJS：

```
import * as http from 'http';

const hostname: string = "localhost";
const port: number = 3000;

const server: http.Server = http.createServer((request: http.IncomingMessage, response: http.ServerResponse): void => {
    response.setHeader('Content-Type', 'application/json');
    response.statusCode = 200;
    response.end(JSON.stringify(request.method === "GET" ? "GET OK!!!" : "POST OK!!!"));
});

server.listen(port, hostname, (): void => {
    console.log(`server running success! http://${hostname}:${port}`);
})
```

在代码编写完成后，使用ts-node命令进行代码测试：

```
$ ts-node ./src/server.ts 
server running success! http://localhost:3000
```

代码测试无误后，使用tsc命令将ts文件编译为js文件，编译完成后的js文件将位于dist目录中：

```
$ tsc
```

由于我们的npm脚本命令serve指向了node ./dist/server.js，故可以直接使用npm run serve来运行服务：

```
$ npm run serve

> nodeproject@1.0.0 serve /Users/yunya/Project/nodeProject
> node ./dist/server.js

server running success! http://localhost:3000
```

最后可以查看一下编译完成的JavaScript文件，它使用的是CommonJS标准，因为我们已经在tsconfig.json文件中对其进行了配置：

```
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
const http = require("http");
const hostname = "localhost";
const port = 3000;
const server = http.createServer((request, response) => {
    response.setHeader('Content-Type', 'application/json');
    response.statusCode = 200;
    response.end(JSON.stringify(request.method === "GET" ? "GET OK!!!" : "POST OK!!!"));
});
server.listen(port, hostname, () => {
    console.log(`server running success! http://${hostname}:${port}`);
});

```

