# axios

## 基本介绍

axios是一个基于Promise的网络请求库，它可以在Node.js与浏览器中运行。

- 在Node.js中运行axios，它将依赖于http库。

- 在浏览器中运行axios，它将依赖于XMLHttpResponse。

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/axios.jpg" alt="Axios HTTP库的使用- 前端随笔| Vincent F0ng" style="zoom: 23%;" />



axios可以说是目前应用最广泛的一款网络请求库，它所提供的接口非常简洁易懂，其名称来源于ax（ajax）io（io）s（system）。

相较于jQuery.ajax()，它更加的智能，比如在上传JSON数据、或者上传文件时你甚至可以直接忽略contentType的设置而直接上传数据。



## 安装方式

在工程化项目中，推荐使用npm或者yarn进行安装：

```
$ npm install axios
$ yarn add axios
```

在非工程化项目中，你可以使用Boot CDN进行引入：

```
<script src='https://cdn.bootcdn.net/ajax/libs/axios/0.21.1/axios.js'></script>
```



## 后端代码

为了方便后续前后端交互的代码测试，我们后端采用Node.js+Koa框架来完成。

- IP：localhost
- PORT：3001

代码如下：

```
const path = require('path');
const fs = require('fs');

// npm install koa --save
const Koa = require('koa');

// npm install koa-router --save ： 用于为Koa框架添加路由功能
const router = require('koa-router')();

// npm install koa-body --save : 用于获取POST请求体参数，以及获取文件对象
const koaBody = require('koa-body');

// npm install @koa/cors --save : 用于解决前后端跨域问题
const cors = require('@koa/cors');

const server = new Koa();

server.use(cors());
server.use(koaBody(
    {
        // 是否支持 multipart-formdata 的表单
        multipart: true,
        formidable: {
            // 上传的目录
            uploadDir: path.join(__dirname, 'upload'),
            // 保持文件的后缀
            keepExtensions: true,
            // 最大支持上传8M的文件
            maxFieldsSize: 8 * 1024 * 1024,
            // 文件上传前的设置
            onFileBegin: (name, file) => {
                const filePath = path.join(__dirname, "upload");
                // 检查是否有upload目录
                if (!fs.existsSync(filePath)) {
                    fs.mkdirSync(filePath);
                    console.log("mkdir success!");
                }
            }
        }
    }
))

server.use(async (ctx, next) => {
    await next();
    ctx.response.set("Content-Type", "application/json");
    ctx.response.statusCode = 200;
})

router.get('/get', async (ctx, next) => {
    await next();
    ctx.response.body = JSON.stringify(ctx.request.query);
});

router.post('/post', async (ctx, next) => {
    await next();
    ctx.response.body = JSON.stringify(ctx.request.body);
});

router.post("/upload", async (ctx, next) => {
    await next();
    const file = ctx.request.files["img"];
    ctx.response.body = JSON.stringify(file);
})


server.use(router.routes());

server.listen(3001, "localhost");
console.log('server started at http://localhost:3001');
```



# 快速上手

## 全局配置

在axios中可以配置一些全局设置，如baseUrl或者timeout，如下所示：

- baseUrl：默认的request host，如当前缀设置为“http://localhost:3001”后，当你发送网络请求时它会自动添加上该前缀
- timeout：请求超时时间，默认为0，即永不超时

示例如下，对于这些全局配置我们可以直接书写在一个配置文件当中：

```
import axios from "axios";

axios.defaults.baseURL = "http://localhost:3001";
axios.defaults.timeout = 2000;
```





## GET

以下是发送GET请求的案例，若想携带参数你必须带上params：

```
<template>
  <div>
    <button @click="callbackfn">get</button>
  </div>
</template>

<script setup>
import axios from "axios";

axios.defaults.baseURL = "http://localhost:3001";
axios.defaults.timeout = 2000;

const callbackfn = (event) => {
  axios({
    method: "GET",
    url: "/get",
    headers: {},
    params: { getParams1: "v1", getParams2: "v2" },
    responseType: "json",
  })
    .then((response) => {
      console.log(response.data);
    })
    .catch((error) => {
      throw error;
    });
};
</script>

<style>
* {
  margin: 9;
  padding: 9;
}
#app {
  height: 100vh;
  width: 100vw;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
```



## POST

以下是发送POST请求的案例，若想携带参数你必须带上data：

```
<template>
  <div>
    <button @click="callbackfn">post</button>
  </div>
</template>

<script setup>
import axios from "axios";

axios.defaults.baseURL = "http://localhost:3001";
axios.defaults.timeout = 2000;

const callbackfn = (event) => {
  axios({
    method: "POST",
    url: "/post",
    headers: {},
    data: { postParams1: "v1", postParams2: "v2" },
    responseType: "json",
  })
    .then((response) => {
      console.log(response.data);
    })
    .catch((error) => {
      throw error;
    });
};
</script>

<style>
* {
  margin: 9;
  padding: 9;
}
#app {
  height: 100vh;
  width: 100vw;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
```



