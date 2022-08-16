# Composition API

Composition API是Vue3中推荐的组件代码书写方式，相较于传统的Options API来说，它能让业务逻辑处理和后期代码维护变的更加简单。

首先我们来看Options API的优缺点，在Options API中，一个组件通常由data()、methods、watch、computed来组成，在这些选项里我们可以将数据和功能进行完美的划分。

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/OptionsAPI.gif" alt="OptionsAPI" style="zoom:67%;" />

但是这样会出现一个问题，随着代码量越来越大，我们对一个功能的追踪也变的越来越困难，因为该功能的不同部分总是分割在了不同的选项中，如我们要追踪关于数据A的所有代码时，需要从data()到methods再至watch中寻找所有数据A出现的地方，这十分的麻烦：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/OptionsAPI缺点.gif" alt="OptionsAPI缺点" style="zoom:67%;" />

而Composition API从根本上解决了这种问题，它针对一个数据开展一条业务线，当你需要寻找与该数据相关的逻辑代码时，它总是一目了然的展现在你的面前，如下图所示，关于数据A的处理都被封装在了一个函数中，不管是data()、methods亦或是watch的逻辑代码都书写在这一个函数中，这对后期维护来讲非常的方便：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/compositionapi演示.gif" alt="compositionapi演示" style="zoom:67%;" />





# setup

## 基本使用

在Composition API中有一个setup()，该函数能够去代替data()、methods、以及watch和computed，你可以将所有该组件能应用到的代码逻辑都写在这个里面，它具有2个参数，props以及context。

让我们通过Composition API书写一个最简单的例子，现在在setup()函数中你不光可以书写属性，还可以书写方法：

```
<body>
    <div id="app">
        <main>
            <span @click="callbackfn">{{message}}</span>
        </main>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            setup(props, context) {
                const message = "hello composition api"
                function callbackfn(event) {
                    console.log("why do you point me?");
                }
                return {
                    message,
                    callbackfn
                }
            }
        })
        app.mount("#app")
    </script>
</body>
```



## this的支持

Composition API和Options API两者可以同时使用，它们是不冲突的。

但是需要注意的是，setup()函数取代了Options API中beforeCreate以及created这2个生命周期钩子函数，在官方文档中你可以找到这一则说明：

![image-20210922161920322](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210922161920322.png)

![image.png](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/bVcUE75-20210922161832812-20210922165753229.png)

这意味着，在setup()函数中你不能使用this访问到data中的数据项，但是可以在data()中通过$options拿到setup()返回的对象：

```
<body>
    <div id="app">
        <main>
            <div>{{dataMessage}}</div>
            <div>{{setupMessage}}</div>
        </main>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            setup(props, context) {
                const setupMessage = "hello composition api"
                // console.log(this.dataMessage);  // Cannot read properties of undefined
                return {
                    setupMessage
                }
            },
            data() {
                const dataMessage = "hello options api"
                console.log(this.$options.setup());  // {setupMessage: 'hello composition api'}
                return {
                    dataMessage
                }
            }
        })
        app.mount("#app")
    </script>
</body>
```





# 响应式数据

## 非响应式支持

Options API中data()返回的数据均是响应式的：

```
<body>
    <div id="app">
        <main>
            <button @click="number++">+</button>
            &nbsp;{{number}}&nbsp;
            <button @click="number--">-</button>
        </main>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                let number = 0
                return {
                    number
                }
            }
        })
        app.mount("#app")
    </script>
</body>
```

而Composition API中setup()返回的数据并不是响应式的：

```
<body>
    <div id="app">
        <main>
            <button @click="number++">+</button>
            &nbsp;{{number}}&nbsp;
            <button @click="number--">-</button>
        </main>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            setup(props, context) {
                let number = 0
                return {
                    number
                }
            }
        })
        app.mount("#app")
    </script>
</body>
```



## ref

ref能够让值类型的数据在Composition API中具有响应式特征，使用前你需要引入它：

