# 基本介绍

## XMLHttpRequest

XMLHttpRequest是JavaScript中原生的，历史悠久的一种发送网络请求的方案。

基本上所有前端框架对于网络请求的部分都是基于它来完成的。

在没有学习XMLHttpRequest之前，与后端的交互我们都会使用&lt;form&gt;来进行，但这样做会引起页面的刷新。

那么在学习了XMLHttpRequest之后，就可以使页面不刷新的同时和后端进行数据交互了，这是一种非常常用的技术，俗称ajax，因此要好好掌握。

在本章节中我们将了解XMLHttpRequest的基本用法，并且会实现一个与jQuery.ajax功能百分之九十相似的网络请求组件。

我是没有读过jQuery.ajax源码的，并且我也不是一个专精前端的程序员，所以后面的代码写的可能不太好，仅和大家在此做个交流。



## 后端代码

为了方便后续代码测试，我们后端采用Python-flask框架来完成。

- IP：localhost
- PORT：5700

代码如下：

```
from flask import Flask
from flask import request
from flask import make_response
from flask import jsonify

app = Flask(__name__)


@app.after_request  # 解决CORS跨域请求
def cors(response):
    response.headers['Access-Control-Allow-Origin'] = "*"
    if request.method == "OPTIONS":
        # 允许的请求头
        response.headers["Access-Control-Allow-Headers"] = "Origin,Content-Type,Cookie,Accept,Token,authorization,user_head"
    return response


@app.route("/get", methods=["get"])
def get():
    user_head = request.headers.get("user_head")
    user_params = request.args
    print(user_params)
    return jsonify(user_params, user_head)


@app.route("/post", methods=["post"])
def post():
    user_head = request.headers.get("user_head")
    user_params = request.form
    print(user_params)
    return jsonify(user_params, user_head)


@app.route("/json", methods=["post"])
def json():
    user_head = request.headers.get("user_head")
    user_params = request.json
    return jsonify(user_params, user_head)


@app.route("/file", methods=["post"])
def file():
    file_obj = request.files.get("avatar")
    if file_obj is None:
        return make_response(jsonify("unload avatar"), 400)
    file_name = file_obj.filename
    file_obj.save(file_name)
    return jsonify("upload avatar success!!")


if __name__ == "__main__":
    app.run(host="localhost", port=5700, debug=True)
```



## open()

open()方法用于创建并返回一个请求对象（单纯创建，并不发送）。

参数释义：

- method：请求方式
- url：请求地址
- async：是否异步请求

注意：如果method为GET方式，则需要我们手动在url中添加参数



# 请求相关

## send()

send()方法用于发送一次网络请求。

参数释义：

- body：method为POST时携带的请求体数据，必须是字符串类型

注意：如果method为GET方式，则直接xhr.send(null)即可，因为GET请求没有请求体



## setRequestHeader()

setRequestHeader()方法用于设置请求头。

参数释义：

- header：请求头的key
- value：请求头的value

注意：每次只能设置一组请求头键值对，如果要设置多组请求头键值对请多次调用该方法





## abort()

abort()方法用于取消本次网络请求。



# 响应相关

## getAllResponseHeaders()

getAllResponseHeaders()方法用于获取所有响应头数据。

返回的数据类型为字符串。



## getResponseHeader()

getResponseHeader()方法用于获取响应头中指定header的value。

返回的数据类型为字符串。



## status

status属性用于返回服务端响应的HTTP响应状态码。

以下是常见的HTTP响应状态码：

- 200：请求成功
- 404：请求失败，请求的资源错误
- 500：请求失败，服务器内部出现错误
- 302：重定向
- 304：缓存优先



## statesText

statesText属性用于返回服务端响应的状态文本。

如OK、NotFound等字样。



## responseText

responseText属性用于返回响应体主体内容。

返回的数据类型为字符串。



# 回调函数

## readyState

readyState属性用于返回一个整数类型的状态值，它表明当前xhr对象的请求状态。

共有5种形态：

- 0：未初始化，尚未调用open()方法
- 1：已初始化，调用了open()方法，但还未调用send()方法
- 2：已发送，调用了send()方法，但还未收到服务端响应
- 3：已接收，已经收到了部分服务端响应数据
- 4：已完成，已经收到了全部服务端响应数据



## readystatechange

readystatechange事件用于设置xhr对象的readState状态发生改变时所执行的回调函数。