## 上传文件

上传文件必须是POST请求，我们可以利用formData对象来进行上传，如下所示：

```
<template>
  <div>
    <input type="file" @change="callbackfn" />
  </div>
</template>

<script setup>
import axios from "axios";

axios.defaults.baseURL = "http://localhost:3001";
axios.defaults.timeout = 2000;

const callbackfn = (event) => {
  // 1.封装formData
  const fd = new FormData();
  const file = event.target.files[0];
  fd.append("img", file);

  // 2.发送请求
  axios({
    method: "POST",
    url: "/upload",
    headers: {},
    data: fd,
    responseType: "json",
  })
    .then((response) => {
      console.log(response.data);
    })
    .catch((error) => {
      throw error;
    });
};
</script>

<style>
* {
  margin: 9;
  padding: 9;
}
#app {
  height: 100vh;
  width: 100vw;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
```





## 并发请求

如果想一次性发送多个请求，则可以使用all()方法，all()方法应当传入一个Array，并且这个Array中可以包含任意个axios请求，当：

```
<template>
  <div>
    <button @click="callbackfn">all</button>
  </div>
</template>

<script setup>
import axios from "axios";

axios.defaults.baseURL = "http://localhost:3001";
axios.defaults.timeout = 2000;

const callbackfn = (event) => {
  axios
    .all([
      axios({method: "GET",url: "/get",params: { getParams1: "v1", getParams2: "v2" }}),
      axios({method: "POST",url: "/post", data: { postParams1: "v1", postParams2: "v2" }}),
    ])
    .then((responseArray) => {
      responseArray.forEach((response) => {
        console.log(response.status);
        console.log(response.data);
      });
    })
    .catch((error) => {
      throw error;
    });
};
</script>

<style>
* {
  margin: 9;
  padding: 9;
}
#app {
  height: 100vh;
  width: 100vw;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
```



# 请求响应

## 请求配置

以下是请求相关的config，摘自axios官网：

