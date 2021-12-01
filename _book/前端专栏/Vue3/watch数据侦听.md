# 基本使用

我们可以使用watch来监听数据层数据的变化。

watch应当是一个对象，且在内部定义了一些方法，这些方法必须与被监听的数据（可以是计算属性）名字相同，它具有2个参数，分别是newValue和oldValue。

watch中的方法只能被动调用，当被侦听的属性发生改变时，将会自动调用侦听器中的同名方法。

下面是一个简单的例子：

![watch基本使用](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/watch基本使用.gif)



代码示例：

```
<body>
    <div id="app">
        <div>
            <button v-on:click="number++">+</button>
            <span>&nbsp;{{number}}&nbsp;</span>
            <button v-on:click="number--">-</button>
        </div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    number: 0
                }
            },
            watch: {
                number(newValue, oldValue) {
                    console.log(`${oldValue} => ${newValue}`);
                }
            }
        }).mount("#app");
    </script>
</body>
```





# 适用场景

watch中的方法是支持异步操作的，所以特别适用于在某些特定场景下进行网络请求。

如页面无刷新的进行前后端交互、表单数据项格式认证等。



# 测试案例

## 人员筛选

以下是demo.css的代码：

```
body {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body main {
  width: 400px;
  display: flex;
  flex-flow: column nowrap;
  justify-content: center;
  align-items: center;
  padding: 2rem;
}

body main>div:nth-child(1) {
  width: 50%;
  display: flex;
  justify-content: space-around;
  margin-bottom: 0.5rem;
}

body main>div:nth-child(2) {
  width: 100%;
}

body main>div:nth-child(2) table {
  width: 100%;
}

body main>div:nth-child(2) table, body main>div:nth-child(2) table * {
  border: 1px solid #aaa;
  border-collapse: collapse;
  text-align: center;
}

body main>div:nth-child(2) table caption {
  background-color: #ddd;
  font-weight: bold;
  font-size: 1.2rem;
}

body main>div:nth-child(2) table tbody tr:nth-of-type(odd) {
  background-color: #ddd;
}

body main>div:nth-child(2) table tbody tr:nth-of-type(odd) td {
  cursor: pointer;
}

body main>div:nth-child(2) table tfoot tr td {
  text-align: right;
  padding-right: 1rem;
}

body main>div:nth-child(3) {
  width: 70%;
  display: flex;
  justify-content: center;
  margin-top: 0.5rem;
}
```

以下是demo.html中的代码：

```
<link rel="stylesheet" href="./demo.css">

<body>
    <div id="app">
        <main>
            <div>
                <select v-model="orderRule">
                    <option value="id">id</option>
                    <option value="age">age</option>
                </select>
                <button @click="order($event, 'asc')">升序</button>
                <button @click="order($event, 'desc')">降序</button>
            </div>
            <div>
                <table>
                    <thead>
                        <tr>
                            <th>id</th>
                            <th>name</th>
                            <th>age</th>
                            <th>gender</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="(row, index) in studentArray">
                            <td>{{row.id}}</td>
                            <td>{{row.name}}</td>
                            <td>{{row.age}}</td>
                            <td>{{row.gender}}</td>
                        </tr>
                    </tbody>
                </table>
            </div>
            <div>
                <input type="text" placeholder="按照名字筛选" v-model="keyword">
            </div>
        </main>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    orderRule: "id",
                    keyword: "",
                    studentMessage: [
                        { id: 1, name: "Jack", age: 18, gender: "male" },
                        { id: 2, name: "Tom", age: 19, gender: "male" },
                        { id: 3, name: "Ken", age: 21, gender: "male" },
                        { id: 4, name: "Anna", age: 17, gender: "female" },
                        { id: 5, name: "Mary", age: 22, gender: "female" }
                    ]
                }
            },
            methods: {
                order(event, type) {
                    // studentMessage发生改变,studentArray重新计算
                    if (type === "asc") {
                        this.studentMessage.sort((a, b) => {
                            return a[this.orderRule] - b[this.orderRule]
                        })
                    } else {
                        this.studentMessage.sort((a, b) => {
                            return b[this.orderRule] - a[this.orderRule]
                        })
                    }
                },
            },
            computed: {
                studentArray() {
                    if (!this.keyword) {
                        return this.studentMessage;
                    }
                    return this.studentMessage.filter((row, index, ary) => {
                        for (let v of Object.values(row)) {
                            if (row.name.match(this.keyword)) {
                                return row;
                            }
                        }
                    })
                }
            },
            watch: {
                keyword(newValue, oldValue) {
                    // 重新计算
                    this.studentArray;
                }
            }
        }).mount("#app");
    </script>
</body>
```