# 简单使用

## contentType说明

contentType是HTTP协议中的一个请求头，它包含了浏览器告知后端服务器本次发送数据的格式。

它和&lt;form&gt;表单的enctype参数类似，共有以下一些常见值的选项：

常见的媒体格式类型如下：

- text/html ： HTML格式
- text/plain ：纯文本格式
- text/xml ： XML格式
- image/gif ：gif图片格式
- image/jpeg ：jpg图片格式
- image/png：png图片格式

以application开头的媒体格式类型：

- application/xhtml+xml ：XHTML格式
- application/xml： XML数据格式
- application/atom+xml ：Atom XML聚合格式
- application/json： JSON数据格式
- application/pdf：pdf格式
- application/msword ： Word文档格式
- application/octet-stream ： 二进制流数据（如常见的文件下载）
- application/x-www-form-urlencoded ： &lt;form enctype=“”&gt;中默认的enctype，form表单数据被编码为key/value格式发送到服务器（表单默认的提交数据的格式）

另外一种常见的媒体格式是上传文件之时使用的：

- multipart/form-data ： 在表单中进行文件上传时，就需要使用该格式



我们接下来会使用2种最主流的格式进行试验：

- application/x-www-form-urlencoded （默认的GET和POST数据格式）
- application/json（前端需要发送的JSON格式数据）





## GET请求

发送GET请求：

```
"use strict";

let xhr = new XMLHttpRequest();

// 绑定回调函数
xhr.addEventListener("readystatechange", () => {
    switch (xhr.readyState) {
        case 0:
            console.log("未初始化，尚未调用open()方法");
            break
        case 1:
            console.log("已初始化，调用了open()方法，但还未调用send()方法");
            break
        case 2:
            console.log("已发送，调用了send()方法，但还未收到服务端响应");
            break
        case 3:
            console.log("已接收，已经收到了部分服务端响应数据");
            break
        default:
            console.log("已完成，已经收到了全部服务端响应数据");
            if (xhr.statusText === "OK") {
                console.log("请求成功");
                console.log(xhr.status);
                console.log(xhr.statusText);
                console.log(xhr.responseText);
            } else {
                console.log("请求失败");
                console.log(xhr.status);
                console.log(xhr.statusText);
                console.log(xhr.responseText);
            }
    }
})

// 请求方式，请求地址与参数，是否开启异步提交  提交的数据格式为url编码
xhr.open("GET", "http://localhost:5700/get?username=Jack&userage=18", true);

// 设置请求头，提交的数据格式为url编码
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded; charset-UTF-8');
xhr.setRequestHeader('user_head', 'HELLO WORLD');

// 发送请求
xhr.send(null);
```



## POST请求

发送POST请求：

```
"use strict";

let xhr = new XMLHttpRequest();

// 绑定回调函数
xhr.addEventListener("readystatechange", () => {
    switch (xhr.readyState) {
        case 0:
            console.log("未初始化，尚未调用open()方法");
            break
        case 1:
            console.log("已初始化，调用了open()方法，但还未调用send()方法");
            break
        case 2:
            console.log("已发送，调用了send()方法，但还未收到服务端响应");
            break
        case 3:
            console.log("已接收，已经收到了部分服务端响应数据");
            break
        default:
            console.log("已完成，已经收到了全部服务端响应数据");
            if (xhr.statusText === "OK") {
                console.log("请求成功");
                console.log(xhr.status);
                console.log(xhr.statusText);
                console.log(xhr.responseText);
            } else {
                console.log("请求失败");
                console.log(xhr.status);
                console.log(xhr.statusText);
                console.log(xhr.responseText);
            }
    }
})

// 请求方式，请求地址与参数，是否开启异步提交
xhr.open("POST", "http://localhost:5700/post", true);

// 设置请求头，提交的数据格式为url编码
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded; charset-UTF-8');
xhr.setRequestHeader('user_head', 'HELLO WORLD');

// 发送请求，提交的数据格式为url编码
xhr.send("username=Jack&userage=18");
```



## JSON格式

JSON格式的数据只能由POST请求发起，并且contentType应设置为appliction/json。

示例如下：