```
{
  // `url` 是用于请求的服务器 URL
  url: '/user',

  // `method` 是创建请求时使用的方法
  method: 'get', // 默认值

  // `baseURL` 将自动加在 `url` 前面，除非 `url` 是一个绝对 URL。
  // 它可以通过设置一个 `baseURL` 便于为 axios 实例的方法传递相对 URL
  baseURL: 'https://some-domain.com/api/',

  // `transformRequest` 允许在向服务器发送前，修改请求数据
  // 它只能用与 'PUT', 'POST' 和 'PATCH' 这几个请求方法
  // 数组中最后一个函数必须返回一个字符串， 一个Buffer实例，ArrayBuffer，FormData，或 Stream
  // 你可以修改请求头。
  transformRequest: [function (data, headers) {
    // 对发送的 data 进行任意转换处理

    return data;
  }],

  // `transformResponse` 在传递给 then/catch 前，允许修改响应数据
  transformResponse: [function (data) {
    // 对接收的 data 进行任意转换处理

    return data;
  }],

  // 自定义请求头
  headers: {'X-Requested-With': 'XMLHttpRequest'},

  // `params` 是与请求一起发送的 URL 参数
  // 必须是一个简单对象或 URLSearchParams 对象
  params: {
    ID: 12345
  },

  // `paramsSerializer`是可选方法，主要用于序列化`params`
  // (e.g. https://www.npmjs.com/package/qs, http://api.jquery.com/jquery.param/)
  paramsSerializer: function (params) {
    return Qs.stringify(params, {arrayFormat: 'brackets'})
  },

  // `data` 是作为请求体被发送的数据
  // 仅适用 'PUT', 'POST', 'DELETE 和 'PATCH' 请求方法
  // 在没有设置 `transformRequest` 时，则必须是以下类型之一:
  // - string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams
  // - 浏览器专属: FormData, File, Blob
  // - Node 专属: Stream, Buffer
  data: {
    firstName: 'Fred'
  },
  
  // 发送请求体数据的可选语法
  // 请求方式 post
  // 只有 value 会被发送，key 则不会
  data: 'Country=Brasil&City=Belo Horizonte',

  // `timeout` 指定请求超时的毫秒数。
  // 如果请求时间超过 `timeout` 的值，则请求会被中断
  timeout: 1000, // 默认值是 `0` (永不超时)

  // `withCredentials` 表示跨域请求时是否需要使用凭证
  withCredentials: false, // default

  // `adapter` 允许自定义处理请求，这使测试更加容易。
  // 返回一个 promise 并提供一个有效的响应 （参见 lib/adapters/README.md）。
  adapter: function (config) {
    /* ... */
  },

  // `auth` HTTP Basic Auth
  auth: {
    username: 'janedoe',
    password: 's00pers3cret'
  },

  // `responseType` 表示浏览器将要响应的数据类型
  // 选项包括: 'arraybuffer', 'document', 'json', 'text', 'stream'
  // 浏览器专属：'blob'
  responseType: 'json', // 默认值

  // `responseEncoding` 表示用于解码响应的编码 (Node.js 专属)
  // 注意：忽略 `responseType` 的值为 'stream'，或者是客户端请求
  // Note: Ignored for `responseType` of 'stream' or client-side requests
  responseEncoding: 'utf8', // 默认值

  // `xsrfCookieName` 是 xsrf token 的值，被用作 cookie 的名称
  xsrfCookieName: 'XSRF-TOKEN', // 默认值

  // `xsrfHeaderName` 是带有 xsrf token 值的http 请求头名称
  xsrfHeaderName: 'X-XSRF-TOKEN', // 默认值

  // `onUploadProgress` 允许为上传处理进度事件
  // 浏览器专属
  onUploadProgress: function (progressEvent) {
    // 处理原生进度事件
  },

  // `onDownloadProgress` 允许为下载处理进度事件
  // 浏览器专属
  onDownloadProgress: function (progressEvent) {
    // 处理原生进度事件
  },

  // `maxContentLength` 定义了node.js中允许的HTTP响应内容的最大字节数
  maxContentLength: 2000,

  // `maxBodyLength`（仅Node）定义允许的http请求内容的最大字节数
  maxBodyLength: 2000,

  // `validateStatus` 定义了对于给定的 HTTP状态码是 resolve 还是 reject promise。
  // 如果 `validateStatus` 返回 `true` (或者设置为 `null` 或 `undefined`)，
  // 则promise 将会 resolved，否则是 rejected。
  validateStatus: function (status) {
    return status >= 200 && status < 300; // 默认值
  },

  // `maxRedirects` 定义了在node.js中要遵循的最大重定向数。
  // 如果设置为0，则不会进行重定向
  maxRedirects: 5, // 默认值

  // `socketPath` 定义了在node.js中使用的UNIX套接字。
  // e.g. '/var/run/docker.sock' 发送请求到 docker 守护进程。
  // 只能指定 `socketPath` 或 `proxy` 。
  // 若都指定，这使用 `socketPath` 。
  socketPath: null, // default

  // `httpAgent` and `httpsAgent` define a custom agent to be used when performing http
  // and https requests, respectively, in node.js. This allows options to be added like
  // `keepAlive` that are not enabled by default.
  httpAgent: new http.Agent({ keepAlive: true }),
  httpsAgent: new https.Agent({ keepAlive: true }),

  // `proxy` 定义了代理服务器的主机名，端口和协议。
  // 您可以使用常规的`http_proxy` 和 `https_proxy` 环境变量。
  // 使用 `false` 可以禁用代理功能，同时环境变量也会被忽略。
  // `auth`表示应使用HTTP Basic auth连接到代理，并且提供凭据。
  // 这将设置一个 `Proxy-Authorization` 请求头，它会覆盖 `headers` 中已存在的自定义 `Proxy-Authorization` 请求头。
  // 如果代理服务器使用 HTTPS，则必须设置 protocol 为`https`
  proxy: {
    protocol: 'https',
    host: '127.0.0.1',
    port: 9000,
    auth: {
      username: 'mikeymike',
      password: 'rapunz3l'
    }
  },

  // see https://axios-http.com/docs/cancellation
  cancelToken: new CancelToken(function (cancel) {
  }),

  // `decompress` indicates whether or not the response body should be decompressed 
  // automatically. If set to `true` will also remove the 'content-encoding' header 
  // from the responses objects of all decompressed responses
  // - Node only (XHR cannot turn off decompression)
  decompress: true // 默认值

}
```



## 响应结构

以下是响应对象response对象的相关结构，摘自axios官网：

```
{
  // `data` 由服务器提供的响应
  data: {},

  // `status` 来自服务器响应的 HTTP 状态码
  status: 200,

  // `statusText` 来自服务器响应的 HTTP 状态信息
  statusText: 'OK',

  // `headers` 是服务器响应头
  // 所有的 header 名称都是小写，而且可以使用方括号语法访问
  // 例如: `response.headers['content-type']`
  headers: {},

  // `config` 是 `axios` 请求的配置信息
  config: {},

  // `request` 是生成此响应的请求
  // 在node.js中它是最后一个ClientRequest实例 (in redirects)，
  // 在浏览器中则是 XMLHttpRequest 实例
  request: {}
}
```



# 拦截器

axios中可添加请求或者响应拦截器来对请求或者响应做2次处理：

```
// 添加请求拦截器
axios.interceptors.request.use(function (config) {
    // 在发送请求之前做些什么
    return config;
  }, function (error) {
    // 对请求错误做些什么
    return Promise.reject(error);
  });

// 添加响应拦截器
axios.interceptors.response.use(function (response) {
    // 2xx 范围内的状态码都会触发该函数。
    // 对响应数据做点什么
    return response;
  }, function (error) {
    // 超出 2xx 范围的状态码都会触发该函数。
    // 对响应错误做点什么
    return Promise.reject(error);
  });
```