```
const { ref } = Vue;
let number = ref(0)
```

它本质上是将这个数据进行了proxy包装，格式如下：

```
proxy({value:0})
```

当我们需要在setup()函数内部修改该值时，必须使用该代理器的value属性进行修改，如：

```
number.value++
```

但是在模板中需要修改该值则可直接进行修改：

```
number++
```

示例如下：

```
<body>
    <div id="app">
        <main>
            <!-- 在setup()函数内部修改 -->
            <button @click="add">+</button>
            <!-- 在模板中进行修改 -->
            <button @click="number++">+</button>
            &nbsp;{{number}}&nbsp;
            <!-- 在setup()函数内部修改 -->
            <button @click="sub">-</button>
            <!-- 在模板中进行修改 -->
            <button @click="number--">-</button>
        </main>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            setup(props, context) {
                const { ref } = Vue;
                let number = ref(0)
                const add = event => {
                    number.value++;
                }
                const sub = event => {
                    number.value--;
                }
                return {
                    number,
                    add,
                    sub
                }
            }
        })
        app.mount("#app")
    </script>
</body>
```



## reactive

reactive能够让引用类型的数据在Composition API中具有响应式特征，使用前你需要引入它：

```
const { reactive } = Vue;
let ary = reactive([1, 2, 3, 4, 5])
```

它本质上是将这个数据进行了proxy包装，格式如下：

```
Proxy {0: 1, 1: 2, 2: 3, 3: 4, 4: 5}
```

如果是Object，则包装后的格式如下：

```
Proxy {name: 'Jack', age: 18, gender: 'male'}
```

我们不管是在模板中，还是在setup()函数中，都可以直接对其进行操作。

示例如下：

```
<body>
    <div id="app">
        <main>
            <!-- 在setup()函数内部修改 -->
            <button @click="push">+</button>
            <!-- 在模板中进行修改 -->
            <button @click="ary.push(ary[ary.length-1]+1)">+</button>
            &nbsp;{{ary}}&nbsp;
            <!-- 在setup()函数内部修改 -->
            <button @click="pop">-</button>
            <!-- 在模板中进行修改 -->
            <button @click="ary.pop()">-</button>
        </main>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            setup(props, context) {
                const { reactive } = Vue;
                let ary = reactive([1, 2, 3, 4, 5])

                const push = event => {
                    const lastValue = ary[ary.length - 1]
                    ary.push(lastValue + 1)
                }
                const pop = event => {
                    ary.pop()
                }

                return {
                    ary,
                    push,
                    pop
                }
            }
        })
        app.mount("#app")
    </script>
</body>
```



## toRefs

有时候我们并不需要return一个完整的对象，而是return一个对象中单独的某个值，这个时候我们必须通过toRefs对对象进行解构赋值，才能够获得响应对象：

```
<body>
    <div id="app">
        <main>
            <ul>
                <li>
                    <span @click="name = name.toUpperCase()">{{name}}</span>
                </li>
                <li>
                    <span @click="age = '*'+age+'*'">{{age}}</span>
                </li>
                <li>
                    <span @click="gender = gender.toUpperCase()">{{gender}}</span>
                </li>
            </ul>
        </main>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            setup(props, context) {
                const { reactive, toRefs } = Vue;
                let message = reactive({
                    name: "Jack",
                    age: 18,
                    gender: "male"
                })
                const { name, age, gender } = toRefs(message)
                return {
                    name,
                    age,
                    gender
                }
            }
        })
        app.mount("#app")
    </script>
</body>
```

它相当于使用ref对每个值进行包裹，所以说在setup()函数内部修改这些被解构出来的值时，需要使用proxy的value属性进行修改：

```
setup(props, context) {
    const { reactive, ref } = Vue;
    let message = reactive({
        name: "Jack",
        age: 18,
        gender: "male"
    })
    return {
        name: ref(message.name),
        age: ref(message.age),
        gender: ref(message.gender)
    }
}
```