```
"use strict";

let xhr = new XMLHttpRequest();

// 绑定回调函数
xhr.addEventListener("readystatechange", () => {
    switch (xhr.readyState) {
        case 0:
            console.log("未初始化，尚未调用open()方法");
            break
        case 1:
            console.log("已初始化，调用了open()方法，但还未调用send()方法");
            break
        case 2:
            console.log("已发送，调用了send()方法，但还未收到服务端响应");
            break
        case 3:
            console.log("已接收，已经收到了部分服务端响应数据");
            break
        default:
            console.log("已完成，已经收到了全部服务端响应数据");
            if (xhr.statusText === "OK") {
                console.log("请求成功");
                console.log(xhr.status);
                console.log(xhr.statusText);
                console.log(xhr.responseText);
            } else {
                console.log("请求失败");
                console.log(xhr.status);
                console.log(xhr.statusText);
                console.log(xhr.responseText);
            }
    }
})

// 请求方式，请求地址与参数，是否开启异步提交
xhr.open("POST", "http://localhost:5700/json", true);

// 设置请求头
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.setRequestHeader('user_head', 'HELLO WORLD');

// 发送请求
xhr.send(JSON.stringify({ "k1": "v1", "k2": "v2" }));
```



# 了解jQuery.ajax的原理

## 实现代码

原生的XmlHttpRequest所提供的方法太过于底层，因此我们可以对其进行封装。

这里以jQuery的Ajax为蓝本，实现一个类似功能的小组件，也是为了方便后期更加了解jQuery.Ajax()做准备。

我们准备实现的功能：

- 实现发送GET请求
- 实现发送POST请求
- 实现更加方便的添加请求头
- 实现发送JSON格式的请求数据
- 实现自动反序列化JSON格式的响应数据
- 实现文件上传

实现如下：

