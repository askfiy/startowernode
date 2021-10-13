# methods

## 基本使用

methods中定义的方法通常都是搭配事件监听做回调的，当然我们也可以主动的对其进行调用。在使用时需要注意以下2点：

- methods下所定义的方法必须加括号才能完成调用
- methods对象内部的方法如果想调用同一Vue应用下的方法或者属性，可使用this进行调用，Vue内部会通过代理器进行查找

如下示例，我们将计算书籍的总价格：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210912143938367.png" alt="image-20210912143938367" style="zoom:80%;" />

代码示例：

```
<body>
    <div id="app">
        <div>books total price: {{getTotalPrice()}}</div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    bookList: [
                        { name: "JavaScript", price: 69 },
                        { name: "HTML5", price: 43 },
                        { name: "CSS3", price: 58 }
                    ],
                }
            },
            methods: {
                getTotalPrice() {
                    return this.bookList.reduce((pre, cur, index, ary) => {
                        return pre + cur.price;
                    }, 0);
                }
            }
        }).mount("#app");
    </script>
</body>
```





# computed

## 基本使用

总价格更像是一个属性，而并非是一个方法，所以针对需要有复杂计算的数据结果，我们可以在Vue应用中定义computed计算属性，再交由mustache进行渲染。

在使用时需要注意以下2点：

- computed下所定义的方法在mustache渲染时不需要加括号就能调用，因此它更像是一个方法
- computed对象内部的方法如果想调用同一Vue应用下的方法或者属性，可使用this进行调用，Vue内部会通过代理器进行查找

如下示例，我们将计算书籍的总价格：

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210912144320609.png" alt="image-20210912144320609" style="zoom:80%;" />

代码示例：

```
<body>
    <div id="app">
        <div>books total price: {{totalPrice}}</div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    bookList: [
                        { name: "JavaScript", price: 69 },
                        { name: "HTML5", price: 43 },
                        { name: "CSS3", price: 58 }
                    ],
                }
            },
            computed: {
                totalPrice(){
                    return this.bookList.reduce((pre, cur, index, ary)=>{
                        return pre + cur.price
                    }, 0)
                }
            }
        }).mount("#app");
    </script>
</body>
```



## 实现原理

computed的内部实现原理是定义了getter方法实现的，我们可以对它进行复现：

```
"use strict";

let bookShop = {
    bookList: [
        { book_name: "JavaScript", price: 99 },
        { book_name: "CSS3", price: 80 },
        { book_name: "HTML5", price: 72 },
    ],
    get total() {
        return this.bookList.reduce((prev, cur, index, array) => {
            return prev + cur.price;
        }, 0)
    }
};

console.log(bookShop.total);

// 251
```

其实使用Proxy全局对象代理可能会更好一些，如下所示：

```
"use strict";

const Vue = {
    data() {
        return {
            bookList: [
                { name: "JavaScript", price: 69 },
                { name: "HTML5", price: 43 },
                { name: "CSS3", price: 58 }
            ],
        }
    },
    computed: {
        totalPrice() {
            return this.bookList.reduce((pre, cur, index, ary) => {
                return pre + cur.price
            }, 0)
        }
    },
    methods: {
        getTotalPrice() {
            return this.bookList.reduce((pre, cur, index, ary) => {
                return pre + cur.price
            }, 0)
        }
    }
}

const app = new Proxy(Vue, {
    // 包含属性和方法以及计算属性
    dataObject: Object.assign(Vue.data(), Vue.methods, Vue.computed),
    get(obj, attribute) {
        // 如果访问的是属性和方法，则直接返回
        if (attribute in obj.data() || attribute in obj.methods) {
            return this.dataObject[attribute]
        }
        // 如果是计算属性，则自动调用
        else if (attribute in obj.computed) {
            return this.dataObject[attribute]()
        }
    },
    set(_, attribute, value) {
        dataObject[attribute] = value;
        return true;
    }
});

app.bookList[0].price = 100;
console.log(app.bookList);
console.log(app.totalPrice);
console.log(app.getTotalPrice());

// [
//     { name: 'JavaScript', price: 100 },
//     { name: 'HTML5', price: 43 },
//     { name: 'CSS3', price: 58 }
// ]
// 201
// 201
```