## toRef

在我们对对象进行解构赋值时，有可能出现没有的值，这个时候我们得到的结果是undefined。

如果后续对该值进行修改，让其进行变更时也需要进行响应式支持的话，则必须使用toRef进行解构赋值，如下所示：

```
<body>
    <div id="app">
        <main>
            <ul>
                <li>{{name}}</li>
                <li>{{age}}</li>
                <li>{{gender}}</li>
                <li>{{score}}</li>
            </ul>
        </main>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            setup(props, context) {
                const { reactive, toRefs, toRef, ref } = Vue;
                let message = reactive({
                    name: "Jack",
                    age: 18,
                    gender: "male"
                })
                let { name, age, gender } = toRefs(message)
                // 现在获取的对象是undefined，因为message中没有score属性
                let score = toRef(message, "score")

                // 3s后将它修改为100，如果没有使用toRefs对其进行包裹，那么这种变更则不是响应式的
                // 它相当于 let score = ref(message.score) || ref(undefined);
                setTimeout(() => {
                    score.value = 100
                }, 3000)

                return {
                    name,
                    age,
                    gender,
                    score
                }
            }
        })
        app.mount("#app")
    </script>
</body>
```





## 本节陈述

这一小结主要针对Composition API对数据非响应式支持做出的介绍，Vue3中提供了很多种解决方案，最常用的就是上面举例的4种：

- ref：让值类型的数据能够支持响应式操作
- reactive：让引用类型的数据能够支持响应式操作
- toRefs：让解构赋值出的对象和源容器对象之间具备响应式操作
- toRef：针对解构赋值没有的对象支持响应式操作

除开reactive，其他诸如ref、toRefs以及toRef的数据变更都需要使用proxy.value属性进行操作。



# 组件化开发

## props参数

我们都知道，在setup()函数中不能使用this，因此在父子通信时父组件传递给子组件的信息需要子组件使用setup()的props参数进行接收，以下是使用方法：

```
<body>
    <div id="app">
        <cpn :child-recv-data="fatherData"></cpn>
    </div>

    <!-- 子组件模板 -->
    <template id="cpn-tpl">
        <div>
            <span>{{childRecvData}}</span>
        </div>
    </template>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";

        const { ref, reactive, toRefs, toRef } = Vue

        const app = Vue.createApp({
            setup(props, context) {
                const fatherData = ref("father data")
                return {
                    fatherData
                }
            }
        })

        app.component("cpn", {
            template: "#cpn-tpl",
            props: {
                childRecvData: { required: true, type: String }
            },
            setup(props, context) {
                // 这里的props等同于上面的props
                const { childRecvData } = props
                return {
                    childRecvData
                }
            }
        })

        app.mount("#app")

    </script>
</body>
```



## context参数

context参数有3个属性可供调用：

- attrs：相当于no_props的属性继承
- slots：能够获取组件中的插槽
- emit：能够进行自定义事件

首先是context.attrs，如同no_props一样，它能获取组件使用时所元素身上所绑定的非动态属性：

```
<body>
    <div id="app">
        <cpn data-font-size="font-size:16px" data-font-color="color:#fff"></cpn>
    </div>

    <!-- 子组件模板 -->
    <template id="cpn-tpl">
        <div>
            <span>cpn</span>
        </div>
    </template>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";

        const { ref, reactive, toRefs, toRef } = Vue

        const app = Vue.createApp({})

        app.component("cpn", {
            template: "#cpn-tpl",
            setup(props, context) {
                console.log(context.attrs["data-font-size"]);   // font-size:16px
                console.log(context.attrs["data-font-color"]);  // color:#fff
                return {}
            }
        })

        app.mount("#app")

    </script>
</body>
```

其次是context.slots，它能获取组件中的槽位信息：