```
"use strict";

class Ajax {

    constructor({ url, contentType, headers, data, beforeSend, dataFilter, success, error, complete, dataType = "", processData = true, type, method = type, traditional = false }) {

        // url：发送请求的地址，String类型
        // type：发送请求的方式，String类型，它和method都可以使用
        // method：发送请求的方式，String类型，它和type都可以使用
        // contentType：数据编码格式，相当于form表单的enctype，String类型
        // headers：设置的请求头，Object类型
        // dataType：接收的响应数据类型，如果是json则自动进行反序列化，String类型
        // data：发送的数据，post请求方式发送在请求体内，get请求发送会将其添加在url后，Object类型
        // processData：如果为true，浏览器将采用application/x-www-form-urlencoded方式对数据进行编码，如果为false则按照传入的contentType为准，bool类型
        // traditional：如果前端传递的是一个Array，如{"k1":[1,2,3,4]}则需要添加一个属性traditional:true，否则后端将接收不到该参数。（实际上接受的时候要使用request.POST.get("k1[]")）来进行接受，这是有问题的

        this.xhr = new XMLHttpRequest();
        this.url = url;
        this.type = (type || method).toUpperCase();
        this.method = method.toUpperCase();
        this.headers = headers;
        this.contentType = processData && !contentType ? "application/x-www-form-urlencoded" : contentType;
        this.data = data;
        this.dataType = dataType.toLowerCase();
        this.traditional = traditional;

        // beforeSend 在发送请求之前调用，并且传入一个XMLHttpRequest作为参数。
        // dataFilter 在请求成功之后调用。传入返回的数据以及"dataType"参数的值。并且必须返回新的数据（可能是处理过的）传递给success回调函数，也就是说它会在success与error之前调用
        // success 在请求成功之后调用。传入返回后的数据，以及包含成功代码的字符串
        // error 在请求出错时调用。传入XMLHttpRequest对象，描述错误类型的字符串以及一个异常对象（如果有的话）
        // complete 当请求完成之后调用这个函数，无论成功或失败。传入XMLHttpRequest对象，以及一个包含成功或错误代码的字符串

        this.beforeSend = beforeSend || function (xhr) { };

        // 该函数默认会自动的根据dataType对象responseText进行解码
        this.dataFilter = dataFilter || function (responseText, dataType) {
            switch (dataType) {
                case "application/json": case "json":
                    // 新增，this.xhr.responseJSON属性
                    this.xhr.responseJSON = JSON.parse(responseText);
                    return this.xhr.responseJSON;
                default:
                    return responseText;
            }
        };
        this.success = success || function (responseText, statusText) { };
        this.error = error || function (responseText, statusText) { };
        this.complete = complete || function (xhr, statusText) { };



        // 绑定回调函数
        this.bindStateChangeEvent();
        // 设置发送的数据并发送
        this.setRequestData();
        // 设置发送方式
        this.setXhrOpen();
        // 设置请求头
        this.setRequestHeader();
        // 发送请求
        this.sendRequest();
    }

    bindStateChangeEvent() {
        let resultText = "";
        let statusText = "";

        // 绑定回调函数，这里采用匿名函数使this指向Object，否则this将指向xhr
        this.xhr.addEventListener("readystatechange", () => {
            switch (this.xhr.readyState) {
                case 1:
                    this.beforeSend(this.xhr);
                    break
                case 4:
                    // 回调函数dataFilter的dataType参数如果你设定了就传递设定的值，如果未设定就用响应头里的contentType进行传递
                    resultText = this.dataFilter(this.xhr.responseText, this.dataType || this.xhr.getResponseHeader("Content-Type").toLowerCase());
                    if (this.xhr.statusText === "OK") {
                        statusText = "success";
                        this.success(resultText, statusText);
                    } else {
                        statusText = "error";
                        this.error(this.xhr, "error");
                    }
                    this.complete(this.xhr, statusText);
                    break
            }
        });
    }

    setRequestData() {
        // 判断是否要进行url编码，有2种情况：
        // 1.contentType是application/x-www-form-urlencoded时需要url编码
        // 2.传入的对象不是serialize()方法已经序列化好后的对象时不需要url编码
        if (this.contentType === "application/x-www-form-urlencoded" && !Object.isFrozen(this.data)) {
            this.data = this.urlEncode(this.data);
        }
    }

    urlEncode() {
        return Object.entries(this.data).reduce((prev, cur, index, array) => {
            // key
            let key = cur[0];
            // value：2种情况，如果value是普通字符串，则直接进行encodeURIComponent编码处理，如果是数组将按照 k1=v1&k2=v2 的方式进行编码，并且判断是否传入traditional
            let value = this.checkValueIsArray(cur);

            if (array.length - 1 === index) {
                // 判断是否是最后一位，如果是最后一位就加上&
                // traditional：如果前端传递的是一个Array，如{"k1":[1,2,3,4]}则需要添加一个属性traditional:true，否则后端将接收不到该参数。（实际上接受的时候要使用request.POST.get("k1[]")）来进行接受，这是有问题的
                return !this.traditional && cur[1] instanceof Array ? prev += `${key}[]=${value}` : prev += `${key}=${value}`;
            } else {
                return !this.traditional && cur[1] instanceof Array ? prev += `${key}[]=${value}&` : prev += `${key}=${value}&`;
            }
        }, "");

    }

    checkValueIsArray(cur) {
        if (cur[1] instanceof Array) {
            cur[1].forEach((value, index, array) => {
                if (index != array.length - 1) {
                    value = encodeURIComponent(value);
                    array[index] = value + "&";
                } else {
                    array[index] = String(value);
                }
            })

            // traditional：如果前端传递的是一个Array，如{"k1":[1,2,3,4]}则需要添加一个属性traditional:true，否则后端将接收不到该参数。（实际上接受的时候要使用request.POST.get("k1[]")）来进行接受，这是有问题的，这里主要对数组中的第2个元素开始及其之后的所有元素进行url编码格式转换
            return this.traditional ? cur[1].join(`${cur[0]}=`) : cur[1].join(`${cur[0]}[]=`)
        }
        else {
            return encodeURIComponent(cur[1])
        }
    }

    setXhrOpen() {
        // 如果请求方式是GET请求，则在url后加上编码后的data，否则将只传递this.url
        // true参数为开启异步调用，也是默认的选项
        this.xhr.open(
            this.method,
            this.method === "GET" ? this.url.concat("?", this.data) : this.url,
            true
        );
    }

    setRequestHeader() {
        // 在不上传文件的时候，请求头Content-Type是必须的
        for (let [k, v] of Object.entries(this.headers)) {
            this.xhr.setRequestHeader(k, v);
        }
        if (this.contentType) {
            this.xhr.setRequestHeader("Content-Type", this.contentType);
        }
    }

    sendRequest() {
        // 发送请求时，如果请求方式为GET将使用this.xhr.send(null)，否则将使用this.xhr.send(this.data)
        this.xhr.send(this.method === "GET" ? null : this.data);
    }
}

function getAjax(ajaxSettings) {
    return new Ajax(ajaxSettings);
}


function serialize(selector) {
    // 整体思路：先获取一个包含所有表单项对象，再将对象进行url编码

    let formNode = document.querySelector(selector);
    let haveValueNodeObject = {};
    Array.prototype.forEach.call(formNode.elements, (element, index, lst) => {
        switch (true) {
            // 多选或者单选select都将组成列表
            case element.tagName === "SELECT":
                Array.prototype.forEach.call(element.options, (option, index, array) => {
                    if (option.selected) {
                        haveValueNodeObject[element.name] ? haveValueNodeObject[element.name].push(option.value) : haveValueNodeObject[element.name] = [option.value];
                    }
                })
                break

            // 多选，组成列表
            case element.type === "checkbox":
                if (element.checked) {
                    haveValueNodeObject[element.name] ? haveValueNodeObject[element.name].push(element.value) : haveValueNodeObject[element.name] = [element.value];
                }
                break

            // 单选
            case element.type === "radio":
                if (element.checked) {
                    haveValueNodeObject[element.name] = element.value
                }
                break

            // 其他的项目，注意文件对象不获取
            default:
                if (element.name && element.type !== "file") {
                    haveValueNodeObject[element.name] = element.value || "";
                }
        }
    }
    )

    // 冻结对象，代表已经经过序列化了，在发送数据时已避免url二次编码
    // 这里是传入一个对象并且继承Ajax的原型对象，因为这样做才能在编码时使用checkValueIsArray方法
    return Object.freeze(Ajax.prototype.urlEncode.call(
        { data: haveValueNodeObject, traditional: true, __proto__: Ajax.prototype },
    ));
}

function serializeArray(selector) {
    // 提取form表单中的数据，并将其构建为一个name：value的数组 [{name:value}, {name:value}, {name:value}]

    let formNode = document.querySelector(selector);
    let haveValueArray = [];
    Array.prototype.forEach.call(formNode.elements, (element, index, lst) => {
        switch (true) {

            case element.tagName === "SELECT":
                Array.prototype.forEach.call(element.options, (option, index, array) => {
                    if (option.selected) {
                        haveValueArray.push({ "name": element.name, "value": option.value });
                    }
                })
                break

            case element.type === "checkbox" || element.type === "radio":
                if (element.checked) {
                    haveValueArray.push({ "name": element.name, "value": element.value });
                }
                break

            default:
                if (element.name && element.type !== "file") {
                    haveValueArray.push({ "name": element.name, "value": element.value });
                }
        }
    }
    )
    return haveValueArray;
}

window.$ = { ajax: getAjax, serialize, serializeArray };
```