最终呈现的效果如下：

![match案例](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/match案例.gif)





## 登录验证

以下是demo.css的代码：

```
body {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  position: relative;
}

body main {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}

body main form {
  display: flex;
  flex-flow: column nowrap;
  justify-content: center;
  align-items: center;
  padding: 0.8rem;
  width: 250px;
  border: 2px solid #ddd;
  position: relative;
}

body main form>div {
  margin-bottom: 0.8rem;
}

body main form>div:first-of-type {
  height: 1rem;
}

body main form>div:first-of-type span {
  font-size: 0.3rem;
  color: red;
}

body main form>div:last-of-type button:nth-of-type(1) {
  width: 6rem;
  margin-right: 0.5rem;
}

body main form>div:last-of-type button:nth-of-type(2), body main form>div:last-of-type button:nth-of-type(3) {
  width: 6rem;
  margin-left: 0.5rem;
}
```

以下是demo.html中的代码：

```
<link rel="stylesheet" href="./demo.css">

<body>
    <div id="app">
        <main>
            <form action="#">
                <div>
                    <span v-if="error">{{error}}</span>
                    <span v-if="!error"></span>
                </div>
                <div>
                    <label for="phone">手机号:</label>
                    <input type="text" id="phone" v-model.number="phone" maxlength="11">
                </div>
                <div>
                    <label for="verify">验证码:</label>
                    <input type="text" id="verify" v-model.trim="verify" maxlength="4">
                </div>
                <div>
                    <button :disabled="!verifyCheck">登录</button>
                    <button v-show="!timer" @click.prevent="sendVerify" :disabled="!phoneCheck">发送验证码</button>
                    <button v-show="timer">已发送 ({{waitSecond}})</button>
                </div>
            </form>
        </main>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    phone: null,
                    phoneCheck: false,
                    verify: null,
                    verifyCheck: false,
                    timer: null,
                    waitSecond: 5,
                    error: null,
                    testVerifyCode: null,
                }
            },
            methods: {
                sendVerify($event) {
                    // 请求后端，模拟验证码的到来
                    this.testVerifyCode = "";
                    for (let i = 0; i < 4; i++) {
                        this.testVerifyCode += String.fromCharCode(Math.floor(Math.random() * (90 - 65 + 1)) + 65);
                    }
                    // 进行计数，再次发送的倒计时
                    this.timer = setInterval(() => {
                        if (this.waitSecond <= 1) {
                            clearInterval(this.timer);
                            this.timer = null;
                            this.waitSecond = 5;
                        } else {
                            this.waitSecond--;
                        }
                    }, 1000)
                }
            },
            watch: {
                phone(newValue, oldValue) {
                    // 前端验证手机号，发送请求给后端
                    if (!(Number.isInteger(this.phone) && String(this.phone).length === 11)) {
                        // 如果没有输入，代表没有错误
                        if (String(this.phone).length === 0) {
                            this.error = null;
                            // 否则就有错误
                        } else {
                            this.error = "手机号不合法";
                        }
                        this.phoneCheck = false;
                    }
                    else {
                        this.error = null;
                        this.phoneCheck = true;
                    }
                },
                verify(newValue, oldValue) {
                    // 请求后端进行验证
                    if (!(this.verify.toUpperCase() === this.testVerifyCode.toUpperCase())) {
                        // 如果没有输入，代表没有错误
                        if (this.verify.length === 0) {
                            this.error = null;
                        }
                        // 否则就有错误
                        else {
                            this.error = "验证码错误";
                        }
                        this.verifyCheck = false;
                    } else {
                        this.error = null;
                        this.verifyCheck = true;
                    }
                }
            }
        }).mount("#app");
    </script>
</body>
```

最终呈现的效果如下：

![watch验证码案例](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/watch验证码案例.gif)