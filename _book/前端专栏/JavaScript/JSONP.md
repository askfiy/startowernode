# 同源策略

所有支持JavaScript的浏览器都不允许跨域请求资源的情况出现，这种策略被称为同源策略（Same origin policy）。

所谓同源无外乎同协议、同域名、同端口。

比如你在`http://localhost:5500`下发送一个请求去获取`http://localhost:5700/`的资源时浏览器会触发同源策略对服务端响应进行拦截。

如下图所示：

![image-20210809213111993](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210809213111993.png)

要突破同源策略只有使用CORS技术和JSONP技术，这里着重介绍JSONP。





# 突破同源策略的标签

JSONP适用于请求一些服务器上的公共开放接口，并不适用于内部项目开发，也就是说只有当你需要去某个服务器上取出一些开放数据时这项技术才会被使用到。

我们仔细的想想，是不是有一些标签可以发送网络请求并且无视同源策略呢？没错，在[MDN Web Docs]((https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy))中详细的记载了有哪些标签可以突破同源策略。

如（仅列举常见标签）：

- &lt;form&gt;
- &lt;script&gt;
- &lt;link&gt;
- &lt;img&gt;
- &lt;video&gt;
- &lt;audio&gt;
- &lt;iframe&gt;

除此之外,CSS中的@font-face字体引入也是允许突破同源策略的。





# 接下来该怎么做

我们可以利用JavaScript新建一个&lt;script&gt;标签，并且通过src属性来访问远程服务端，那么这样就能获取到数据。

远程服务端会返回给你一个特定格式的字符串，如下所示：

```
"callbackfn('The data you want')"
```

仔细观察不难发现，它是准备调用一个JavaScript中的函数。

如果你定义了该函数的话，当JavaScript解析返回结果时，将自动的执行该函数。

最后我们应该将这个无用的&lt;script&gt;删除。

<p><p>

OK，到了这里我们要明白2点，整个JSONP其实是一个双向约定的过程，必须有2个必要条件：

- 作为服务方来说，你必须按照“callback(“other”)”的方式来定义请求的返回结果
- 作为请求方来说，你必须知道远程服务端返回的字符串格式，并且定义好该函数

整个JSONP请求过程图如下所示：

![image-20210809220328062](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210809220328062.png)

# 代码实现

前端代码实现，这里是用的vscode默认服务器打开的页面，端口号是5500：

```
<body>
    <button type="button">click me</button>
    <p><span></span></p>
</body>
<script>
    "use strict";
    let btnNode = document.querySelector("button");
    let spanNode = document.querySelector("span");

    btnNode.addEventListener("click", (event) => {
        let scriptNode = document.createElement("script");
        scriptNode.src = "http://localhost:5700/";
        document.head.append(scriptNode);
        document.head.removeChild(scriptNode);
    })

    function callbackfn(result) {
        spanNode.innerText = result;
    }

</script>
```

后端代码实现，这里采用Python的Flask框架作为后端，每次都返回一个随机数：

```
from logging import debug, error
from flask import Flask
import random

app = Flask(__name__)

@app.route(rule="/", methods=["GET"], strict_slashes=False)
def publicAPI():
    number = random.randint(1, 100)
    return f"callbackfn({number})"

if __name__ == "__main__":
    app.run(host="localhost", port=5700, debug=True)
```

运行结果：

![JSONP](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/JSONP.gif)

