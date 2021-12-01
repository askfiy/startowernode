# mustache

## 名称由来

mustache本意为胡子的意思，因其Vue中模板渲染语法格式长的比较像2片胡子，故便有了这样的名字：

```
{{变量}}
```

当使用mustache语法对模板进行渲染操作时，Vue内部会查找数据层中是否具有该变量，如果有则将该变量的值替换到模板上。

它与后端语言中的模板渲染非常的相似，如果你有前后端混合式开发的经验，那么掌握它应该再轻松不过了。

初学者应当注意一点，只有在被挂载的元素下才可使用mustache语法，如果在被挂载元素之外使用mustache语法是不生效的。



## 内容渲染

如下所示，我们通过mustache渲染出一个基本的个人信息：

![image-20210911172545954](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210911172545954.png)

```
<body>
    <div id="app">
        <div>name : {{information.name}}</div>
        <div>age : {{information.age}}</div>
        <div>gender : {{information.gender}}</div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    information: {
                        name: "Jack",
                        age: 19,
                        gender: "male"
                    }
                }
            }
        }).mount("#app");
    </script>
</body>
```







## 四则运算

在mustache语法中，可以进行四则运算，如下所示：

![image-20210911172642962](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210911172642962.png)

```
<body>
    <div id="app">
        <div>{{x + y}}</div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    x : 100,
                    y : 200
                }
            }
        }).mount("#app");
    </script>
</body>
```





## 调用方法

mustache语法中也能够调用方法，总之\{\{\}\}中可以进行任何操作，他都会向当前Vue应用去请求获取这些属性或方法：

![image-20210911172821614](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210911172821614.png)

```
<body>
    <div id="app">
        <div>{{show()}}</div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            methods: {
                show() {
                    return "Yes";
                }
            }
        }).mount("#app");
    </script>
</body>
```





## 响应式渲染

当Vue中的属性发生更改时，会响应到页面中，此时Vue会重新渲染DOM，响应式的前端框架也是Vue的一大特色：![vue响应式渲染](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/vue响应式渲染.gif)

代码如下：

```
<body>
    <div id="app">
        <div>{{ name }}</div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data(){
                return {
                    name : "Jack"
                }
            }
        }).mount("#app");
    </script>
</body>
```





# 版本差异

## 废弃的过滤器

在Vue2的版本中提供了过滤器的选项，旨在让模板的渲染具有更多的可能性。

但由于功能过于鸡肋，于是在Vue3版本中移除了。



## 全局过滤器

全局过滤器可以由所有的的Vue组件使用。它应当定义在Vue.filter()中，且是一个函数，使用mustache语法时加上|即可主动完成调用。

```
<body>
    <div id="app">
        {{currentTime | timeFormat}}
    </div>

    <script src='https://cdn.bootcdn.net/ajax/libs/vue/2.6.9/vue.js'></script>
    <script src="https://cdn.bootcss.com/moment.js/2.24.0/moment.min.js"></script>
    <script>
        "use strict";
        Vue.filter("timeFormat", value => {
            // value就是 | 左边的值
            return moment(value).format("YYYY-MM-DD HH:mm:ss");
        })
        const app = new Vue({
            el: "#app",
            data() {
                return { currentTime: new Date() }
            }
        })
    </script>
</body>
```



## 局部过滤器

局部过滤器仅能由当前的的Vue组件使用。它应当定义在Vue示例中，关键字为filters，且是一个对象，使用mustache语法时加上|即可主动调用filters下的方法。

```
<body>
    <div id="app">
        {{currentTime | timeFormat}}
    </div>

    <script src='https://cdn.bootcdn.net/ajax/libs/vue/2.6.9/vue.js'></script>
    <script src="https://cdn.bootcss.com/moment.js/2.24.0/moment.min.js"></script>
    <script>
        "use strict";
        const app = new Vue({
            el: "#app",
            data() {
                return { currentTime: new Date() }
            },
            filters: {
                timeFormat(value) {
                    // value就是 | 左边的值
                    return moment(value).format("YYYY-MM-DD HH:mm:ss");
                }
            }
        })

    </script>
</body>
```



## 过滤器参数

过滤器函数中的第一个参数固定死为|左边的值。

当我们有多个参数时，可以进行如下的传递方式，下面的示例将计算七天后的时间是多久：

```
<body>
    <div id="app">
        {{currentTime | addTime(7,"days")}}
    </div>

    <script src='https://cdn.bootcdn.net/ajax/libs/vue/2.6.9/vue.js'></script>
    <script src="https://cdn.bootcss.com/moment.js/2.24.0/moment.min.js"></script>
    <script>
        "use strict";
        const app = new Vue({
            el: "#app",
            data() {
                return { currentTime: new Date() }
            },
            filters: {
                addTime(value, ...arg) {
                    // value就是 | 左边的值,arg是右边的两个值
                    return moment(value).add(...arg).format("YYYY-MM-DD HH:mm:ss");
                }
            }
        })
    </script>
</body>
```
