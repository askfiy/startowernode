# window介绍

window是一个内置的全局对象，在其中存储了许多公用的方法。

所有的浏览器都支持window对象，它并不需要我们手动的进行创建。

JavaScript中一切皆对象，故浏览器中每一个HTML文档都对应一个window对象，如下图所示：

![image-20210801141200995](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210801141200995.png)





# 常用方法

window中的常用方法如下：

| 方法            | 描述                                                       |
| --------------- | ---------------------------------------------------------- |
| alert()         | 显示带有一段消息和一个确认按钮的警告框                     |
| confirm()       | 显示带有一段消息以及确认按钮和取消按钮的对话框             |
| prompt()        | 显示可接收到用户输入的对话框                               |
| open()          | 打开一个新窗口并进入指定网址                               |
| close()         | 关闭浏览器窗口                                             |
| setinterval()   | 按照指定的周期（以毫秒计）来调用函数或计算表达式，循环调用 |
| clearInterval() | 取消由 setInterval() 设置的 timeout                        |
| setTimeout()    | 在指定的毫秒数后调用函数或计算表达式，只调用一次           |
| clearTimeout()  | 取消由 setTimeout() 方法设置的 timeout                     |
| scrollTo()      | 移动滚动条至指定位置                                       |

此外还有4个常用子对象：

| 对象           | 描述                             |
| -------------- | -------------------------------- |
| history        | 存储当前窗口的浏览记录           |
| location       | 存储当前窗口的页面信息           |
| localStorage   | 存储在浏览器中的信息，具有持久性 |
| sessionStorage | 存储在当前窗口的信息，具有临时性 |



# 交互框系列

## alert()

显示带有一段消息和一个确认按钮的警告框：

```
<script>

    "use strict";

    alert("hello,world");  // window为全局对象，不用加前缀

</script>
```

显示结果：

![image-20200810192842827](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/1881426-20200810220644162-1048302860.png)

## confirm()

显示带有一段消息以及确认按钮和取消按钮的对话框。

点击确定是true，取消是false：

```
<script>

    "use strict";

    let select = confirm("云崖是个帅哥对吗?");  // window为全局对象，不用加前缀

    console.log(select); // 点击确定是true，取消是false

</script>
```

显示结果：

![image-20200810193515078](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/1881426-20200810220643756-12299026.png)



## prompt()

显示可提示用户输入的对话框。

可用变量变量接收输入的内容：

```
<script>

    "use strict";

    let message = prompt("请输入信息");  // window为全局对象，不用加前缀

    console.log(message); // HELLO,WORLD

</script>
```

显示结果：

![image-20200810193701503](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/1881426-20200810220643450-2059328047.png)





# 窗口系列

## open()

open()方法用于打开一个新窗口并进入指定网址。

参数详解：