## 支持的上传格式

由于我们JavaScript的原生Ajax高度按照jQuery.ajax作为蓝本。所以常见的特性要与其保持一致。

如支持的原生上传数据格式实现：

- 仅支持上传k-v的对象，不支持上传数组
- 前端上传的数据中，不允许出现对象嵌套的形式。如{"k1":{"k1-1":v1}}，这样只会得到{“k1” : “object”}
- 如果前端传递的是一个Array，如 {"k1":[1, 2, 3, 4]} 则需要添加一个属性 traditional:true，否则后端将接收不到该参数。（实际上接受的时候要使用request.POST.get("k1[]")）来进行接受，这是有问题的

如果你上传的数据是json格式，当然就不会出现这些问题了。



## 如何发送GET请求

发送GET请求示例如下：

```
// 发送GET请求
$.ajax({
    url: "http://localhost:5700/get",
    method: "get",
    headers: {
        "user_head": "Hello World",
    },
    data: { "name": "Jack", "age": 18 },
    dataType: "json",
    success: (responseText, statusText) => {
        console.log(responseText);
        console.log(statusText);
    },
    error: (responseText, statusText) => {
        console.log(responseText);
        console.log(statusText);
    },
    complete: (xhr, statusText) => {
        console.log("此处可获取响应头");
    }
})
```



## 如何发送POST请求

发送POST请求示例如下：

```
 // 发送post请求
 $.ajax({
    url: "http://localhost:5700/post",
    method: "post",
    headers: {
        "user_head": "Hello World",
    },
    data: { "name": "Jack", "age": 18 },
    dataType: "json",
    success: (responseText, statusText) => {
        console.log(responseText);
        console.log(statusText);
    },
    error: (responseText, statusText) => {
        console.log(responseText);
        console.log(statusText);
    },
    complete: (xhr, statusText) => {
        console.log("此处可获取响应头");
    }
})
```





