# 事件监听

## v-on

通过v-on可对DOM元素进行事件监听，当用户某一行为触发后将自动执行mthods中定义的方法。

语法格式如下，当无参数进行传递时，可省略括号的书写：

```
<button v-on:事件=回调函数(参数)>点我</button>
```

下面是一个小案例：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/v-on事件监听.gif" style="zoom:80%;" />

代码示例：

```
<body>
    <div id="app">
        <div>
            <button v-on:click="add">+</button>
            <span>&nbsp;{{number}}&nbsp;</span>
            <button v-on:click="sub">-</button>
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
            methods: {
                add() {
                    this.number++;
                },
                sub() {
                    this.number--;
                }
            }
        }).mount("#app");
    </script>
</body>
```



## 简写形式

我们可以省略v-on:的前缀，使用@来进行代替，如下所示：

```
<button @事件=回调函数(参数)>点我</button>
```

注意区分与v-bind的区别：

```
<button @事件=回调函数(参数)>点我</button>
<div :data-desc="attribute">???</div>
```





# 事件对象

## 无参调用与事件对象

当没有参数进行传递时，回调函数默认接收一个形参即Event事件对象：

```
<body>
    <div id="app">
        <button @click="callback">click me</button>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            methods: {
                callback(event) {
                    console.log(event);
                }
            }
        }).mount("#app");
    </script>
</body>
```



## $event与事件对象

当有参数进行传递时，我们必须手动传入事件对象，且事件对象必须定义为$event：

```
<body>
    <div id="app">
        <button @click="callback($event, 1, 2, 3)">click me</button>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            methods: {
                callback(event, ...args) {
                    console.log(event);
                    console.log(args);
                }
            }
        }).mount("#app");
    </script>
</body>
```



# 事件修饰符

## .once

使用.once修饰符后，该事件只会监听一次。当执行完这一次动作后将取消监听：

```
<body>
    <div id="app">
        <button @click.once="callback($event, 'hello')">click me</button>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            methods: {
                callback(event, str) {
                    console.log(str);
                }
            }
        }).mount("#app");
    </script>
</body>
```



## .prevent

使用.prevent修饰符来阻止默认事件的发生：

```
<body>
    <div id="app">
        <a @click.prevent="callback($event, 'hello')" href="http://www.google.com/">go to google search</a>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            methods: {
                callback(event, str) {
                    console.log(str);
                }
            }
        }).mount("#app");
    </script>
</body>
```



## .stop

使用.stop修饰符阻止事件冒泡：

```
<style>
    .father {
        width: 200px;
        height: 200px;
        background-color: #ddd;
    }

    .son {
        width: 100px;
        height: 100px;
        background-color: #ccc;
    }
</style>

<body>
    <div id="app">
        <div @click="fatherCallback" class="father">
            <!-- 父亲事件不会触发 -->
            <div @click.stop="sonCallback" class="son"></div>
        </div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            methods: {
                fatherCallback(event) {
                    console.log("father callback");
                },
                sonCallback(event) {
                    console.log("son callback");
                }
            }
        }).mount("#app");
    </script>
</body>
```



## .self

使用.self修饰符，也可以防止事件冒泡。只有点击到自己时才触发，不会通过冒泡触发：

```
<style>
    .father {
        width: 200px;
        height: 200px;
        background-color: #ddd;
    }

    .son {
        width: 100px;
        height: 100px;
        background-color: #ccc;
    }
</style>

<body>
    <div id="app">
        <!-- 点儿子时父亲的同名事件回调函数不会触发 -->
        <div @click.self="fatherCallback" class="father">
            <div @click="sonCallback" class="son"></div>
        </div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            methods: {
                fatherCallback(event) {
                    console.log("father callback");
                },
                sonCallback(event) {
                    console.log("son callback");
                }
            }
        }).mount("#app");
    </script>
</body>
```



## .captrue

使用.capture，开启事件捕获。当子元素点击事件后，事件被派发到父元素时，会先执行完父元素监听事件的回调函数，再执行子元素监听事件的回调函数。

注意区分事件冒泡与事件捕获：

- 事件捕获：在子元素事件发生后先运行父元素的同事件回调函数、再运行子元素的事件回调函数

- 事件冒泡：在子元素事件发生后先运行子元素的事件回调函数、再运行父元素的同事件回调函数

代码示例：

```
<style>
    .father {
        width: 200px;
        height: 200px;
        background-color: #ddd;
    }

    .son {
        width: 100px;
        height: 100px;
        background-color: #ccc;
    }
</style>

<body>
    <div id="app">
        <div @click.capture="fatherCallback" class="father">
            <div @click="sonCallback" class="son"></div>
        </div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            methods: {
                fatherCallback(event) {
                    console.log("father callback");
                },
                sonCallback(event) {
                    console.log("son callback");
                }
            }
        }).mount("#app");
    </script>
</body>
```







# 鼠标修饰符

鼠标操作也有常见的修饰符，如下所示：

- .left：按下左键
- .right：按下右键
- .middle：按下中间

案例释义：

```
<style>
    .area {
        width: 200px;
        height: 200px;
        display: inline-block;
        background-color: #ddd;
        margin-left: 1rem;
        text-align: center;
        line-height: 200px;
    }
</style>

<body>
    <div id="app">
        <!-- 按下鼠标左键 -->
        <div class="area" @mousedown.left="callback">1</div>
        <!-- 按下鼠标右键 -->
        <div class="area" @mousedown.right="callback">2</div>
        <!-- 防止打开鼠标右键弹窗 -->
        <div class="area" @contextmenu.prevent @click="callback">3</div>
        <!-- 防止左键双击文本选中 -->
        <div class="area" @selectstart.prevent @click="callback">4</div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            methods: {
                callback(event) {
                    console.log("mouse callback");
                }
            }
        }).mount("#app");
    </script>
</body>
```



# 键盘修饰符

使用keyup和keydown来进行键盘事件的监听。

- keyup：按键按下
- keydown：按键松开

键盘操作也有常见的修饰符，如下所示：

- .enter：按下回车
- .tab：按下制表符
- .delete：按下删除或者退格
- .esc：按下退出
- .ctrl：按下CTRL按键
- .alt：按下ALT按键
- .shift：按下SHIFT按键

除此之外，你也可以使用其他的修饰符，由于键盘按键较多这里不再进行例举。

如下示例：

```
<body>
    <div id="app">
        <!-- 按下a+x -->
        <input class="area" @keyup.a.x="callback">
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            methods: {
                callback(event) {
                    console.log("keyboard callback");
                }
            }
        }).mount("#app");
    </script>
</body>
```



# 修饰符连用

修饰符可以连用，但是连用时一定要注意它们的顺序。

如下所示：

```
<button @click.once.stop=func>点我</button>
```

这个例子的意思非常明显，只执行一次，阻止默认事件。