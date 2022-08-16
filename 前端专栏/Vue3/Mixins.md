# 基本介绍

Mixins机制是Vue2中经常使用的一种手段，Vue3中也同样支持，但是使用的场景越来越少了。

它意在将多个组件中具有相同内容的地方进行提炼，如下所示，我们的cpn1和cpn2的data以及methods都是相同的，因此可以将这些共有的部分提炼成一个Mixin对象：

![mixins基本使用](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/mixins基本使用.gif)

代码示例：

```
<body>
    <div id="app">
        <cpn1></cpn1>
        <cpn2></cpn2>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const common = {
            data() {
                return {
                    message: "hello world"
                }
            },
            methods: {
                change(event) {
                    this.message = this.message.toUpperCase()
                }
            }
        }
        const app = Vue.createApp({
            components: {
                cpn1: {
                    mixins: [common,],
                    template: "<div><div @click='change'>cpn1-{{message}}</div></div>"
                },
                cpn2: {
                    mixins: [common,],
                    template: "<div><div @click='change'>cpn2-{{message}}</div></div>"
                }
            }
        }).mount("#app")
    </script>
</body>
```





# 属性覆盖

如果组件中拥有和Mixin对象中的同名属性或方法，那么只会运行组件中的属性或方法。

![image-20210919143903255](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210919143903255.png)

代码示例：

```
<body>
    <div id="app">
        <cpn1></cpn1>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const common = {
            data() {
                return {
                    message: "hello mixin"
                }
            }
        }
        const app = Vue.createApp({
            components: {
                cpn1: {
                    mixins: [common,],
                    template: "<div><div>{{message}}</div></div>",
                    data() {
                        return {
                            message: "hello cpn1"
                        }
                    }
                }
            }
        }).mount("#app")
    </script>
</body>
```





# 钩子函数

如果Mixin对象中定义了生命周期钩子函数，而组件中也定义了同名的生命周期钩子函数，那么会先运行Mixin对象的生命周期钩子函数，再运行组件自己的生命周期钩子函数。

![image-20210919144318343](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210919144318343.png)

代码示例：

```
<body>
    <div id="app">
        <cpn1></cpn1>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const common = {
            created(){
                console.log("mixin created running");
            }
        }
        const app = Vue.createApp({
            components: {
                cpn1: {
                    mixins: [common,],
                    template: "<div><div>cpn</div></div>",
                    created(){
                        console.log("cpn created running");
                    }
                }
            }
        }).mount("#app")
    </script>
</body>
```





# 普通属性

如果Mixin对象中定义了一个普通属性，那么在组件中想要使用这个普通属性时需要加上$options的前缀符进行访问，因为它不会将Mixin的普通属性放入到data中：

```
<body>
    <div id="app">
        <cpn1></cpn1>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const common = {
            message : "hello mixin"
        }
        const app = Vue.createApp({
            components: {
                cpn1: {
                    mixins: [common,],
                    template: "<div><div>{{this.$options.message}}</div></div>",
                }
            }
        }).mount("#app")
    </script>
</body>
```

实际上$options会将所有的组件中的普通属性进行存入，如下所示：

```
"use strict";
const common = {
    message : "hello mixin"
}
const app = Vue.createApp({
    components: {
        cpn1: {
            mixins: [common,],
            template: "<div><div>{{this.$options.message}}</div></div>",
            cpnName : "cpn1",
            mounted(){
                console.log(this.$options.cpnName);
                // cpn1
            }
        }
    }
}).mount("#app")
```

如果Mixin对象和组件都定义了同名的普通属性，那么会优先选择组件中的同名属性。

我们可以通过配置项来更改这个优先级，用于决定组件到底使用哪一个普通属性，如下所示：

```
<body>
    <div id="app">
        <cpn1></cpn1>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const common = {
            message: "hello mixin"
        }
        const app = Vue.createApp({
            components: {
                cpn1: {
                    mixins: [common,],
                    template: "<div><div>{{this.$options.message}}</div></div>",
                    message: "hello cpn"
                }
            }
        })
        // 这会作用到内部所有组件中
        app.config.optionMergeStrategies.message = (mixinValue, cpnValue) => {
            // 先返回mixin的，如果没有再返回cpn的
            return mixinValue || cpnValue;
        }
        app.mount("#app")
    </script>
</body>
```



# 全局注册

上面的Mixin对象的注册方式为局部注册，只能在cpn1中生效：

```
const app = Vue.createApp({
    components: {
        cpn1: {
            mixins: [common,],
            template: "<div><div>{{this.$options.message}}</div></div>",
        }
    }
})
```

我们可以将Mixin对象改为全局注册，这样所有的组件中都能访问Mixin对象中的属性以及方法了：

```
<body>
    <div id="app">
        <cpn1></cpn1>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            components: {
                cpn1: {
                    template: "<div><div>{{this.$options.message}}</div></div>",
                }
            }
        })
        // 全局注册
        app.mixin({
            message: "hello mixin"
        })
        app.mount("#app")
    </script>
</body>
```

