# 基本介绍

在通过webpack开发时我们期望拥有一款更加高效的工具来提高我们的开发效率。

如果你使用vscode进行开发，那么可以通过webpack的--watch搭配live server来进行实时加载，但是这样做并不是最完美的解决方案，它依然存在以下几点问题：

- 每次源代码更新后所有的模块资源都会重新打包，即使你只修改了上百个模块中的其中一个
- 每次的编译打包操作都会引起磁盘的读写，因为打包后的资源文件会放置在磁盘上

为了解决这些问题，webpack中提供了一个server服务器，名为devServer，使用它来进行开发是我们最佳的选择。

它支持热更新、同时打包后的文件是放置在内存中，并不会写入磁盘，因此打包的速度非常快。

在进行代码编写时，devServer能够让我们更快的预览结果。



# devServer配置

要想使用devServer，先要对其进行下载，devServer只在开发环境下使用，所以需要加上-dev的后缀：

```
$ npm install webpack-dev-server --save-dev
```

要想启用devServer，推荐使用npm脚本进行操作，你只需要在package.json中的script中写入以下命令即可：

```
"scripts": {
  "dev-build": "npx webpack --config ./webpack_config/webpack.dev.config.js",
  "pro-build": "npx webpack --config ./webpack_config/webpack.pro.config.js",
  "dev-server": "npx webpack serve --config ./webpack.config.js"
},
```

然后再命令行终端中敲入以下命令即可成功开启服务：

```
$ npm run dev-server
```

它的端口号默认为8080，你可以对其进行访问。





# 相关配置

以下是devServer中的相关配置，我们要将其书写在webpack.config.js中：

```
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { DefinePlugin } = require('webpack')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const CssMinimizerPlugin = require("css-minimizer-webpack-plugin");
const CopyWebpackPlugin = require("copy-webpack-plugin");

const { resolve } = require('path');

process.env.NODE_ENV = "development"   // 加载browserslist的development兼容性列表

module.exports = {
    mode: "development",
    devtool: false,
    entry: "./src/entry.js",
    output: {
        filename: "main.js",
        path: resolve(__dirname, "dist")
    },
    // 在开发环境下，我们需要屏蔽掉browserslist的兼容性设置
    // 只需要将target设置为web即可，否则热更新将失效
    target: "web",
    devServer: {
    	// 指定devServer的域名
        host: 'localhost',
        // 指定devServer的端口号
        port: 8080,
        // 是否开启热更新(一个模块更新后，并不重复打包其他的模块)
        hot: true,
        // 服务启动成功后是否自动打开浏览器
        open: false,
        // 自动通过gzip压缩打包后的文件（在内存中），提升性能，推荐设置为true
        compress: true,
        // 在H5中（常见于单页面应用路由跳转里），当路由页面跳转后点击刷新页面时可能会发生404的情况，通过设置为true以进行解决
        historyApiFallback: true,
        // 服务器代理 --> 解决开发环境跨域问题（推荐使用Chrome插件CORS进行解决，更方便省力）
        proxy: {
            // 一旦devServer(5000)服务器接受到 /api/xxx 的请求，就会把请求转发到另外一个服务器(3000)
            '/api': {
                target: 'http://localhost:3000',
                // 发送请求时，请求路径重写：将 /api/xxx --> /xxx （去掉/api）
                pathRewrite: {
                    '^/api': ''
                }
            }
        }
    },
 	...
}
```



# HMR

如果想使用热更新的功能，我们除开需要在devServer中配置hot为true，还需要在入口文件entry.js中配置支持热更新的模块：

```
import "./static/js/index.js"   // 背后导入了index.css
import "./static/js/login.js"   // 背后导入了login.css

if (module.hot) {
    module.hot.accept(
        // 支持热更新功能的模块
        [
            "./static/js/index.js",
            "./static/js/login.js",
        ],
        // 热更新后执行的回调函数
        ()=>{
            console.log("模块更新了");
        }
    )
}
```

上述这样配置的意思是：

- 当更新了login.js模块内容时，不会影响其他模块
- 当更新了index.js模块内容时，不会影响其他模块

一个项目中有多少个小组件，就配置多少个热更新即可。

我们来看一下它的效果，首先我在login.js中添加了一个输入框：

```
console.log("login");

document.body.append(
    document.createElement("input")
)
```

然后当我更新index.js文件内容时，login.js输入框中输入的内容并不会被清除掉：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/HMR热更新.gif" alt="HMR热更新" style="zoom:50%;" />

如果你想让Vue支持热更新，请参阅官方教程，[点我跳转](https://webpack.docschina.org/guides/hot-module-replacement/#other-code-and-frameworks)：

![image-20210908213557350](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210908213557350.png)