## 指定get/set

computed中可以指定一个对象，该对象必须提供get和set方法。

- get：当获取该computed时自动触发
- set：当设置该computed时自动触发，value属性为设置的新值

示例如下，当获取number时实际上获取的是\_n，当设置number时实际上是设置的\_n：

```
<body>
    <div id="app">
        <div>{{number}}</div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    _n: 100
                }
            },
            computed: {
                number: {
                    get() {
                        return this._n
                    },
                    set(value) {
                        this._n = value
                    }
                }
            }
        }).mount("#app");
    </script>
</body>
```



# 两者异同

## 同-响应特性

如果修改了数据源，那么计算属性和方法都会重新进行页面渲染。

如下所示，书籍总价本来为170，当我们修改其中任意一本书的价格后，书籍总价格也将发生改变：



<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/vue-计算属性和方法的计算特性.gif" alt="vue-计算属性和方法的计算特性" style="zoom:80%;" />

代码示例：

```
<body>
    <div id="app">
        <div>computed price: {{totalPrice}}</div>
        <div>methods price: {{getTotalPrice()}}</div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";
        const app = Vue.createApp({
            data() {
                return {
                    bookList: [
                        { name: "JavaScript", price: 69 },
                        { name: "HTML5", price: 43 },
                        { name: "CSS3", price: 58 }
                    ],
                }
            },
            computed: {
                totalPrice() {
                    return this.bookList.reduce((pre, cur, index, ary) => {
                        return pre + cur.price
                    }, 0)
                }
            },
            methods: {
                getTotalPrice() {
                    return this.bookList.reduce((pre, cur, index, ary) => {
                        return pre + cur.price
                    }, 0)
                }
            }
        }).mount("#app");
    </script>
</body>
```



## 异-缓存特性

computed具有缓存特性，即多次调用只会调用一次，只有当数据源发生改变时才会自行调用一次。

而methods没有缓存特性，调用几次就执行几次，并且当数据源发生改变时也会自动调用之前的次数。

如下所示，下面模板中3次调用了computed，实际上它只执行了1次，而methods同样也是3次调用，但是执行了3次，当数据源发生改变后computed会多加1次，而methods会每一个都重新执行1次，总计为新增3次：

![vue-计算属性的缓存特性](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/vue-计算属性的缓存特性.gif)

代码示例：

```
<body>
    <div id="app">
        <div>
            <p>computed total : {{total}}</p>
            <p>computed total : {{total}}</p>
            <p>computed total : {{total}}</p>
            <p><mark>computed call count : {{ccc}}</mark></p>
        </div>
        <hr>
        <div>
            <p>methods getTotal : {{getTotal()}}</p>
            <p>methods getTotal : {{getTotal()}}</p>
            <p>methods getTotal : {{getTotal()}}</p>
            <p><mark>methods call count : {{mcc}}</mark></p>
        </div>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js"></script>
    <script>
        "use strict";

        let computedCallCount = 0;
        let methodsCallCount = 0;

        const app = Vue.createApp({
            data() {
                return {
                    num1: 100,
                    num2: 100,
                    ccc: computedCallCount,
                    mcc: methodsCallCount,
                }
            },
            computed: {
                total() {
                    computedCallCount++;
                    this.ccc = computedCallCount;
                    return this.num1 + this.num2;
                }
            },
            methods: {
                getTotal() {
                    methodsCallCount++;
                    this.mcc = methodsCallCount;
                    return this.num1 + this.num2;
                }
            }
        }).mount("#app");
    </script>
</body>
```