- url：新窗口打开的网址链接
- \_target：一个可选的字符串，它规定了新窗口的名称，或者从何处打开新窗口，如果你填入的参数在\_self、\_parent、\_top、\_blank或者已有窗口名称之间，它将在规定处打开新窗口，如果你填入了其他参数，它将视为该新窗口的名称
- features：一个可选的字符串，声明了新窗口要显示的标准浏览器的特征。如果省略该参数，新窗口将具有所有标准特征。在[窗口特征](https://www.w3school.com.cn/htmldom/met_win_open.asp#windowfeatures)这个表格中，我们对该字符串的格式进行了详细的说明。
- replace： 一个可选的布尔值。规定了装载到窗口的 URL 是在窗口的浏览历史中创建一个新条目，还是替换浏览历史中的当前条目。支持下面的值：true - URL 替换浏览历史中的当前条目。false - URL 在浏览历史中创建新的条目。

代码示例：

```
<script>

    "use strict";

    open("http://www.google.com"); // 打开一个新窗口，进入指定的网址

</script>
```

显示结果：

![img](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/1881426-20200813152043625-2027897263.png)



## close()

close()方法用于关闭当前的浏览器窗口。

```
<script>

    "use strict";

    let select = confirm("点击确定关闭当前窗口");

    if (select) { close() };

</script>
```

显示结果：

![image-20200810194821513](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/1881426-20200810220642833-1196300257.png)



# 定时器系列

## setInterval()

按照指定的周期（以毫秒计）来调用函数或计算表达式，循环调用。

```
<script>

    "use strict";

    setInterval(() => {

        console.log("hello,world");

    }, 3000);  // 每隔3000毫秒运行一次
 
</script>
```



## clearInterval()

取消由setInterval()设置的timeout，这代表将不会继续循环执行setInterval()中的代码。

以下示例将演示使用setInterval()与clearInterval()制作一个定时器。

![关键帧](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/1881426-20200810220642480-532088239-20210801163605938.gif)

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>

    <input class="show" type="text" readonly placeholder="开始计时">
    <button onclick="start(this)">开始计时</button>
    <button onclick="end(this)">停止计时</button>

</body>

<script>

    "use strict";

    let tag = null;

    function start(ele) {

        if (tag == null) {
            let time = new Date().toLocaleString();
            ele.previousElementSibling.value = time;

        }

        tag = setInterval(() => {
            let time = new Date().toLocaleString();
            ele.previousElementSibling.value = time;  // 上一个兄弟标签
        }, 1000);

    }
    
    function end(ele) {

        clearInterval(tag); // 取消继续循环
        tag = null; 
        ele.previousElementSibling.previousElementSibling.value = "继续计时";

    }

</script>
</html>
```

还有一个复杂版的：

![计时器](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/计时器.gif)

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        main {
            height: 100vh;
            width: 100vw;
            display: flex;
            flex-flow: column nowrap;
            justify-content: center;
            align-items: center;
        }

        main div:nth-of-type(1) {
            padding: 1rem;
            background-color: #ddd;
            font-weight: lighter;
            margin-bottom: .5rem;
            border-radius: 50%;
            width: 100px;
            height: 100px;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        main div:nth-of-type(2) {
            display: flex;
            border: 1px solid #ddd;
            padding: .5rem;
        }

        main div:nth-of-type(2) button {
            margin-right: .5rem;
        }

        button {
            color: #444444;
            background: #F3F3F3;
            border: 1px #DADADA solid;
            padding: 5px 10px;
            border-radius: 2px;
            font-weight: bold;
            font-size: 9pt;
            outline: none;
        }

        button:hover {
            border: 1px #C6C6C6 solid;
            box-shadow: 1px 1px 1px #EAEAEA;
            color: #333333;
            background: #F7F7F7;
        }

        button:active {
            box-shadow: inset 1px 1px 1px #DFDFDF;
        }

        /* Blue button as seen on translate.google.com*/
        button.blue {
            color: white;
            background: #4C8FFB;
            border: 1px #3079ED solid;
            box-shadow: inset 0 1px 0 #80B0FB;
        }

        button.blue:hover {
            border: 1px #2F5BB7 solid;
            box-shadow: 0 1px 1px #EAEAEA, inset 0 1px 0 #5A94F1;
            background: #3F83F1;
        }

        button.blue:active {
            box-shadow: inset 0 2px 5px #2370FE;
        }

        /* Orange button as seen on blogger.com*/
        button.orange {
            color: white;
            border: 1px solid #FB8F3D;
            background: -webkit-linear-gradient(top, #FDA251, #FB8F3D);
            background: -moz-linear-gradient(top, #FDA251, #FB8F3D);
            background: -ms-linear-gradient(top, #FDA251, #FB8F3D);
        }

        button.orange:hover {
            border: 1px solid #EB5200;
            background: -webkit-linear-gradient(top, #FD924C, #F9760B);
            background: -moz-linear-gradient(top, #FD924C, #F9760B);
            background: -ms-linear-gradient(top, #FD924C, #F9760B);
            box-shadow: 0 1px #EFEFEF;
        }

        button.orange:active {
            box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.3);
        }

        /* Red Google Button as seen on drive.google.com */
        button.red {
            background: -webkit-linear-gradient(top, #DD4B39, #D14836);
            background: -moz-linear-gradient(top, #DD4B39, #D14836);
            background: -ms-linear-gradient(top, #DD4B39, #D14836);
            border: 1px solid #DD4B39;
            color: white;
            text-shadow: 0 1px 0 #C04131;
        }

        button.red:hover {
            background: -webkit-linear-gradient(top, #DD4B39, #C53727);
            background: -moz-linear-gradient(top, #DD4B39, #C53727);
            background: -ms-linear-gradient(top, #DD4B39, #C53727);
            border: 1px solid #AF301F;
        }

        button.red:active {
            box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.2);
            background: -webkit-linear-gradient(top, #D74736, #AD2719);
            background: -moz-linear-gradient(top, #D74736, #AD2719);
            background: -ms-linear-gradient(top, #D74736, #AD2719);
        }
    </style>
</head>

<body>
    <main>
        <div>
            <time>未计时</time>
        </div>
        <div>
            <p><button class="blue">开始计时</button></p>
            <p><button class="orange">停止计时</button></p>
            <p><button class="red">清空计时</button></p>
        </div>
    </main>

</body>
<script>
    "use strict";

    let time = document.querySelector("time");
    let startBTN = document.querySelector("button[class=blue]");
    let stopBTN = document.querySelector("button[class=orange]");
    let resetBTN = document.querySelector("button[class=red]");

    let tag = null;
    let timeArray = [0, 0, 0];
    let len = timeArray.length;

    function modifyTime() {
        // 修改显示时间
        time.innerText = timeArray.map((value, index, array) => {
            let v = String(value);
            return v.length < 2 ? 0 + v : v
        }).join(":");
    }

    startBTN.addEventListener("click", event => {
        // 防止多次点击计时后计时器加快的bug
        if(tag){
            return
        }
        modifyTime();
        event.target.innerText = "正在计时";
        tag = setInterval(() => {
            if (timeArray[len - 1] != 60) {
                timeArray[len - 1] += 1;
            } else {
                timeArray[len - 1] = 1
                timeArray[len - 2] += 1;
            }

            if (timeArray[len - 2] == 60) {
                timeArray[len - 2] = 0;
                timeArray[len - 3] += 1;
            }

            if (timeArray[len - 3] == 24) {
                clearInterval(tag)
            }

            modifyTime();

        }, 1000)
    })

    stopBTN.addEventListener("click", event => {
        if (!tag) {
            alert("未开始计时!")
            return
        }
        clearInterval(tag);
        tag = null;
        startBTN.innerText = "继续计时";
    })

    resetBTN.addEventListener("click", event => {
        clearInterval(tag);
        tag = null;
        time.innerText = "未计时";
        startBTN.innerText = "开始计时";
        timeArray = [0, 0, 0];
    })

</script>

</html>
```



## setTimeout()

在指定的毫秒数后调用函数或计算表达式，只调用一次。

```
<script>

    "use strict";

    setTimeout(() => {

        console.log("hello,world");

    }, 3000);  // 3000毫秒后打印一次hello，world
 
</script>
```





## clearTimeout()

取消由setTimeout()设置的timeout，这代表将不会继续循环执行setTimeout()中的代码。

![关键帧](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/1881426-20200810220641875-354223191.gif)

```
<script>

    "use strict";

    let tag = setTimeout(() => {

        alert("HELLO,WORLD");

    }, 1000); // 一千毫秒后将打印HELLO,WORLD

    let select = confirm("如果您点击确定，会有一个弹窗在1s后向您打招呼，如果您点击取消，则没有弹窗向您打招呼。");

    if (select == false) {

        clearTimeout(tag);

    }
    // 由于同步任务在宏任务之前，所以先运行同步任务。
    
</script>
```





## 注意事项

setInterval()和setTimeout()虽然都是以毫秒进行计时，但是本身有一个最低限度。

- setInterval()的最短间隔时间是10毫秒
- setTimeout()的最短时间间隔是4毫秒，也就是说，小于4毫秒的时间间隔会被调整到4毫秒

此外，由于JavaScript是单线程的，所以定时器都不是立即执行，而是在所有同步任务执行完成后才会执行。

```
"use strict";


setTimeout(() => {
    console.log("4s");
}, 4)

for (let i = 0; i < 100000; i++) {
    console.log(i);
}
```



# history对象

## 基本介绍

history是window对象的一个子对象，实际应用中比较少见。

它包含用户在当前浏览器窗口中访问过的URL，通过它提供的方法可以进行前进或者后退。

相当于浏览器中这2个按键：

![image-20200810212742434](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/1881426-20200810220641348-1903270831.png)

方法如下所示：

| 方法           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| back()         | 加载 history 列表中的前一个URL                               |
| forward()      | 加载 history 列表中的下一个URL                               |
| go()           | 加载 history 列表中的某个具体页面                            |
| pushState()    | 向 history 列表中压入一个URL，页面会进行跳转                 |
| replaceState() | 先清空 histroy 列表，再向列表中压入一个URL，页面会进行跳转，但是无法进行back操作 |

每次用户访问过的url，都存储在history列表中，你可以将它理解为一个栈。

back()方法是出栈操作，forward()是进栈操作，而go()方法可以从栈中任意位置调出URL元素项：

![image-20210930002404584](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210930002404584.png)



## back() & forward()

以下有两个h5的页面，一个为主页一个为首页。

```
// 主页
<body>

    <a href="子页.html">跳转到子页</a>
    <button onclick="history.forward()">forward</button>

</body>
```

```
// 子页
<body>
        <button onclick="history.back()">back-返回到主页</button>
</body>
```

显示结果：

![关键帧](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/1881426-20200810220640952-923592011.gif)



## go()

使用go()也可以达到上述效果，但是里面参数要设置成+1或者-1，这样才会跳转最近一次的页面，如果设置大于1，则跳转最近n次的页面。

```
// 主页
<body>

    <a href="子页.html">跳转到子页</a>
    <button onclick="history.go(1)">go(1)</button>

</body>
// 子页
<body>
        <button onclick="history.go(-1)">go(-1)-返回到主页</button>
</body>
```

显示结果：

![关键帧](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/1881426-20200810220640345-760957695.gif)



# Localtion对象

## 基本介绍

location对象也是window对象下的一个子对象，它主要包含了当前页面的一些信息，如URL。

常用属性：

| 属性                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [hash](https://www.w3school.com.cn/jsref/prop_loc_hash.asp)  | 设置或返回从井号 (#) 开始的 URL（锚）                        |
| [host](https://www.w3school.com.cn/jsref/prop_loc_host.asp)  | 设置或返回主机名和当前 URL 的端口号                          |
| [hostname](https://www.w3school.com.cn/jsref/prop_loc_hostname.asp) | 设置或返回当前 URL 的主机名                                  |
| [href](https://www.w3school.com.cn/jsref/prop_loc_href.asp)  | 设置或返回完整的 URL ，如果设置href，则相当于在当前页打开新的文档 |
| [pathname](https://www.w3school.com.cn/jsref/prop_loc_pathname.asp) | 设置或返回当前 URL 的路径部分                                |
| [port](https://www.w3school.com.cn/jsref/prop_loc_port.asp)  | 设置或返回当前 URL 的端口号                                  |
| [protocol](https://www.w3school.com.cn/jsref/prop_loc_protocol.asp) | 设置或返回当前 URL 的协议                                    |
| [search](https://www.w3school.com.cn/jsref/prop_loc_search.asp) | 设置或返回从问号 (?) 开始的 URL（查询部分）                  |

常用方法：

| 方法                                                         | 描述                   |
| :----------------------------------------------------------- | :--------------------- |
| [assign()](https://www.w3school.com.cn/jsref/met_loc_assign.asp) | 加载新的文档           |
| [reload()](https://www.w3school.com.cn/jsref/met_loc_reload.asp) | 重新加载当前文档       |
| [replace()](https://www.w3school.com.cn/jsref/met_loc_replace.asp) | 用新的文档替换当前文档 |



## 方法案例

以下是使用location对象下提供的方法案例：

```
<script>

    "use strict";

    location.assign("http://www.google.com/");
    // 页面跳转访问google，不能通过浏览器自带的back按钮返回。

    location.reload();
    // 刷新当前页面，类似F5刷新功能

    location.replace("http://www.google.com/");
    // 使用google来替换当前页面，不能通过浏览器自带的back返回

</script>
```



# localStorage & sessionStorage

## 基本介绍

localStorage和sessionStorage都是window对象下的2个子对象。

后端中有些数据会存储在这2个地方，比如cookie，session-key等。

- localStorage中存储的信息会永久保留在浏览器中，重置浏览器后信息清除

- sessionStorage中存储的信息会临时保留在当前页面中，关闭页面后信息清除

以下是它们的操作方式：

```
localStorage.attribute = value;  // 新增或修改存储信息
del localStorage.attribute       // 删除存储信息
```

如果通过第三方工具包如$cookie进行操作就很简单，它是将数据存储在localStorage中的：

| 名称                 | 作用                                         |
| -------------------- | -------------------------------------------- |
| clear                | 清空localStorage上存储的数据                 |
| getItem              | 读取数据                                     |
| hasOwnProperty       | 检查localStorage上是否保存了变量x，需要传入x |
| key                  | 读取第i个数据的名字或称为键值(从0开始计数)   |
| length               | localStorage存储变量的个数                   |
| propertyIsEnumerable | 用来检测属性是否属于某个对象的               |
| removeItem           | 删除某个具体变量                             |
| setItem              | 存储数据                                     |
| toLocaleString       | 将（数组）转为本地字符串                     |
| valueOf              | 获取所有存储的数据                           |