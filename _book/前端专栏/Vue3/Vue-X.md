# Vue-X

## 基本介绍

Vue-X是Vue全家桶中非常重要的一环，它能够使我们更加方便的集中管理组件中的数据，在大型项目中Vue-X的使用频率非常之高，因此Vue-X的学习是必不可少的。

目前Vue-X版本已更新到4.x，我们就以4.x为例了解它的用法。

[官方文档](https://next.vuex.vuejs.org/zh/index.html)

<img src="https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/29002857-9e802f08-7ab4-11e7-9c31-604b5d0d0c19.png" alt="Logos for Vuex and Vue-Router? · Issue #6305 · vuejs/vue · GitHub" style="zoom:8%;" />





## 优势对比

在之前我们通过props与$emit()亦或是provide()和inject()的方式进行组件通信时，所有的数据都是绑定在某个组件内部的，如果其他组件想要使用以及修改这些数据，则必须由数据源组件将数据与修改方法进行下发。

对于props与$emit()通信来说，它的数据传递过程会显得十分繁琐，必须由父组件通过props一层一层的将数据向下进行传递，或者通过$emit()将数据一层一层往上进行传递，如果组件层级过多，则会导致后期代码维护困难的问题出现。

而Vue-X则将公用数据与组件进行剥离，你可以将它理解为是一个数据仓库，组件树中所有的组件都能从这个数据仓库中获得想要的数据、同时组件树中任何一个组件对仓库中数据的修改均可以影响到其它的组件。

![image-20211007152615655](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20211007152615655.png)





## 工作流程

每一个Vux-X你都可以将它理解为一个仓库，而在仓库中，有3个重要的概念：

- actions：行为
- mutations：变化
- state：状态

我们可以看一下这张图：

![image-20211008145724574](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20211008145724574.png)

释义如下：

- vue components可以通过dispatch或者commit直接发生行为（actions）或者触发变化（mutations）
- actions只能是由vue components通过dipatch引起的，且它的作用只能是通过commit触发变化（mutations）
- state只能由mutations进行改变，并且改变后会及时的将数据进行响应并渲染在所有vue components中
- devtools只能监听mutations

换而言之，仓库store中的数据状态（state）会发放给vue components，而当我们想要修改这些数据状态时则必须先通过dispatch发起行为（actions），然后行为会通过commit触发变化（mutations），由变化（mutations）来对仓库中数据的状态（state）进行改变。

或者你可以直接在vue components中通过commit触发变化（mutations），再由变化（mutations）来对仓库中数据的状态（state）进行改变。

不论如何，这是一个单向的工作流程，你不能直接在vue components中修改仓库store中的数据状态（state），因为devtools不会检测这一环节，如果这样做，则会造成数据不可预测的情况发生。



## 主要组成

以下是整个Vue-X中store的组成部分：

- state：一个对象，内部可以包含任意类型的数据，是真正存储数据的地方，而对数据的修改只能通过mutations中定义的方法进行
- mutations：一个对象，内部能包含一系列的方法，这些方法必须是同步的，它主要的作用在于对state中的数据修改提供接口
- actions：一个对象，内部能包含一系列的方法，这些方法可以是同步的或者是异步的，常用于与后端API进行交互，但这些方法不能直接修改state中的数据，必须间接的通过mutations修改state中的数据
- getters：一个对象，内部能包含一系列的方法，这些方法必须是同步的，它类似于computed计算属性，当使用getters对象中的方法时不需要加括号即可完成调用
- modules：一个对象，主要用于在业务复杂时对数据进行模块化分类

现在我们再来看看下面这张图，你就能完全明白它们的功能了：

![image-20211009162955798](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20211009162955798.png)



# 基本使用

## 初始化项目

使用Vite生成项目，跟随指引一步一步的进行初始化：

```
$ npm init @vitejs/app <project-name>
```

进入到项目根目录下，安装Vue-X插件，输入以下命令后它将会安装Vue-X 4.x的最新版本：

```
$ npm install vuex@next --save
```

在src下新建一个store目录，并在其中新建一个index.js文件，该文件将作为Vue-X插件的入口文件存在：

```
$ mkdir ./src/store
$ touch ./src/store/index.js
```

删除.src/components/HelloWorld.vue：

```
$ rm -rf ./src/components/HelloWorld.vue
```

清空.src/App.vue中的代码，并粘贴下面的代码：

```
<template>
  <div>hello world</div>
</template>

<script setup>
</script>

<style>

</style>
```

接下来你就可以启动这个Vue项目了，默认项目的端口是3000：

```
$ npm run dev
```



## 快速上手

要想使用Vue-X插件，我们就必须在.src/store/index.js文件中对其进行引入：

```
// 1.导入创建仓库的方法
import { createStore } from "vuex"

// 2.创建仓库并进行配置
const store = createStore({
    strict: process.env.NODE_ENV !== "production",
    // 定义数据
    state: {
        counter: 65,
    },
    // 定义多样化展示数据的方法
    getters: {
        char(state) {
            return String.fromCharCode(state.counter);
        }
    },
    // 定义修改数据的方法
    mutations: {
        MODIFY(state, playload) {
            state.counter = playload.number;
        }
    },
    // 定义请求后端数据的方法
    actions: {

    },
    // 对数据进行模块化分类
    modules: {

    }
})

// 3.导出仓库
export default store
```

然后需要到.src/main.js中为当前的Vue应用加载这个插件：

```
import { createApp } from 'vue'
import App from './App.vue'
import vueX from "./store/index"

const app = createApp(App);
// 加载插件
app.use(vueX);
app.mount('#app');
```



## 严格模式

根据第一章中的Vue-X工作流程我们可以得知，state的改变只能通过mutations进行，但是这个条件默认情况下并不是严格限制的，所以我们可以对此开启严格模式进行限制。

注意！因为严格模式在项目上线后会造成性能损失，所以这里采用了一个if条件作为判断，如果是开发模式则开启严格模式、如果是生产模式则关闭严格模式：

```
import { createStore } from "vuex"

// 开启严格模式
const store = createStore({
    strict: process.env.NODE_ENV !== "production",
    state: {
        ...
    },
    ...
})

// 3.导出仓库
export default store
```



## 仓库对象

想在组件中使用Vue-X中的数据，我们必须先拿到这个仓库对象。

你可以在组件的模板中直接进行获取：

```
{{ $store }}
```

要想在组件的脚本中使用这个仓库对象你必须先对其进行导入：

```
import { useStore } from "vuex";
```

接下来需要进行实例化操作：

```
const store = new useStore();
```

而后你就可以使用这个仓库对象了，在仓库对象中的任意数据、方法你都可以拿到。

如，拿到store.state下的counter：

```
// template
{{ $store.state.counter }}

// script
console.log(store.state.counter)
```



## state

store.state根据图示可以发现，在组件内部它可以直接进行使用，但是当要修改store.state中的数据时必须通过store.mutations中定义的方法进行。

![image-20211007225235155](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20211007225235155.png)

如，我们可以在直接在模板中拿到store.state.counter：

```
<template>
  <div>
    <span>{{ $store.state.counter }}</span>
  </div>
</template>
```

这个时候页面上将显示65，因为在store.state中我们定义的counter为65：

```
state: {
    counter: 65,
},
```



## getters

store.getters本质和store.state差不多，store.getters与store.state的关系就如同data与computed的关系，它内部所包含的方法必须是同步的。

我们在组件内部可以直接使用store.getters的方法，但是当要修改store.getters中方法所展示的数据时就必须先通过store.mutations修改store.state中的数据，而一旦store.state中的数据发生变化，那么store.getters中方法所展示的数据会也自动进行更新。

![image-20211007225745187](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20211007225745187.png)

如，我们可以在直接在模板中拿到store.getters.char：

```
<template>
  <div>
    <span>{{ $store.state.counter }} -&gt; {{ $store.getters.char }}</span>
  </div>
</template>
```

这个时候页面上将显示65 -&gt; A，因为store.getters.char方法我们定义的是返回该数字的Unicode字符表现形式：

```
getters: {
    char(state) {
        return String.fromCharCode(state.counter);
    }
},
```

注意，store.getters中的方法均可定义一个名为state的形参，它指向了当前的store.state。



## mutations

store.mutations中定义的方法主要是作为修改store.state中所定义的数据而存在的，它内部所包含的方法必须是同步的。

![image-20211007233050646](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20211007233050646.png)

它可以在组件中或者store.actions通过store.commit(type, playload)的方式传递新的数据进入仓库，以便于修改store.state中的数据。

```
store.commit(type, playload)

// type为store.mutations中定义的方法名
// playload可以为store.mutations中定义的方法传递参数，通常情况下它应该是一个对象
```

如下所示：

```
<template>
  <div>
    <div>
      <span>{{ $store.state.counter }} -&gt; {{ $store.getters.char }}</span>
    </div>
    <button @click="generateRandomNumber">change</button>
  </div>
</template>

<script setup>
import { useStore } from "vuex";

const store = new useStore();

const generateRandomNumber = () => {
  const number = Math.floor(Math.random() * (90 - 65 + 1)) + 65;
  store.commit("modify", { number });
};
</script>

<style>
</style>
```

当我们点击&lt;button&gt;按钮后，将触发组件的generateRandomNumber方法，该方法会生成一个65-90之间的随机数，而后会对store进行通知。

当store接到commit()通知后，会在store.mutations中寻找符合type的方法：

```
mutations: {
    modify(state, playload) {
        state.counter = playload.number;
    }
},
```

注意，store.getters中的方法均可定义一个名为state的形参，它指向了当前的store.state。

而playload形参则接受store.commit()所提交的参数，通常情况下我们希望它传递的是一个对象。



## actions

store.actions中定义的方法主要是作为获取backend API中的数据而存在的，它内部所包含的方法可以是同步的，也可以是异步的。

它只能在组件中通过store.dispatch(type, playload, ?options)进行调用。

```
store.dispatch(type, playload, ?option)

// type为store.dispatch中定义的方法名
// playload可以为store.dispatch中定义的方法传递参数，通常情况下它应该是一个对象
// options能够为store.dispatch中定义的方法提供一些选项，通常情况下它的使用较少，可以不必关注
```

注意，store.actions只能获取backend API中的数据，而不能直接修改store.state中定义的数据，我们必须通过store.commit()触发store.mutations下定义的方法，再由它来修改store.state中的数据。

![image-20211007233815030](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20211007233815030.png)

我们用一个最简单的案例来进行说明，下面是一个后端接口，它可以返回一些数据：

```
const Koa = require('koa')
const router = require('koa-router')()
const koaBody = require("koa-body")
const cors = require('@koa/cors')

const server = new Koa()

server.use(cors())
server.use(router.routes())
server.use(koaBody({
    multipart: true
}))

server.use(async (ctx, next) => {
    await next();
    ctx.response.set("Content-Type", "application/json")
})

router.post("/api/books", async (ctx, next) => {
    await next();
    const bookList = [
        { id: 1, name: "红楼梦", price: 199 },
        { id: 2, name: "西游记", price: 159 },
        { id: 3, name: "三国演义", price: 149 },
        { id: 4, name: "水浒传", price: 129 }
    ]
    ctx.response.body = JSON.stringify(
        {
            code: 200,
            message: bookList
        }
    )
})

server.listen(3001, "localhost", 128, () => {
    console.log("server started to http://localhost:3001");
})
```

我们可以在store.actions中定义一个getBooks的方法，它将会对后端/api/books发起请求，然后将bookList通过store.mutations下的addBooks方法添加到store.state中：

```
// 1.导入创建仓库的方法
import { createStore } from "vuex"
import axios from "axios"

// 2.创建仓库并进行配置
const store = createStore({
    strict: process.env.NODE_ENV !== "production",
    // 定义数据
    state: {
        bookList: [],
    },
    // 定义修改数据的方法
    mutations: {
        addBooks(state, playload) {
            state.bookList = playload.bookList
        }
    },
    // 定义请求后端数据的方法
    actions: {
        getBooks(store, playload) {
            axios({
                url: "/api/books",
                baseURL: "http://localhost:3001",
                method: "POST",
                responseType: "json"
            }).then((response) => {
                if (response.data.code === 200) {
                    const bookList = response.data.message
                    store.commit("addBooks", { bookList })
                }
            })
        }
    },
})

// 3.导出仓库
export default store
```

在组件中，我们必须使用store.dispatch()方法才能对actions下的方法发起调用。

而对于网络的请求，一般情况下我们会将其放在onCreated()钩子函数（只获取一次的数据）或者onMounted()钩子函数（每次刷新页面都重新获取的数据）中：

```
<template>
  <div>
    <ul>
      <li v-for="book in $store.state.bookList" :key="book.id">
        {{ book.name }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import { useStore } from "vuex";
import { onMounted } from "vue";

const store = new useStore();
onMounted(() => {
  store.dispatch("getBooks", {});
});
</script>

<style>
</style>
```



## modules

在之前我们总是将所有的数据存放在一个仓库中，这样管理起来比较杂乱：

![image-20211009163121847](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20211009163121847.png)

其实每个不同类别的数据你都可以将它们做成一个独立的模块，你可以将它理解为一个小仓库，最后再汇总到大仓库中：

![image-20211009163044393](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20211009163044393.png)

示例如下：

```
// 1.导入创建仓库的方法
import { createStore } from "vuex"

// 2.定义小仓库
const cart = {
    state: {},
    getters: {},
    mutations: {},
    actions: {},
}

// 3.定义大仓库
const store = createStore({
    strict: process.env.NODE_ENV !== "production",
    modules: {
        cart
    }
})

// 4.导出大仓库
export default store
```

当在使用小仓库下的方法时，你必须加上小仓库的名字：

```
// 获取小仓库中的内容
{{ $store.state.小仓库名字.内容 }}

// 调用小仓库的mutations下的方法
store.commit("小仓库名字/方法名", playload)

// 调用小仓库的actions下的方法
store.dispatch("小仓库名字/方法名", playload)
```



# 实战演示

## 项目一览

现在我们可以用一个小的demo案例来加深对Vue-X的使用影响。

一个简单的购物车案例，当你在主页中选择商品后，它会将商品添加到购物车中并且计算支付总价。

我们需要2个组件：

- Index：展示商品信息、勾选商品
- Shop：展示购物车信息与购物总价格

以下是项目的成品效果：

![Vux-X项目演示](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/Vux-X项目演示.gif)



## 后端代码

后端还是采用Node.js + Koa框架进行搭建，一共只有1个API接口，返回一些商品信息：

```
const Koa = require('koa')
const router = require('koa-router')()
const static = require("koa-static")
const path = require('path')
const koaBody = require("koa-body")
const cors = require('@koa/cors')

const server = new Koa()

const protocol = "http"
const hostname = "localhost"
const port = 3001
const serverAddr = protocol.concat(":", "//", hostname, ":", port);


server.use(cors())
server.use(router.routes())
server.use(static(
    path.join(__dirname, "./static")
))
console.log(path.join(__dirname, "./static"));
server.use(koaBody({
    multipart: true
}))

server.use(async (ctx, next) => {
    await next();
    ctx.response.set("Content-Type", "application/json")
})

router.post("/api/goods", async (ctx, next) => {
    await next();
    const goodsList = [
        { id: 1, name: "北京烤鸭", price: 98, img: serverAddr + "/1.jpg" },
        { id: 2, name: "窝窝头", price: 24, img: serverAddr + "/2.jpg" },
        { id: 3, name: "烤山药", price: 67, img: serverAddr + "/3.jpg" },
        { id: 4, name: "小米儿", price: 18, img: serverAddr + "/4.jpg" }
    ]
    ctx.response.body = JSON.stringify(
        {
            code: 200,
            message: goodsList
        }
    )
})

server.listen(port, hostname, 128, () => {
    console.log(`server started to ${serverAddr}`);
})
```



## 插件代码

对于Vue-X的store目录下的index.js文件，我们可以进行更加详细的拆分。

如将state、getters、mutations、actions、mutations_type以及actions_type还有modules拆分成不同的文件：

```
./src/store
├── actions.js
├── actions_type.js
├── getters.js
├── index.js
├── modules.js
├── mutations.js
├── mutations_type.js
└── state.js
```

以下各个文件中的代码，首先是./src/store/index.js：

```
import { createStore } from "vuex"

import state from "./state"
import getters from "./getters"
import mutations from "./mutations"
import actions from "./actions"
import modules from "./modules"

export default createStore({
    state,
    getters,
    mutations,
    actions,
    modules
})
```

下面是./src/store/state.js：

```
export default {
    // 所有商品信息 
    /*
        [
            {id : 1, name : "phone", price : 98, img : "http://localhost:30001/x.jpg"}
        ]
    */
    goodsList: [],
    // 当前购物车信息
    /*
        [
            {id : 1, name : "phone", price : 98, img : "http://localhost:30001/x.jpg", number : 1}
        ]
    */
    shopList: []
}
```

其次是./src/store/getters.js：

```
export default {
    // 计算购物车总价
    totalPrice(state) {
        return state.shopList.reduce((pre, goods) => {
            return pre + goods.price * goods.number
        }, 0)
    }
}
```

然后是./src/store/mutations_type.js：

```
export default {
    // 初始化所有商品数据
    loadAllGoods: "loadAllGoods",
    // 向购物车新增一个商品
    incrShopList: "incrShopList",
    // 移除购物车中的一个商品
    decrShopList: "decrShopList",
    // 修改购物车中的商品数量
    changeShopGoodsNumber: "changeShopGoodsNumber"
}
```

下面是./src/store/mutations.js：

```
import mutationsType from "./mutations_type"

export default {
    // 初始化所有商品数据
    [mutationsType.loadAllGoods](state, playload) {
        state.goodsList = playload.goodsList
    },
    // 先购物车中新增一个商品
    [mutationsType.incrShopList](state, playload) {
        for (const goods of state.shopList) {
            // 如果存在就数量+1
            if (goods.name === playload.goods.name && goods.id === playload.goods.id) {
                goods.number += 1
                return
            }
        }
        // 不存在、直接添加
        state.shopList.push(playload.goods)
    },
    // 移除购物车中的一个商品
    [mutationsType.decrShopList](state, playload) {
        state.shopList.splice(playload.goodsIndex, 1)
    },
    // 修改购物车中的商品数量
    [mutationsType.changeShopGoodsNumber](state, playload) {
        // 这里始终是用的加法运算，假如你的购物车中有1个商品，那么 1 + -1 就等于 1 - 1
        state.shopList[playload.goodsIndex].number += playload.number
    }
}
```

以及./src/store/actions_type.js：

```
export default {
    // 请求所有商品
    getAllGoods: "getAllGoods"
}
```

还有./src/store/actions.js：

```

import actionsType from "./actions_type"
import mutationsType from "./mutations_type"
import axios from "axios"

export default {
    // 发送请求，并且将请求后的商品信息添加至
    [actionsType.getAllGoods](store, playload) {
        axios({
            baseURL: "http://localhost:3001",
            url: "api/goods",
            method: "POST",
            responseType: "json"
        }).then((response) => {
            if (response.data.code === 200) {
                const goodsList = response.data.message;
                store.commit(mutationsType.loadAllGoods, { goodsList })
            }
        })
    }
}
```

最后是modules.js：

```
export default {

}
```



## 组件代码

定义好Index.vue以及Shop.vue，并在App.vue中进行引入。

下面是./src/App.vue的代码：

```
<template>
  <div id="app-tpl">
    <Index />
    <Shop />
  </div>
</template>

<script setup>
import Index from "./components/Index.vue";
import Shop from "./components/Shop.vue";
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  list-style: none;
}
#app-tpl {
  display: flex;
  flex-flow: column nowrap;
  padding: 2rem;
}
</style>
```

Index.vue的代码：

```
<template>
  <div id="index-tpl">
    <ul>
      <li v-for="goods in $store.state.goodsList" :key="goods.id">
        <span class="goods_name">{{ goods.name }}</span>
        <img
          :src="goods.img"
          class="goods_img"
          @click.self="addShopGoods(goods, $event)"
        />
        <span class="goods_price">￥{{ goods.price }}</span>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { onMounted } from "vue";
import { useStore } from "vuex";

import actionsType from "../store/actions_type";
import mutationsType from "../store/mutations_type";

const store = new useStore();

onMounted(() => {
  store.dispatch(actionsType.getAllGoods, {});
});

const addShopGoods = (goods, event) => {
  store.commit(mutationsType.incrShopList, { goods: { ...goods, number: 1 } });
};
</script>



<style scoped>
#index-tpl {
  width: 100vw;
  height: 50vh;
  display: flex;
  justify-content: center;
  align-items: center;
}
#index-tpl ul {
  display: flex;
  justify-content: center;
  align-items: center;
}
#index-tpl ul li {
  display: flex;
  flex-flow: column nowrap;
  justify-content: space-between;
  align-items: center;
  border: 1px solid #ddd;
}
#index-tpl ul li .goods_name,
#index-tpl ul li .goods_price {
  background-color: red;
  width: 100%;
  text-align: center;
  color: #fff;
  font-size: 1.2rem;
}

#index-tpl ul li .goods_img:hover {
  cursor: pointer;
}
</style>
```

还有Shop的代码：

```
<template>
  <div id="shop-tpl">
    <table>
      <thead>
        <tr>
          <th>名称</th>
          <th>价格</th>
          <th>数量</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(goods, index) in $store.state.shopList" :key="goods.id">
          <td>{{ goods.name }}</td>
          <td>￥{{ goods.price * goods.number }}</td>
          <td>
            <button @click.self="addGoodsNumber(index, $event)">+</button>
            <span>{{ goods.number }}</span>
            <button @click.self="subGoodsNumer(index, $event)">-</button>
          </td>
        </tr>
      </tbody>
      <tfoot>
        <tr>
          <td colspan="3">总结 : ￥{{ $store.getters.totalPrice }}</td>
        </tr>
      </tfoot>
    </table>
  </div>
</template>

<script setup>
import { useStore } from "vuex";
import mutationsType from "../store/mutations_type";
const store = useStore();

const addGoodsNumber = (goodsIndex, event) => {
  const number = +1;
  store.commit(mutationsType.changeShopGoodsNumber, { goodsIndex, number });
};

const subGoodsNumer = (goodsIndex, event) => {
  const number = -1;
  // 如果商品购物车中已添加的数量大于1，就不用移除该商品
  if (store.state.shopList[goodsIndex].number > 1) {
    store.commit(mutationsType.changeShopGoodsNumber, { goodsIndex, number });
  }
  // 否则就需要将该商品移除至购物车
  else {
    store.commit(mutationsType.decrShopList, { goodsIndex });
  }
};
</script>

<style scoped>
#shop-tpl {
  width: 100vw;
  height: 50vh;
  display: flex;
  justify-content: center;
  align-items: flex-start;
}
#shop-tpl table {
  border-collapse: collapse;
  width: 880px;
}
#shop-tpl table tr td,
#shop-tpl table tr th {
  border: 1px solid #aaa;
  text-align: center;
  padding: 0.2rem;
}

#shop-tpl table thead tr {
  background: #dddd;
}

#shop-tpl table tbody tr td:last-of-type span {
  margin: 0 0.5rem;
}
#shop-tpl table tbody tr td:last-of-type button {
  width: 2rem;
}

#shop-tpl table tr:nth-child(even) {
  background: #dddd;
}
</style>
```



# 版本区别

## 创建仓库

在Vue-X4.0之前，创建仓库并不是通过下面的方式创建：

```
import { createStore } from "vuex"
```

而是通过VueX.store()进行创建：

```
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})
```



## 获取仓库

在Options API中，如果想要在&lt;script&gt;中获取store仓库对象，则需要使用this进行调用：

```
this.$store.state.data;
this.$store.commit('mutationsType', playload);
this.$store.dispatch('actionsType', playload);
```