## 如何发送JSON数据

发送JSON格式数据示例如下：

```
// 发送json格式数据
$.ajax({
    url: "http://localhost:5700/json",
    method: "post",
    headers: {
        "user_head": "Hello World",
    },
    data: JSON.stringify({ "name": "Jack", "age": 18 }),    // 1.手动进行JSON格式化
    dataType: "json",
    contentType: "application/json",                        // 2.请求头中声明本次发送的是JSON格式数据
    success: (responseText, statusText) => {
        console.log(responseText);
        console.log(statusText);
    },
    error: (responseText, statusText) => {
        console.log(responseText);
        console.log(statusText);
    },
    complete: (xhr, statusText) => {
        console.log("此处可获取响应头");
    }
})
```





## 如何提交form表单的数据

针对form表单提交的数据，我们可以使用封装好的2个函数serialize()和serializeArray()。

- serialize()：提取form表单中的数据项，并对其做url编码处理，返回一个字符串，注意，它不会提取文件选择框

- serializeArray()：提取form表单中的数据，并将其构建为一个name：value的数组，注意，它不会提取文件选择框，最终格式为 [{name:value}, {name:value}, {name:value}]，


示例如下，如果是serialize()则直接提交即可：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <form action="#" id="register">
        <div>
            <label for="username">用户名</label>
            <input type="text" id="username" name="username" required>
        </div>

        <div>
            <label for="password">密码</label>
            <input type="password" id="password" name="password" required>
        </div>
        <div>
            <label for="male">男</label>
            <input type="radio" name="gender" value="male" id="male" required>
            <label for="female">女</label>
            <input type="radio" name="gender" value="female" id="female">
        </div>

        <div>
            <label for="basketball">篮球</label>
            <input type="checkbox" name="hobby" value="basketball" id="basketball" required>
            <label for="football">足球</label>
            <input type="checkbox" name="hobby" value="football" id="football">
            <label for="volleyball">排球</label>
            <input type="checkbox" name="hobby" value="volleyball" id="volleyball">
        </div>

        <div>
            <label for="city">城市</label>
            <select name="city" id="city" multiple>
                <option value="beijing">北京</option>
                <option value="shanghai">上海</option>
                <option value="shenzhen">深圳</option>
            </select>
        </div>

        <div>
            <label for="avatar">上传头像</label>
            <input type="file" name="avatar" id="avatar">
        </div>

        <div>
            <button type="button">提交</button>
        </div>

    </form>
</body>
<script src="./demo.js"></script>
<script>
    document.querySelector("button").addEventListener("click", (event) => {
        $.ajax({
            url: "http://localhost:5700/post",
            method: "post",
            headers: {
                "user_head": "Hello World",
            },
            data: $.serialize("#register"),
            dataType: "json",
            success: (responseText, statusText) => {
                console.log(responseText);
                console.log(statusText);
            },
            error: (responseText, statusText) => {
                console.log(responseText);
                console.log(statusText);
            },
            complete: (xhr, statusText) => {
                console.log("此处可获取响应头");
            }
        })

        console.log($.serialize("#register"));
        // username=%E4%BA%91%E5%B4%96&password=123&gender=male&hobby=basketball&hobby=football&city=shanghai&city=shenzhen
    })

</script>

