# 双向绑定

## v-model

v-model指令常用于与from表单元素做绑定使用。

- 当表单内容发生改变时，数据层的数据也会发生改变

- 当数据层的数据发生改变时，表单内容也会发生改变

这种绑定关系是双向的，因此v-model也被称之为双向绑定：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/v-model基本使用.gif" alt="v-model基本使用" style="zoom:80%;" />

代码示例：

```
<body>
    <div id="app">
        <div>
            <label for="phone">phone:</label>
            <input type="text" name="phone" id="phone" title="Please enter your phone number" v-model="phoneNumber">
        </div>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    phoneNumber: ""
                }
            },
        }).mount("#app");
    </script>
</body>
```



## v-bind和v-model的区别

很多初学者可能疑惑v-bind和v-model的区别，这里特别指出一下：

- v-bind常用于与元素的属性做绑定，并且这个绑定关系是单向的，只有当数据层的数据发生改变时，元素的属性才会发生变更
- v-model常用于与表单元素做绑定，并且这个绑定关系是双向的，无论哪一方的数据发生改变，都会影响另一方数据的改变

举个例子，我们在之前没有学习Vue时要通过AJAX提交表单数据时只有从元素身上获取value属性值后再进行提交，但是有了Vue的双向绑定之后这一切都变的非常简单，如下示例，我们只需要将需要提交的表单元素和数据层中的提交数据做绑定，然后就可以快速的进行提交了：

```
<body>
    <div id="app">
        <form action="#">
            <div>
                <label for="name">name</label>
                <input type="text" id="name" placeholder="please enter user name" v-model="username">
            </div>
            <div>
                <label for="pwd">password</label>
                <input type="password" id="pwd" placeholder="please enter password" v-model="password">
            </div>
            <div>
                <button type="submit" @click.prevent="login">login</button>
            </div>
        </form>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script src="https://cdn.bootcdn.net/ajax/libs/axios/0.21.1/axios.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    username: "",
                    password: ""
                }
            },
            methods: {
                login(event) {
                    const url = "http://localhost:3000/";
                    axios.post(
                        url,
                        {
                            username: this.username,
                            password: this.password
                        }
                    ).then(response => {
                        switch (response.status) {
                            case 200:
                                console.log(response.data);
                                break
                            default:
                                console.log("login fail!");
                        }
                    })
                }
            }
        }).mount("#app");
    </script>
</body>
```

后端代码，采用Node.js完成：

```
const http = require('http');

const server = http.createServer((req, res) => {
    res.setHeader("Access-Control-Allow-Origin", "*");
    if (req.method === "OPTIONS") {
        res.setHeader("Access-Control-Allow-Headers", "*");
    }

    let data = "";
    req.on("data", chunk => {
        data += chunk.toString();
    })
    req.on("end", () => {
        res.writeHead(200, { "content-type": "text/plain" });
        console.log(data);
        res.end("login success!");
    })
})

server.listen(3000, "localhost");
```



# 表单操作

## input&textarea

v-model指令直接用于&lt;input&gt;以及&lt;textarea&gt;上时，当他们的value发生改变后数据层被绑定的数据也会发生改变。

注意，v-model指令直接用于&lt;input&gt;以及&lt;textarea&gt;上时数据层的绑定对象可以是任意类型：

```
<body>
    <div id="app">
        <div>
            <label for="username">username</label>
            <input id="username" type="text" v-model="username">
        </div>
        <div>
            <label for="introduction">introduction</label>
            <textarea id="introduction" v-model="introduction"></textarea>
        </div>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    username: "",
                    introduction: "",
                }
            }
        }).mount("#app");
    </script>
</body>
```



## checkbox

v-model绑定input:checkbox时，数据层的绑定对象必须是一个数组类型，因为checkbox是多选：

```
<body>
    <div id="app">
        <div>
            <label for="basketball">basketball</label>
            <input type="checkbox" id="basketball" v-model="hobby" value="basketball" checkbox="hobby">

            <label for="football">football</label>
            <input type="checkbox" id="football" v-model="hobby" value="football" checkbox="hobby">

            <label for="volleyball">volleyball</label>
            <input type="checkbox" id="volleyball" v-model="hobby" value="volleyball" checkbox="hobby">
        </div>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    hobby: []
                }
            }
        }).mount("#app");
    </script>
</body>
```





## radio

v-model绑定input:radio时，元素身上可以不用指定name属性，只要v-model相同，就会产生互斥效果，此外checked属性也不需要做指定，只要元素的value和数据层的数据一致，默认就会进行选中。

注意，v-model指令绑定在input:radio时数据层的绑定对象可以是任意类型：

```
<body>
    <div id="app">
        <div>
            <label for="male">male</label>
            <input type="radio" id="male" v-model="gender" value="male">

            <label for="female">female</label>
            <input type="radio" id="female" v-model="gender" value="female">
        </div>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    gender: "male",
                }
            }
        }).mount("#app");
    </script>
</body>
```



## select单选&复选

v-model绑定&lt;select&gt;时，需要区分单选还是多选。

- 如果&lt;select&gt;是单选，那么数据层的绑定对象可以是任意类型
- 如果&lt;select&gt;是多选，那么数据层的绑定对象必须是数组类型

单选示例：

```
<body>
    <div id="app">
        <div>
            <select name="addr" v-model="address">
                <option value="beijing">beijing</option>
                <option value="shanghai">shanghai</option>
                <option value="chongqing">chongqing</option>
            </select>
        </div>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    address: 'beijing',
                }
            }
        }).mount("#app");
    </script>
</body>
```

多选示例：

```
<body>
    <div id="app">
        <div>
            <select name="addr" v-model="choice" multiple="multiple">
                <option value="beijing">beijing</option>
                <option value="shanghai">shanghai</option>
                <option value="chongqing">chongqing</option>
            </select>
        </div>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    choice: []
                }
            }
        }).mount("#app");
    </script>
</body>
```



# v-model修饰符

## .lazy

使用该修饰符时，数据层的绑定对象不会进行实时刷新，而是等到表单元素失去焦点后才会更新：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/v-model的lazy修饰符.gif" alt="v-model的lazy修饰符" style="zoom:80%;" />

代码示例：

```
<body>
    <div id="app">
        <div>
            <input type="text" v-model.lazy="content">
        </div>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    content:""
                }
            }
        }).mount("#app");
    </script>
</body>
```



## .number

使用该修饰符时，当你输入的内容为纯数字时将会自动转为int类型后再存储到数据层的绑定对象身上。

如果不使用该修饰符，则数据层绑定对象存储的string类型的数据：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/v-model的number修饰符.gif" alt="v-model的number修饰符" style="zoom:80%;" />

代码示例：

```
<body>
    <div id="app">
        <div>
            <input type="number" v-model.number="number">
        </div>
        <div>
            <input type="number" v-model="string">
        </div>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    number: null,
                    string: null
                }
            }
        }).mount("#app");
    </script>
</body>
```



## .trim

使用该修饰符时，将会自动移除表单元素输入内容的两侧空白：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/v-model的trim修饰符.gif" alt="v-model的trim修饰符" style="zoom:80%;" />

代码示例：

```
<body>
    <div id="app">
        <div>
            <input type="text" v-model.trim="trimString">
        </div>
        <div>
            <input type="text" v-model="noTrimString">
        </div>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    trimString: "",
                    noTrimString: ""
                }
            }
        }).mount("#app");
    </script>
</body>
```

