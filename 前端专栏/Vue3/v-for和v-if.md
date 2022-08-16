# 流程控制

## v-for

通过v-for可快速的进行循环，从而能够使开发人员更方便的构建HTML结构。

如下示例，我们通过v-for指令迅速的生成了一份学生名单：

![image-20210913210515628](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210913210515628.png)

代码示例：

```
<style>
    table,
    table * {
        border: 1px solid #aaa;
        border-collapse: collapse;
        width: 400px;
        text-align: center;
    }

    table caption {
        background-color: #ddd;
        font-weight: bold;
        font-size: 1.2rem;
    }

    table tbody tr:nth-of-type(odd) {
        background-color: #ddd;
    }

    table tbody tr td {
        cursor: pointer;
    }

    table tfoot tr td {
        text-align: right;
        padding-right: 1rem;
    }
</style>

<body>
    <div id="app">
        <table>
            <caption>student message</caption>
            <thead>
                <tr>
                    <th>name</th>
                    <th>age</th>
                    <th>gender</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="row in studentMessage" :key="row.id">
                    <td>{{row.name}}</td>
                    <td>{{row.age}}</td>
                    <td>{{row.gender}}</td>
                </tr>
            </tbody>
            <tfoot>
                <tr>
                    <td colspan="3">statistics time:1998-08-02</td>
                </tr>
            </tfoot>
        </table>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    studentMessage: [
                        { id: 1, name: "Jack", age: 18, gender: "male" },
                        { id: 2, name: "Tom", age: 21, gender: "male" },
                        { id: 3, name: "Mary", age: 17, gender: "female" },
                        { id: 4, name: "Anna", age: 18, gender: "female" }
                    ]
                }
            }
        }).mount("#app");
    </script>
</body>
```





## 遍历数组

对数组的遍历，可使用for/in来进行操作，如下所示：

```
<ul>
	<li v-for="(value, index) in ary">{{value}} - {{index}}</li>
</ul>
```

也可以进行for/of操作，效果同for/in相同：

```
<ul>
	<li v-for="(value, index) of ary">{{value}} - {{index}}</li>
</ul>
```

除此之外，你也可以先对数组创建出一个迭代器后再对其进行遍历操作，如kes()、values()、entries()等这些方法都是可行的：

```
<ul>
	<li v-for="(value, index) of ary.entries()">{{value}} - {{index}}</li>
</ul>
```



## 遍历对象

对对象的遍历，可使用for/in来进行操作，如下所示：

```
<ul>
    <li v-for="(value, key, index) in obj">{{value}} - {{key}} - {{index}}</li>
</ul>
```

也可以进行for/of操作，效果同for/in相同：

```
<ul>
    <li v-for="(value, key, index) of obj">{{value}} - {{key}} - {{index}}</li>
</ul>
```

除此之外，你也可以先对对象创建出一个迭代器后再对其进行遍历操作，如Object.kes()、Object.values()、Object.entries()等这些方法都是可行的：

```
<ul>
    <li v-for="(value, index) in Object.entries(obj)">{{value}} - {{index}}</li>
</ul>
```



## key属性

在使用v-for指令时，你应当同时为元素指定一个key属性，并且该属性必须和被渲染的值具有一对一的关系，这样做可以提升后期的操作效率。

其实Vue并不会直接操纵DOM对象，在Vue与真实DOM之间存在着一层虚拟DOM，Vue的操作会先映射到虚拟DOM上，然后再由虚拟DOM映射到真实DOM上。

我们以操纵数组为例：

```
<ul>
	<li v-for="(value, index) in ary">{{value}} - {{index}}</li>
</ul>
```

如果你在使用v-for渲染数组时指定了key属性，它将会依照diff算法将生成的标签在虚拟DOM中直接进行插入，这是非常高效的，它不会引起后面元素的位置后移：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210913213635577.png" alt="image-20210913213635577" style="zoom:80%;" />

如果你未在使用v-for渲染数组时指定key属性，那么在插入新元素时将会引起后面元素的位置后移，在元素较多时会拉低操纵效率：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210913213717735.png" alt="image-20210913213717735" style="zoom:80%;" />

我们用一个简单的例子来进行举例，下面有4个radio单选框，我们先选中值为D的单选框，然后再D的前面插入一个值为C的单选框。

首先是未指定key属性的情况，可以发现这个时候出现了一个BUG，本来我们选中的是D，但是插入C后它选中的变成了C：

![v-for无key](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/v-for无key.gif)

接下来我们指定key属性，就可以避免这个BUG了：

![v-for有key1](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/v-for有key1.gif)

代码如下：