```
<body>
    <div id="app">
        <cpn>
            <template v-slot:default>
                <span>default slot</span>
            </template>
        </cpn>
    </div>

    <!-- 子组件模板 -->
    <template id="cpn-tpl">
        <div>
            <slot></slot>
        </div>
    </template>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";

        const { ref, reactive, toRefs, toRef, h } = Vue

        const app = Vue.createApp({})

        app.component("cpn", {
            template: "#cpn-tpl",
            setup(props, context) {
                // {__v_isVNode: true, __v_skip: true, type: 'span', props: null, key: null, …}
                console.log(context.slots.default()[0]);
                // 修改槽位中元素样式
                return () => h("div", { style: "color:#aaa" }, [context.slots.default()])
            }
        })

        app.mount("#app")

    </script>
</body>
```

最后是context.emit，它能发送自定义事件，可用于子组件向父组件传递信息，这个是最常用的属性：

```
<body>
    <div id="app">
        <cpn @read-event="callbackfn"></cpn>
    </div>

    <!-- 子组件模板 -->
    <template id="cpn-tpl">
        <div>
            <span>cpn</span>
        </div>
    </template>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";

        const { ref, reactive, toRefs, toRef, h } = Vue

        const app = Vue.createApp({
            setup(props, context) {
                function callbackfn(...params) {
                    console.log(params);
                    // ['child data']
                }
                return {
                    callbackfn
                }
            }
        })

        app.component("cpn", {
            template: "#cpn-tpl",
            setup(props, context) {
                const childData = "child data"
                context.emit("readEvent", childData)
                return {}
            }
        })

        app.mount("#app")
    </script>
</body>
```



## readonly

通过readonly让对象不可更改，这种不可更改是更深层次的限定，比如数组嵌套对象时，你既不能修改外部数组中的内容，也不能修改内部对象中的内容。

下面是readonly简单的使用例子，我们可将它用于组件通信当中，对于一些只能看不能修改的数据非常方便：

```
<body>
    <div id="app">
        {{message}}
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";

        const { ref, reactive, toRefs, toRef, readonly } = Vue

        const app = Vue.createApp({
            setup(props, context) {
                // 只能看，不能改
                const message = readonly(
                    reactive(
                        [
                            { id: 1, name: "Jack", age: 19 },
                            { id: 1, name: "Tom", age: 18 },
                            { id: 1, name: "Mary", age: 21 }
                        ]
                    )
                )
                return {
                    message
                }
            }
        })

        app.mount("#app")

    </script>
</body>
```





## inject与provide

令人激动的消息，相信在之前学习组件通信时，你对props和emit的通信方式心存怨念，认为这样太麻烦了。

不错，有许许多多的人和你具有同样的想法，这不，在Vue3中迎来了更简单好用的组件通信方式，即inject()和provide()。

使用它们就像是使用消息发布订阅系统一样，你只需要在某一个组件上通过provide()发送出一则数据，那么该Vue应用下所有的组件都可以使用inject()来对该数据进行接收。

这意味着兄弟组件、爷孙组件等都可以直接的进行通信了，而不再是将数据一层一层的进行传递。

下面是一个简单的使用案例：

```
<body>
    <div id="app">
        <cpn></cpn>
    </div>

    <!-- 子组件模板 -->
    <template id="cpn-tpl">
        <div>
            <span>{{message}}</span>
        </div>
    </template>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";

        const { ref, reactive, toRefs, toRef, readonly, inject, provide } = Vue

        const app = Vue.createApp({
            setup(props, context) {
                const message = readonly(
                    reactive(
                        [
                            { id: 1, name: "Jack", age: 19 },
                            { id: 1, name: "Tom", age: 18 },
                            { id: 1, name: "Mary", age: 21 }
                        ]
                    )
                )
                // 发布数据，指定key和value
                provide("message", message)
                return {}
            }
        })

        app.component("cpn", {
            template: "#cpn-tpl",
            setup(props, context) {
                // 订阅数据，指定key和defaultValue，如果没有该数据则采用defaultValue
                const message = inject("message", "default value")
                return {
                    message
                }
            }
        })

        app.mount("#app")

    </script>
</body>
```





