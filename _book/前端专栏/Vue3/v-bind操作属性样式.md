# 属性绑定

## v-bind

通过v-bind指令可以单向的为模板元素的属性动态绑定一个属性值，在使用时应当注意以下2点：

- 被双引号包裹的属性值没有被任何其他引号包裹时，该值将会被认为是Vue应用中数据层的某一个数据，会在Vue应用中先查找该数据再进行赋值
- 被双引号包裹的属性值同时还被其他引号所包裹时，该属性值将会被认为是一个普通的字符串，会直接进行赋值

如下示例。

1）被双引号包裹的属性值没有被任何其他引号包裹时，将访问data方法中返回的对象里的desc属性：

```
<!-- 渲染结果 -->
<!-- <div data-desc="this is a div">???</div> -->

<body>
    <div id="app">
        <div v-bind:data-desc="desc">???</div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    desc: "this is a div"
                }
            }
        }).mount("#app");
    </script>
</body>
```

2）被双引号包裹的属性值同时还被其他引号所包裹时，该属性值将被当做普通字符串处理：

```
<!-- 渲染结果 -->
<!-- <div data-desc="desc">???</div> -->

<body>
    <div id="app">
        <div v-bind:data-desc="'desc'">???</div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    desc: "this is a div"
                }
            }
        }).mount("#app");
    </script>
</body>
```



## 简写形式

我们可以省略v-bind的前缀，直接采用:进行绑定，如下所示：

```
<div v-bind:data-desc="attribute">???</div>
<div :data-desc="attribute">???</div>
```



# 样式操作

## :class对象操作

将:class的属性值定义为一个对象，若key的value为true，则将该key进行应用，反之则不进行应用。

除此之外，你也可以定义class属性，用于设置一些永远不会改变的样式。

语法格式如下：

```
<div :class={类1:true,类2:false,类3:true} class="固定不变的类">内容</div>
```

在下面的例子中，我们定义了.red、.italic、.opacity三个类，通过:class对象中不同key的状态切换，可以达到不同的样式效果：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/vue-class对象操作.gif" alt="vue-class对象操作" style="zoom:80%;" />

代码示例：

```
<style>
    .red {
        color: red;
    }

    .italic {
        font-style: italic;
    }

    .opacity {
        opacity: 0.5;
    }
</style>

<body>
    <div id="app">
        <div :class={red:redStatus,italic:italicStatus,opacity:opacityStatus}>
            hello world
        </div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    redStatus: true,
                    italicStatus: true,
                    opacityStatus: true,
                }
            }
        }).mount("#app");
    </script>
</body>
```



## :class数组操作

将:class的属性值定义为一个数组，数组中的元素将被当做类名进行应用。

使用场景一般较少，语法格式如下：

```
<div :class=[变量1,变量2,"字符串3"] class="固定不变的类">内容</div>

<!-- 注意，当数组内的元素没有加引号是会认为是一个数据层中的数据，会去当前Vue应用中寻找 -->
<!-- 如果数组内的元素加上引号，则被认为是一个字符串 -->
```

我更喜欢将它这样进行使用：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/vue-class数组操作.gif" alt="vue-class数组操作" style="zoom:80%;" />

代码示例：

```
<style>
    .red {
        color: red;
    }

    .italic {
        font-style: italic;
    }

    .opacity {
        opacity: 0.5;
    }
</style>

<body>
    <div id="app">
        <div :class="styleClass">
            hello world
        </div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    styleClass: ["red", "opacity", "italic"]
                }
            }
        }).mount("#app");
    </script>
</body>
```





## :style对象操作

将:style的属性值定义为一个对象，key为css的样式名，value为css的设置值。

除此之外，你也可以定义style属性，用于设置一些永远不会改变的样式。

注意它的格式是这样的，:style中的key必须为小驼峰式命名法，如果你对key进行了引号包裹，则可以使用-分割命名：

```
<div :style={color:"red",fontSize:"12px",backgroundColor:bgColor} style="不变的样式:值">内容</div>

<!-- 注意!最后的background没有添加单引号，这使得bgColor会当作数据层中的数据去当前Vue应用中获取 -->
```

示例演示：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210911214455881.png" alt="image-20210911214455881" style="zoom:80%;" />

代码示例：

```
<body>
    <div id="app">
        <div :style={background:bgStyle,fontStyle:"italic"} style="text-align: center;">
            hello world
        </div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    bgStyle:"linear-gradient(90deg, #667eea 0%, #764ba2 100%)"
                }
            }
        }).mount("#app");
    </script>
</body>
```



## :style数组操作

除开可以使用对象语法绑定:style属性，也可以通过数组语法进行绑定。

数组中的元素必需是能够从Vue实例中获取到的数据。

示例演示：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210911214706171.png" alt="image-20210911214706171" style="zoom:80%;" />

代码示例：

```
<body>
    <div id="app">
        <div :style=[fontStyle,ColorStyle] style="text-align: center;">
            hello world
        </div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    fontStyle: { "font-size": "24px", "font-style": "italic" },
                    ColorStyle: { "color": "red", "background-color": "green" },
                }
            }
        }).mount("#app");
    </script>
</body>
```