```
<body>
    <div id="app">
        <button @click.once="addElement($event)" style="margin-bottom: 1rem;">add radio C</button>
        <div v-for="(value, index) in ary" :key="value">
            <label :for="value+index">
                <input :id="value+index" type="radio" name="select" :value="value">
                {{value}}
            </label>
        </div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    ary: ["A", "B", "D", "E"]
                }
            },
            methods: {
                addElement(event) {
                    this.ary.splice(2, 0, "C")
                }
            }
        }).mount("#app");
    </script>
</body>
```

因此，在使用v-for进行循环时我们一定要注意添加key属性，推荐使用id作为key的值，不推荐使用index，因为index总是能够发生变化，不够稳定。



# 分支循环

## v-if

v-if指令的条件如果为true，那么将渲染该标签及其内部的子标签，如果为false时则不会进行渲染。

它类似于v-show，但是实际上两者之前还有很大的差别，可以暂时先这么认为：

```
<body>
    <div id="app">
        <div v-if="condition">{{condition}}</div>
        <div v-if="!condition">{{condition}}</div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    condition: true
                }
            }
        }).mount("#app");
    </script>
</body>
```



## v-else

v-else指令应该和v-if指令连用。效果与v-if相反。

这两组指令一个代表如果怎样就渲染，一个代表否则怎样就渲染。

```
<body>
    <div id="app">
        <div v-if="condition">{{condition}}</div>
        <div v-else>{{condition}}</div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    condition: true
                }
            }
        }).mount("#app");
    </script>
</body>
```



## v-else-if

v-else-if是额外的判断条件，当有v-if/v-else/v-else-if在时，只会执行其中的一条。

```
<body>
    <div id="app">
        <div v-if="grades >= 80">成绩优秀</div>
        <div v-else-if="grades >= 60">成绩及格</div>
        <div v-else-if="grades >= 40">成绩一般</div>
        <div v-else="grades > 90">成绩较差</div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    grades: 80
                }
            }
        }).mount("#app");
    </script>
</body>
```





## v-if和v-show

v-if和v-show的关键区别如下：

- v-if在条件为false时，标签会进行销毁，条件为true时会重新进行创建，整个过程会重新渲染DOM
- v-show在条件为false时，标签不会进行销毁而是通过css进行隐藏，整个过程不会重新渲染DOM

综合来看，v-show的性能要高于v-if，但是当出现多个组件频繁互相切换的常见，还是推荐使用v-if，因为它的代码逻辑会更清晰。





# 版本差异

## Vue2中的key

在Vue2中，我们的真实DOM在某些情况下的渲染结果与我们料想之中的渲染结果存在一些偏差。

如下所示，我们定义了2个&lt;input&gt;框，用户可以自己选择进行手机登录还是邮箱登录。

但是当用户在一个&lt;input&gt;框中输入值后再进行切换，就会出现窜值的情况（即不同的输入框切换会保留一样的值）：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/v-if的key1.gif" alt="v-if的key1" style="zoom:60%;" />

造成这种结果的原因是由于虚拟DOM的渲染复用，虚拟DOM进行渲染时会尽量复用已存在的组件，而不是创建新的组件。

上述示例中看起来写了两个&lt;input&gt;，其实是被复用了，所以导致窜值问题的出现，使用key属性可解决这个问题：

```
<body>
    <div id="app">
        <input v-if="type == 'phone'" type="text" placeholder="手机注册" key="phone">
        <input v-if="type == 'email'" type="text" placeholder="邮箱注册" key="email">
        <br>
        <label for="phoneReigster">手机
            <input id="phoneReigster" type="radio" v-model="type" value="phone">
        </label>
        <label for="emailReigster">邮箱
            <input id="emailReigster" type="radio" v-model="type" value="email">
        </label>
    </div>
    <script src='https://cdn.bootcdn.net/ajax/libs/vue/2.6.9/vue.js'></script>
    <script>
        "use strict";
        const app = new Vue({
            el: "#app",
            data() {
                return {
                    type: "phone",
                }
            },
        });
    </script>
</body>
```

在Vue3中，v-if已经不需要再手动的添加key了，因为Vue内部会自动的为每个添加了v-if指令的标签生成一个key属性。但是v-for指令中的key依旧是必须的，不要将两者搞混淆了。

![image-20210914142655090](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210914142655090.png)





## 优先级更改

Vue中允许同时在一个元素上使用v-if和v-for指令。

- Vue2：v-for的优先级高于v-if，它会先进行渲染，再决定要不要将渲染结果映射到真实DOM
- Vue3：v-if的优先级高于v-for，它会先查看条件是否满足，再决定要不要进行渲染

我们并不推荐在同一个元素上使用v-if和v-for，这并不是一个良好的编码习惯。