# 计算属性

## computed

Composition API中的computed使用与Options API中computed的使用已经不同了。

你必须先导入它然后再到setup()中进行定义，示例如下，computed()参数可以是一个function：

```
<body>
    <div id="app">
        <main>
            <div>
                <span>{{number1}}</span>
            </div>
            <div>
                <span>{{number2}}</span>
            </div>
            <div>
                <! -- 修改number1的值，number2会重新进行计算 -->
                <button @click="number1++">number1 + 1</button>
                <br>
                <button @click="number1--">number1 - 1</button>
            </div>
        </main>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const { ref, reactive, computed } = Vue;
        const app = Vue.createApp({
            setup(props, context) {
                let number1 = ref(100);
                let number2 = computed(() => {
                    return number1.value * 2
                })
                return {
                    number1,
                    number2
                }
            }
        })
        app.mount("#app")
    </script>
</body>
```



## get和set

Composition API中的computed()参数也可以是一个Object，该Object允许定义get和set方法，这意味着你可以对computed attribute进行重新赋值。

示例如下：

```
<body>
    <div id="app">
        <main>
            <div>
                <button @click="number++">+</button>
                <span>&nbsp;{{number}}&nbsp;</span>
                <button @click="number--">-</button>
            </div>
        </main>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const { ref, reactive, computed } = Vue;
        const app = Vue.createApp({
            setup(props, context) {
                let _n = ref(100);
                let number = computed({
                    get() {
                        console.log("computed get()");
                        return _n.value;
                    },
                    set(newValue) {
                        console.log(("computed set()"));
                        _n.value = newValue;
                    }
                })
                return {
                    number
                }
            }
        })
        app.mount("#app")
    </script>
</body>
```





# 数据侦听

## watch

同computed一样，如果你想在Composition API中使用watch进行数据监听，则必须先导入后使用。

以下是关于watch最基本的使用，当点击&lt;span&gt;元素时，会触发watch侦听：

```
<body>
    <div id="app">
        <main>
            <span @click="count++">count</span>
        </main>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const { ref, reactive, watch } = Vue;
        const app = Vue.createApp({
            setup(props, context) {
                let count = ref(0);
                // 要监听的属性，回调函数(新值，旧值)
                watch(count, (newValue, oldValue) => {
                    console.log(`count ${oldValue} => ${newValue}`);
                })
                return {
                    count
                }
            }
        })
        app.mount("#app")
    </script>
</body>
```

Compostion API中的watch允许监听对象的某一个属性，这非常的便捷，如下所示我们只侦听ary中最后一位数据项的变化：

```
<body>
    <div id="app">
        <main>
            <ul>
                <li v-for="v in ary">
                    <input type="text" v-if="ary.indexOf(v) === ary.length-1" :value="v"
                        @change="callbackfn(ary.indexOf(v), $event)">
                    <input type="text" v-else :value="v" @change="callbackfn(ary.indexOf(v), $event)">
                </li>
            </ul>
        </main>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const { ref, reactive, watch } = Vue;
        const app = Vue.createApp({
            setup(props, context) {
                let ary = reactive(
                    ["A", "B", "C", "D"]
                );

                function callbackfn(index, event) {
                    ary[index] = event.target.value
                }

                // 第一个参数必须是一个函数，返回你要侦听的对象属性
                // 第二个参数是回调函数(新值，旧值)
                // 如下所示，只侦听ary的最后一个元素变更
                watch(() => ary[ary.length - 1], (newValue, oldValue) => {
                    console.log(`ary last element change : ${oldValue} => ${newValue}`);
                })

                return {
                    ary,
                    callbackfn
                }
            }
        })
        app.mount("#app")
    </script>
</body>
```

Composition API中的watch现在可以更加方便的实现监听多个属性的变化了，相对于Options API中的watch这一点十分强大。

