# 基本介绍

我们都知道，使用&lt;form&gt;表单提交数据时，会造成页面的刷新。

如果你想提交数据的同时不让页面进行刷新，则可用XMLHttpRequest发送异步网络请求。

但是在这里我们介绍一种&lt;iframe&gt;搭配&lt;form&gt;表单实现的伪Ajax请求，也能让页面不刷新的同时提交数据，它的兼容性是最好的，因为不需要借助XMLHttpRequest。







# 实现过程

首先，&lt;form&gt;标签有一个target属性，它能规定在何处显示action URL的提交结果。

在target属性中，有一个可设置的选项为framename，即在内联框架&lt;iframe&gt;中打开action的提交结果。

我们可以设置一个display为none的&lt;iframe&gt;，并且将其与&lt;form&gt;标签的target属性与&lt;iframe&gt;的name属性进行绑定，这样提交后服务端的响应结果就会存储在这个&lt;iframe&gt;标签中，接下来再通过JavaScript拿到&lt;iframe&gt;中的响应内容即可。



> 注意！伪Ajax请求只能应用在前后端混合开发中，前后端分离项目中由于&lt;iframe&gt;子页面与document主页面跨域，故不能获取到数据
>
> 也就是说，如果页面A和页面B不是来自同一个域，那么我们将不能在页面A上去获取页面B的数据，而&lt;iframe&gt;就是一个单独的嵌套子页面



后端采用Python的Flask框架完成。

由于&lt;form&gt;表单提交数据时并不会触发浏览器的同源策略，所以后端并不需要做CORS：

```
from flask import Flask
from flask import request
from flask import make_response
from flask import jsonify
from flask.templating import render_template

app = Flask(__name__, template_folder="./templates")


@app.route(rule="/", methods=["GET"])
def index():
    return render_template("index.html")


@app.route(rule="/register", methods=["POST"])
def register():
    user_message = {
        **request.values.to_dict()
    }
    avatar = request.files.get("avatar")
    user_message["avatar"] = "uploaded success" if avatar else "no uploaded"
    return jsonify(user_message)


if __name__ == "__main__":
    app.run(host="localhost", port=5700, debug=True)
```

前端的index.html代码：

```
<body>
    <iframe style="display: none;" name="formTarget"></iframe>

    <!-- 注意，上传文件时依然需要指定enctype="multipart/form-data" -->
    <form action="/register" method="POST" enctype="multipart/form-data" target="formTarget">
        <div>
            <label for="username">用户名</label>
            <input type="text" id="username" name="username">
        </div>

        <div>
            <label for="password">密码</label>
            <input type="password" id="password" name="password">
        </div>
        <div>
            <label for="male">男</label>
            <input type="radio" name="gender" value="male" id="male">
            <label for="female">女</label>
            <input type="radio" name="gender" value="female" id="female">
        </div>

        <div>
            <label for="basketball">篮球</label>
            <input type="checkbox" name="hobby" value="basketball" id="basketball">
            <label for="football">足球</label>
            <input type="checkbox" name="hobby" value="football" id="football">
            <label for="volleyball">排球</label>
            <input type="checkbox" name="hobby" value="volleyball" id="volleyball">
        </div>

        <div>
            <label for="city">城市</label>
            <select name="city" id="city">
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
            <button type="submit">提交</button>
        </div>
    </form>

    <footer></footer>

</body>
<script>
    "use strict";

    let iframeNode = document.querySelector("iframe");

    iframeNode.addEventListener("load", (event) => {
        // 从iframe表单中提取出内容
        let submitResult = event.target.contentWindow.document.body.innerHTML;
        // 如果前后端分离，则因为同源策略的影响不能获取数据
        document.querySelector("footer").innerHTML = submitResult;
    })

</script>
```



显示结果：

![伪Ajax](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/伪Ajax.gif)

# 组件封装

其实伪Ajax请求用的比较少，因为它的局限性很强，就是只能在前后端混合开发时使用。

在这里我还是将它封装成一个组件，用于混合开发的朋友做一个参考，实现思路如下：

- 通过JavaScript创建一个&lt;iframe&gt;，并添加name属性
- 接收一个&lt;form&gt;表单，将其target属性设置为&lt;iframe&gt;的name属性
- 注册一个回调函数

代码实现：

```
<body>
    <!-- 注意，上传文件时依然需要指定enctype="multipart/form-data" -->
    <form action="/register" method="POST" enctype="multipart/form-data" id="register">
        <div>
            <label for="username">用户名</label>
            <input type="text" id="username" name="username">
        </div>

        <div>
            <label for="password">密码</label>
            <input type="password" id="password" name="password">
        </div>
        <div>
            <label for="male">男</label>
            <input type="radio" name="gender" value="male" id="male">
            <label for="female">女</label>
            <input type="radio" name="gender" value="female" id="female">
        </div>

        <div>
            <label for="basketball">篮球</label>
            <input type="checkbox" name="hobby" value="basketball" id="basketball">
            <label for="football">足球</label>
            <input type="checkbox" name="hobby" value="football" id="football">
            <label for="volleyball">排球</label>
            <input type="checkbox" name="hobby" value="volleyball" id="volleyball">
        </div>

        <div>
            <label for="city">城市</label>
            <select name="city" id="city">
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
            <button type="submit">提交</button>
        </div>
    </form>

    <footer></footer>

</body>
<script>

    "use strict";


    function foreAjax(selector, callbackfn = (resultText) => { }) {
        let node = document.querySelector(selector);
        let iframeNode = document.createElement("iframe");

        iframeNode.style.display = "none";
        iframeNode.name = "targetIframe";
        node.append(iframeNode);
        node.target = iframeNode.name;

        iframeNode.addEventListener("load", event => {
            callbackfn(event.target.contentWindow.document.body.innerText);
        })

    }

    // 使用方式
    foreAjax("#register", (resultText) => {
        console.log(resultText);
    })

</script>
```