</html>
```

如果是serializeArray()，需要使用appliction/json的方式进行提交，因为该方法返回的是一个数组：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <form action="#" id="register">
        <div>
            <label for="username">用户名</label>
            <input type="text" id="username" name="username" required>
        </div>

        <div>
            <label for="password">密码</label>
            <input type="password" id="password" name="password" required>
        </div>
        <div>
            <label for="male">男</label>
            <input type="radio" name="gender" value="male" id="male" required>
            <label for="female">女</label>
            <input type="radio" name="gender" value="female" id="female">
        </div>

        <div>
            <label for="basketball">篮球</label>
            <input type="checkbox" name="hobby" value="basketball" id="basketball" required>
            <label for="football">足球</label>
            <input type="checkbox" name="hobby" value="football" id="football">
            <label for="volleyball">排球</label>
            <input type="checkbox" name="hobby" value="volleyball" id="volleyball">
        </div>

        <div>
            <label for="city">城市</label>
            <select name="city" id="city" multiple>
                <option value="beijing">北京</option>
                <option value="shanghai">上海</option>
                <option value="shenzhen">深圳</option>
            </select>
        </div>

        <div>
            <label for="avatar">上传头像</label>
            <input type="file" name="avatar" id="avatar">
        </div>

        <div>
            <button type="button">提交</button>
        </div>

    </form>
</body>
<script src="./demo.js"></script>
<script>
    document.querySelector("button").addEventListener("click", (event) => {
        $.ajax({
            url: "http://localhost:5700/json",
            method: "post",
            headers: {
                "user_head": "Hello World",
            },
            data: JSON.stringify($.serializeArray("#register")),    // 1.手动进行JSON格式化
            dataType: "json",
            contentType: "application/json",                        // 2.请求头中声明本次发送的是JSON格式数据
            success: (responseText, statusText) => {
                console.log(responseText);
                console.log(statusText);
            },
            error: (responseText, statusText) => {
                console.log(responseText);
                console.log(statusText);
            },
            complete: (xhr, statusText) => {
                console.log("此处可获取响应头");
            }
        })


        console.log($.serializeArray("#register"));
        // Array(7)
        // 0: {username: "云崖"}
        // 1: {password: "123"}
        // 2: {gender: "male"}
        // 3: {hobby: "basketball"}
        // 4: {hobby: "football"}
        // 5: {city: "shanghai"}
        // 6: {city: "shenzhen"}
    })

</script>

</html>
```





## 如何进行文件上传

如果要发送文件，我们需要借助FormData对象进行数据提交，以下是注意事项。

- 在&lt;form&gt;表单中上传文件，必须要将enctype设置为multipart/form-data。

- 但是在使用XmlHttpRequest对象上传文件时，并不需要指定 contentType为multipart/form-data 格式，所以不添加contentType请求头。

- contentType应设置为false，即不使用任何数据格式，不使用任何编码
- processData应设置为false，不让浏览器做任何数据格式的编码

示例如下，我们使用FormData搭配serializeArray()方法实现一个真正意义上的异步提交表单：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <form action="#" id="register">
        <div>
            <label for="username">用户名</label>
            <input type="text" id="username" name="username" required>
        </div>

        <div>
            <label for="password">密码</label>
            <input type="password" id="password" name="password" required>
        </div>
        <div>
            <label for="male">男</label>
            <input type="radio" name="gender" value="male" id="male" required>
            <label for="female">女</label>
            <input type="radio" name="gender" value="female" id="female">
        </div>

        <div>
            <label for="basketball">篮球</label>
            <input type="checkbox" name="hobby" value="basketball" id="basketball" required>
            <label for="football">足球</label>
            <input type="checkbox" name="hobby" value="football" id="football">
            <label for="volleyball">排球</label>
            <input type="checkbox" name="hobby" value="volleyball" id="volleyball">
        </div>

        <div>
            <label for="city">城市</label>
            <select name="city" id="city" multiple>
                <option value="beijing">北京</option>
                <option value="shanghai">上海</option>
                <option value="shenzhen">深圳</option>
            </select>
        </div>

        <div>
            <label for="avatar">上传头像</label>
            <input type="file" name="avatar" id="avatar">
        </div>

        <div>
            <button type="button">提交</button>
        </div>

    </form>
</body>
<script src="./demo.js"></script>
<script>
    document.querySelector("button").addEventListener("click", (event) => {

        // 获取上传的文件对象
        let fileNode = document.querySelector("#avatar");
        let fileObj = fileNode.files[0];

        // 使用FormData用于伪造form表单提交的数据
        let fd = new FormData();

        // 添加文件
        fd.append(fileNode.name, fileObj);

        // 添加其他表单项
        $.serializeArray("#register").forEach((obj, index, array) => {
            fd.append(obj.name, obj.value);
        });

        // 发送json格式数据
        $.ajax({
            url: "http://localhost:5700/file",
            method: "post",
            headers: {
                "user_head": "Hello World",
            },
            data: fd,            // 直接发送ForData对象即可
            dataType: "json",
            contentType: false,  // 必须设置为false
            processData: false,  // 必须设置为false
            success: (responseText, statusText) => {
                console.log(responseText);
                console.log(statusText);
            },
            error: (xhr, statusText) => {
                console.log(xhr.responseText);
                console.log(statusText);
            },
            complete: (xhr, statusText) => {
                console.log(statusText);
            }
        })
    })
</script>

</html>
```