下面这个例子中不管是number1发生改变还是number2发生改变，都将触发watch的回调函数：

```
<body>
    <div id="app">
        <main>
            <p><span @click="number1++">{{number1}}</span></p>
            <p><span @click="number2++">{{number2}}</span></p>
        </main>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const { ref, reactive, watch } = Vue;
        const app = Vue.createApp({
            setup(props, context) {
                let number1 = ref(100);
                let number2 = ref(100);
                // watch(数组<监听对象1，监听对象2>, 回调函数(数组<监听对象1新值，监听对象1旧值>, 数组<监听对象2新值，监听对象2旧值>)=>{})
                watch([number1, number2], (
                    (
                        [number1NewValue, number1OldValue],
                        [number2NewValue, number2OldValue]
                    ) => {
                        console.log(`number1 change ${number1NewValue} ${number1OldValue}`)
                        console.log(`number2 change ${number2NewValue} ${number2OldValue}`)
                    }
                ))
                return {
                    number1,
                    number2
                }
            }
        })
        app.mount("#app")
    </script>
</body>
```

watch中允许传入第三个参数配置对象，如下示例:

```
<body>
    <div id="app">
        <main>
            {{message}}
        </main>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const { ref, reactive, watch } = Vue;
        const app = Vue.createApp({
            setup(props, context) {
                let message = reactive(
                    [
                        { id: 1, name: "Jack", age: 19 },
                        { id: 1, name: "Tom", age: 18 },
                        { id: 1, name: "Mary", age: 21 }
                    ]
                )
                watch(message, ((newValue, oldValue) => {
                    console.log(`message change ${oldValue} => ${newValue}`);
                }
                ), {
                    // 及早侦听，默认为false，如果为true，它将会在页面一打开就触发callbackfn，而不是在数据发生变更时才触发callbackfn
                    // 默认的watch为false，即惰性侦听，只有在在数据发生变更时才触发callbackfn
                    immediate: true,
                    // 深度侦听，默认为true， 即当多层对象嵌套时它会侦听所有对象内部的变化，而不仅仅是一层
                    deep: true
                })
                return {
                    message,
                }
            }
        })
        app.mount("#app")
    </script>
</body>
```



## watchEffect

Composition API中新增了watchEffect侦听。

它与watch侦听的区别在于：

- watchEffect是对当前setup()函数下所有数据的全局侦听，而watch只能侦听一个或者多个，需要我们手动进行配置
- watchEffect的侦听回调函数是没有参数的，而watch侦听的回调函数是具有参数的
- watchEffect的侦听是及早侦听，而watch的侦听默认是惰性侦听（可通过配置项配置为及早侦听）

如下示例，我们使用watchEffect侦听当前setup()函数中所有数据的变化：

```
<body>
    <div id="app">
        <main>
            <table>
                <thead>
                    <tr>
                        <th>name</th>
                        <th>age</th>
                        <th>gender</th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td>
                            <input type="text" v-model="name">
                        </td>
                        <td>
                            <input type="number" v-model="age">
                        </td>
                        <td>
                            <select v-model="gender">
                                <option value="male">male</option>
                                <option value="female">female</option>
                            </select>
                        </td>
                    </tr>
                </tbody>
            </table>
        </main>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const { ref, reactive, watchEffect } = Vue;
        const app = Vue.createApp({
            setup(props, context) {
                let name = ref("");
                let age = ref(0);
                let gender = ref("male");
                // 会监听name、age、gender。
                // 只能拿到当前的值，不能拿到之前的值
                watchEffect(() => {
                    console.log("start watchEffect");
                    console.log(name.value);
                    console.log(age.value);
                    console.log(gender.value);
                })
                return {
                    name,
                    age,
                    gender
                }
            }
        })
        app.mount("#app")
    </script>
</body>
```



# 其他知识

## 钩子函数变更

在Options API中如果你需要定义生命周期钩子函数，则只需要新增对应的选项即可，如：

```
"use strict";
const app = Vue.createApp({
    beforeCreate(){
        console.log("beforeCreate");
    },
    created(){
        console.log("created");
    }
})
app.mount("#app")
```

而在Composition API中，你必须先导入这些钩子函数，然后在setup()函数中对它们进行使用，注意导入时需要加上前缀on，如下所示：

```
"use strict";
const { onBeforeMount, onMounted } = Vue;
const app = Vue.createApp({
    setup(props, context) {
        onBeforeMount(() => {
            console.log("beforeMount");
        })
        onMounted(() => {
            console.log("mounted");
        })
    }
})
app.mount("#app")
```

官方例举了它们详细的变更记录，如下表所示：

| Options API     | Composition API             |
| --------------- | --------------------------- |
| beforeCreate    | 没有了，被setup()函数取代了 |
| created         | 没有了，被setup()函数取代了 |
| beforeMount     | onBeforeMount               |
| mounted         | onMounted                   |
| beforeUpdate    | onBeforeUpdate              |
| updated         | onUpdated                   |
| beforeUnmount   | onBeforeUnmount             |
| unmounted       | onUnmounted                 |
| errorCaptured   | onErrorCaptured             |
| renderTracked   | onRenderTracked             |
| renderTriggered | onRenderTriggered           |
| activated       | onActivated                 |
| deactivated     | onDeactivated               |



## 辅助性函数

setup()中可以定义任何数据或者对象，当你的业务非常复杂时，我们也可以定义多个辅助性函数来让代码结构更清晰，如下所示：

```
<body>
    <div id="app">
        <button @click="aData.callbackfn">{{aData.a}}</button>
        <button @click="bData.callbackfn">{{bData.b}}</button>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";


        const { ref, reactive, computed } = Vue;

        // 数据A相关逻辑
        function logicA() {
            const _a = ref("a");
            const a = computed({
                get() {
                    return _a.value
                }
            })
            const callbackfn = () => {
                console.log("hello a");
            }
            return {
                a,
                callbackfn
            }
        }

        // 数据B相关逻辑
        function logicB() {
            const _b = ref("b");
            const b = computed({
                get() {
                    return _b.value
                }
            })
            const callbackfn = () => {
                console.log("hello b");
            }

            return {
                b,
                callbackfn
            }
        }


        const app = Vue.createApp({
            setup(props, context) {
                // 调用辅助性函数
                const aData = reactive(logicA());
                console.log(aData);
                const bData = reactive(logicB());
                return {
                    aData,
                    bData
                }
            }
        })
        app.mount("#app")
    </script>
</body>
```



## 获取真实DOM对象

在某些时候我们需要获取真实的一个DOM对象，该如何做呢？

其实你可以为这个元素绑定一个ref属性，该ref属性指向setup()函数中的一个变量。

然后我们可以通过这个变量的value属性拿到真实的DOM对象，它同样也适用于为组件进行绑定。等同于Vue2中的this.$refs的用法。

整体流程如下所示：

```
<body>
    <div id="app">
        <main>
            <!-- 1.指定需要绑定的变量，这里可以是一个普通的DOM对象，也可以是一个组件的应用 -->
            <span ref="spanNode">span</span>
            <div ref="divNode">div</div>
        </main>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const { ref, reactive, onMounted } = Vue;
        const app = Vue.createApp({
            setup(props, context) {
                // 2. 绑定的变量必须通过ref进行包裹
                let spanNode = ref(null);
                let divNode = ref(null);
                // 3.接下来你就可以通过value属性拿到DOM元素或者组件本身
                onMounted(() => {
                    {
                        console.log(spanNode.value);  // <span>span</span>
                        console.log(divNode.value);  // <div>div</div>
                    }
                })
                // 你必须将它们返回出去
                return {
                    spanNode,
                    divNode
                }
            }
        })
        app.mount("#app")
    </script>
</body>
```

