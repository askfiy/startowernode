# Vue-cli

## 安装cli

[Vue-cli](https://cli.vuejs.org/)是Vue官方发布的构建Vue项目的脚手架，通过Vue-cli我们可以快速的生成Vue项目以及构建开发环境。

一言以蔽之，使用Vue-cli能让你的Vue项目搭建更加轻松，测试更加方便。

首先先全局安装最新版的Vue-cli：

```
$ npm install -g @vue/cli
```

安装完成后使用以下命令查看版本，注意Vue-cli版本号必须大于4.5才能运行Vue3项目：

```
$ vue -V
```

如果你之前安装过Vue-cli，可通过以下命令进行升级：

```
$ npm update -g @vue/cli
```

如果你习惯使用yarn而不是npm，则输入以下命令即可：

```
$ yarn global add @vue/cli               -- 安装
$ yarn global upgrade --latest @vue/cli  -- 升级
```





## 创建项目

在Vue3版本后，对于项目的构建我们推荐使用[Vite](https://cn.vitejs.dev/)来进行，在Vue2时期，你可以通过直接通过Vue-cli来构建你的项目，Vite相比于直接通过Vue-cli创建项目来说更加的简单也更容易上手。

下面让我们构建自己的第一个Vue工程化项目，注意项目的命名风格必须是kebab-case风格：

```
$ npm init @vitejs/app <project-name>
```

在运行完上面的命令后，它会让你输入构建项目的框架framework，选择vue即可，另外会让你选择语言类型，如果不用TypeScript，则选用vue即可：

```
✔ Package name: … demo-project
✔ Select a framework: › vue
✔ Select a variant: › vue
```

下一步我们需要进入到刚刚创建的项目中：

```
$ cd <project-name>
```

运行以下命令安装package.json中所有的依赖：

```
$ npm install
```

接下来你就可以启动这个Vue项目了，默认项目的端口是3000：

```
$ npm run dev
```

如果你习惯使用yarn而不是npm，则输入以下命令即可：

```
$ yarn create @vitejs/app <project-name>
$ cd <project-name>
$ yarn
$ yarn dev
```

注意！如果你的Node版本是V7及以上，则需要在启动项目之前运行下面这一条命令来重新安装esbuild，否则项目将不能启动：

```
$ node node_modules/esbuild/install.js
```

以下是启动后的界面，这个count是可以点击的，算是一个小demo吧：

![image-20210925231823589](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210925231823589.png)



## 目录解析

以下是整个Vue初始化项目的目录结构：

```
├── node_modules
│   ├── ...
├── public
│   └── favicon.ico
├── src
│   ├── App.vue
│   ├── assets
│   │   └── logo.png
│   ├── components
│   │   └── HelloWorld.vue
│   └── main.js
├── index.html
├── package-lock.json
├── package.json
├── README.md
├── .gitignore
└── vite.config.js
```

当前根目录下子目录说明：

- node_modules：项目所需要的模块安装目录
- public：存放公共的静态资源文件
- src：源代码文件
- *dist：当运行npm run build命令后会生成该目录，存放打包后的资源文件

当前根目录下的文件说明：

- index.html：由于Vue更多的是单页面开发，所以index.html也就是这个项目的单页面，你可以将它理解为根组件模板，可在此中引入css-reset等文件
- package-lock.json：项目中固定版本的依赖包清单文件
- package.json：项目中非固定版本的依赖包清单文件，npm脚本存储的文件
- README.md：项目的详细信息描述文件
- .gitgnore：git管理时忽略的目录或文件清单
- vite.config.js：项目配置文件，在Vue2时期该文件名为vue.config.js

src目录中的子目录与文件说明：

- assets：项目中组件的静态资源目录，可创建imgs、video等目录存放静态资源
- components：项目中组件的存放目录，可在内创建更详细的子目录用于区分不同功能的组件
- App.vue：根组件，即index.html中#div所挂载的组件
- main.js：项目入口文件，用于将App.vue和index.html进行挂载联系
- *router：存放路由文件的目录，通常情况下需要使用Vue-router插件后方可具备
- *store：存放数据文件的目录，通常情况下需要使用VueX插件后方可具备



## npm脚本

使用Vite构建Vue项目后，它默认的会添加3个npm脚本，我们可以在package.json中查看到：

```
{
  "scripts": {
    "dev": "vite", // 启动开发服务器
    "build": "vite build", // 为生产环境构建产物
    "serve": "vite preview" // 本地预览生产构建产物
  }
}
```

运行npm脚本，你需要通过以下方式执行上面的3个命令：

```
$ npm run 脚本名称
```

我们还可以指定额外的命令行选项，如 --port或 --https。运行npx vite --help获得完整的命令行选项列表。

## 启动流程

当我们执行了npm run dev后，它内部会做哪些事情呢？我们可以从index.html文件中入手，首先它的结构其实很简单，就是一个普普通通的HTML文件，只不过在&lt;body&gt;标签内部加入了根组件模板元素div#app以及导入了main.js：

```
<body>
    <div id="app"></div>
    <script type="module" src="/src/main.js"></script>
</body>
```

接下来我们可以看一下main.js，可以很清晰的看到它就是通过App.vue创建了一个Vue应用：

```
import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')
```

而关于App.vue，仔细看&lt;template&gt;以及&lt;script&gt;标签的内容，它的&lt;script&gt;标签内部其实采用了Vue3的语法，下面我们会对其进行详细介绍，至于&lt;script&gt;中的内容你目前可以将它看成下面这一段代码：

```
<script>
import HelloWorld from "./components/HelloWorld.vue";
export default {
  components: {
    HelloWorld,
  },
};
</script>

<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <HelloWorld msg="Hello Vue 3 + Vite" />
</template>
```

这一段代码的意思是说，导出了一个默认的组件，并且该组件中注册了一个HelloWorld的子组件，而且还在&lt;template&gt;模板中进行了使用，至于为什么App.vue中这个导出的组件没有定义template属性呢，实际上它内部已经定义好了，就是指向下面这个template。

然后我们需要看一下，HelloWorld组件，也是关注一下&lt;script&gt;标签就好，你可能现在看不懂这个文件（你电脑上的），没关系，我们可以参照下面这个代码：

```
<script>
export default {
  name : "HelloWorld",
  props : {
    msg : String
  },
  data(){
    return {
      count : 0
    }
  }
}
</script>
```

很简单，它通过props接收了一个msg，就是App.vue中传递的那个。

然后内部的data()中返回了一个count，这个count计数为0。其他就没什么了，至此我们的启动流程算是跑完了，以下是关于初始化项目中组件的划分：

![image-20210925231747764](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210925231747764.png)





# 一些细节

## @符号

在Vue2的组件中当你需要通过import语句导入其他组件或资源时，可通过@符号代指src目录，如下示例：

```
import HelloWorld from "@/components/HelloWorld.vue";
```

但是在Vue3中需要我们自己来进行配置，打开vite.config.json文件，粘贴一下内容即可：

```
import { resolve } from 'path'

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      "@": resolve(__dirname, "src")
    }
  }
})

```

如果你熟悉webpack，那么上面的操作对你来说肯定不会陌生，如果你不熟悉webpack也没关系，它其实就是为src目录取一个别名而已。





## vue文件

vue文件你可以将它认为是一个独立的组件，通常情况下该文件的命名风格均采用大驼峰式命名法。

我们以App.vue来进行举例，每个vue文件都有3部分组成，且在&lt;script&gt;中书写组件时不需要指定template属性，它内部会自动的进行挂载：

```
<script>
  // 组件内容
</script>

<template>
  // 组件模板
</template>

<style>
  // 组件样式
</style>
```

下面我们可以自己书写一个组件，并在App.vue中进行注册：

```
# src/components/Test.vue

<script>
export default {
  name: "test",
  data() {
    return {
      message: "test component",
    };
  },
};
</script>

<template>
  <div>{{ message }}</div>
</template>

<style scoped>
div {
  display: inline-block;
  background: #fff;
  padding: 1rem;
}
</style>
```

然后再到App.vue中导入并注册：

```
# src/components/App.vue

<script>
import Test from "@/components/Test.vue"
export default {
  components: {
    Test,
  },
};
</script>

<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <Test />
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```

注意在导入一个vue文件时必须要添加上后缀名：

```
import Test from "./components/Test.vue";
```

如果你不想添加后缀名，也可以通过vite.config.js来进行设置，总体配置和webpack差不多：

```
import { resolve } from 'path'

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      "@": resolve(__dirname, "src")
    },
    extensions: [".vue",]           // 忽略后缀名，自动匹配
  }
})
```

当配置完成后，在导入vue文件时就不需要加上后缀了：

```
import Test from "@/components/Test";
```





## style - scoped

通过观察可以发现，除了根组件App.vue外的其他组件中，&lt;style&gt;标签会多一个属性scoped。

添加了该属性的&lt;style&gt;的样式只会在当前组件中生效。

这意味着App.vue的&lt;style&gt;全局生效的，你可以在此引入一些公共的样式资源，如reset-css：

```
<style>
@import url("./assets/css/css-reset.css");
</style>
```



## script - setup

Vue3的工程化项目中，几乎所有的&lt;script&gt;标签都会有一个setup属性，这正是Composition API所带来的变化。

它的意思在于，在这个&lt;script&gt;标签中你可以直接书写之前在setup()中的代码，且不用再进行return暴露接口的操作。

除此之外，它内部会自动进行export将该组件进行导出，如下所示：

我们以HelloWorld组件来进行举例，下面是它原本的代码，实际上就是Composition API的setup()简写：

```
<script setup>
// 不用return {msg} ，内部会自动进行
// 也不用return export， 内部会自动进行
import { ref } from 'vue'

defineProps({
  msg: String
})

const count = ref(0)
</script>
```

将它修改成完整的Composition API，效果一致，代码量增加：

```
<script>
import { ref } from "vue";

export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },
  setup(props, context) {
    return {
      count: ref(0),
    };
  },
};
</script>
```

将它修改成Options API，效果一致，代码量增加：

```
<script>
export default {
  name : "HelloWorld",
  props : {
    msg : String
  },
  data(){
    return {
      count : 0
    }
  }
}
</script>
```



## setup中的API

&lt;script setup&gt;实际上是实验的功能，不排除后期会全部进行应用。

在&lt;script setup&gt;中编写代码时，需要注意以下API的变更。

- 如果想注册组件，直接import即可，它会自动进行注册：

    ```
    import Cpn from "@/components/Cpn.vue"
    ```

- 使用defineProps替代props：

    ```
    <script setup>
    const props = defineProps({
      foo: String
    })
    </script>
    ```

- 使用defineEmits代替context.emit()：

    ```
    <script setup>
    // 先注册
    const emit = defineEmits(['change', 'delete'])
    // 再使用
    emit("change", "Hello")
    emit("delete", 1)
    </script>
    ```



其他更多API请参阅[官方文档](https://v3.cn.vuejs.org/api/sfc-script-setup.html)



# 项目上手

## 项目一览

我们以一个todolist来作为Vue-cli的熟悉项目，最终效果如下：

![toList的demo](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/toList的demo.gif)

上面这个例子由1个大组件和3个小组件构成，分别是：

- main-cpn：整体组件
- header-cpn：头部组件
- left-cpn：左侧日程选择组件
- right-cpn：右侧日程展示组件

我们需要在components目录中新建一个toListCpn的子目录，并且将这些组件放进去，再次提醒，vue文件的命名要以大驼峰式命名法进行：

```
src/components/
└── toListCpn
    ├── Header.vue
    ├── Left.vue
    ├── Main.vue
    └── Right.vue
```



## 选项式API演示

首先需要从App.vue中引入Main.vue，并且定义一些全局样式：

```
<script>
import Main from "./components/toListCpn/Main.vue";
export default {
  components: {
    Main,
  },
};
</script>

<template>
  <Main />
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

#app {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}
</style>
```

其次是Main.vue的代码，我们定义了事件列表eventList与一系列修改事件列表的方法，因为该组件所产生的数据对象eventList会分发给其他组件，所以当其他组件需要修改事件列表时也必须依赖该组件所提供的方法：

```
<template>
  <main>
    <!-- header -->
    <Header :eventList="eventList" />
    <!-- left -->
    <Left :registerEvent="registerEvent" />
    <!-- right -->
    <Right
      :eventList="eventList"
      :unRegisterEvent="unRegisterEvent"
      :changeEventStatus="changeEventStatus"
    />
  </main>
</template>

<script>
import Header from "./Header.vue";
import Left from "./Left.vue";
import Right from "./Right.vue";
export default {
  name: "Main",
  data() {
    return {
      eventList: [],
    };
  },
  methods: {
    registerEvent(time, event) {
      this.eventList.push({ time, event, finish: false });
    },
    unRegisterEvent(index) {
      this.eventList.splice(index, 1);
    },
    changeEventStatus(index) {
      this.eventList[index].finish = !this.eventList[index].finish;
    },
  },
  components: {
    Header,
    Left,
    Right,
  },
};
</script>

<style scoped>
main {
  border: 1px solid #ddd;
  background: #4ca64c;
}
main:before {
  clear: both;
  content: "";
  display: block;
}
</style>
```

接下来是Header.vue的代码，Header组件只需要获取eventList即可，因为它仅展示eventList中的已注册事件和已完成事件：

```
<template>
  <div id="header">
    <div>
      <h3>{{ currentTime }}</h3>
    </div>
    <div>
      <span>已完成({{ getFinishEvent }})/今日事件({{ getTotalEvent }})</span>
    </div>
  </div>
</template>

<script>
export default {
  name: "Header",
  props: {
    eventList: Array,
  },
  computed: {
    getTotalEvent() {
      return this.eventList.length;
    },
    getFinishEvent() {
      return this.eventList.reduce((pre, cur, idx, ary) => {
        return (pre += cur.finish ? 1 : 0);
      }, 0);
    },
    currentTime() {
      const date = new Date();
      let format = "YYYY年MM月DD日";

      const config = {
        YYYY: date.getFullYear(), // 获取年份
        MM: date.getMonth() + 1, // 获取月份，月份+1是因为Js中的月份是0-11
        DD: date.getDate(), // 获取天数
        HH: date.getHours(), // 获取小时
        mm: date.getMinutes(), // 获取分
        ss: date.getSeconds(), // 获取秒
      };
      for (const key in config) {
        format = format.replace(key, config[key]);
      }
      return format;
    },
  },
};
</script>

<style scoped>
#header {
  width: 400px;
  height: 80px;
  padding: 0.5rem;
  margin-bottom: 0.5rem solid #ddd;
  border-bottom: 1px solid #ddd;
}
#header div:first-of-type {
  font-size: 1.6rem;
}
#header div:last-of-type {
  font-size: 0.8rem;
}
</style>
```

然后是Left.vue的代码，Left组件中只需要接收注册事件的方法即可：

```
<template>
  <div id="left">
    <input
      type="text"
      placeholder="输入日程信息"
      v-model.lazy="eventDescribe"
    />
    <ul>
      <li
        v-for="time in timeList"
        :key="time"
        @click="timeClickfn(time)"
      >
        {{ time }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: "Left",
  props: {
    registerEvent: Function,
  },
  data() {
    return {
      eventDescribe: "",
      eventTime: null,
    };
  },
  computed: {
    timeList() {
      let time = [];
      for (let i = 0; i < 24; i++) {
        if (String(i).length === 1) {
          time.push(String.prototype.concat("0", i, ":", "00"));
        } else {
          time.push(String.prototype.concat(i, ":", "00"));
        }
      }
      return time;
    },
  },
  methods: {
    timeClickfn(time) {
      if (!this.eventDescribe) {
        alert("您还未输入日程信息");
        return;
      }
      this.eventTime = time;
      this.registerEvent(this.eventTime, this.eventDescribe);
      this.eventDescribe = "";
    },
  },
};
</script>

<style scoped>
#left {
  width: 200px;
  height: 200px;
  padding: 0.5rem;
  float: left;
  border-right: 1px solid #ddd;
  background: #4cd66a;
}

#left input {
  width: 100%;
  height: 18%;
  margin-bottom: 2%;
  outline: none;
}
#left ul {
  list-style: none;
  text-align: center;
  height: 80%;
  overflow-y: scroll;
  background: #fff3fe;
}
#left ul li:hover {
  background-color: #ddd;
  font-size: 1.4rem;
  cursor: pointer;
}
</style>
```

最后是Right.vue的代码，Right组件中所做的工作比较多，需要注销事件、修改事件状态、删除事件，但是代码量很少，因为这些方法都在Main组件中定义好了，直接调用即可，如下所示：

```
<template>
  <div id="right">
    <ul>
      <li v-for="(row, index) in eventList" :key="row">
        <input type="checkbox" @change="changeEventStatus(index)" />
        <time>{{ row.time }}</time>
        <span>{{ row.event }}</span>
        <button @click="unRegisterEvent(index)">删除</button>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: "Right",
  props: {
    eventList: Array,
    unRegisterEvent: Function,
    changeEventStatus: Function,
  },
};
</script>

<style scoped>
#right {
  width: 200px;
  height: 200px;
  padding: 0.5rem;
  float: left;
  overflow-y: scroll;
  background: #fff3fe;
}
#right > ul {
  height: 100%;
  margin-bottom: 2%;
}
#right > ul li:after {
  clear: both;
  content: "";
  display: block;
}
#right > ul li:hover button {
  display: inline-block;
}
#right > ul li {
  border: 1px solid #aaa;
  padding: 0 0.5rem;
  height: 1.5rem;
}
#right > ul li > input[type="checkbox"] {
  margin-right: 5%;
}
#right > ul li > time {
  width: 3rem;
  display: inline-block;
  margin-right: 5%;
  text-align: left;
}
#right > ul li > span {
  width: 70%;
}
#right > ul li > button {
  float: right;
}
#right > ul li > button {
  display: none;
}
</style>
```



## 组合式API演示

我们基于上面选项式API来将所有的JavaScript代码改装成Vue3的组合式API。

首先是App.vue，它只需要直接导入Main组件即可，内部会自动将该组件注册成为App的子组件：

```
<script setup>
import Main from "./components/toListCpn/Main.vue";
</script>
```

其次是Main.vue，在它里面我们需要使用reactive将eventList修改为响应式的，且还需要使用provide将数据分发给子组件们：

```
<script setup>
import { reactive } from "vue";
import { provide } from "vue";

import Header from "./Header.vue";
import Left from "./Left.vue";
import Right from "./Right.vue";

const eventList = reactive([]);

const registerEvent = (time, event) => {
  eventList.push({ time, event, finish: false });
};
const unRegisterEvent = (index) => {
  eventList.splice(index, 1);
};
const changeEventStatus = (index) => {
  eventList[index].finish = !eventList[index].finish;
};

provide("eventList", eventList);
provide("registerEvent", registerEvent);
provide("unRegisterEvent", unRegisterEvent);
provide("changeEventStatus", changeEventStatus);

</script>
```

下面是Header.vue的代码，Header组件只需要通过inject接收Main组件中provide过来的eventList即可：

```
<template>
  <div id="header">
    <div>
      <h3>{{ currentTime }}</h3>
    </div>
    <div>
      <span>已完成({{ getFinishEvent }})/今日事件({{ getTotalEvent }})</span>
    </div>
  </div>
</template>

<script setup>
import { inject } from "vue";
import { computed } from "vue";

let eventList = inject("eventList");

let getTotalEvent = computed(() => {
  return eventList.length;
});

let getFinishEvent = computed(() => {
  return eventList.reduce((pre, cur, idx, ary) => {
    return (pre += cur.finish ? 1 : 0);
  }, 0);
});

let currentTime = computed(() => {
  const date = new Date();
  let format = "YYYY年MM月DD日";

  const config = {
    YYYY: date.getFullYear(), // 获取年份
    MM: date.getMonth() + 1, // 获取月份，月份+1是因为Js中的月份是0-11
    DD: date.getDate(), // 获取天数
    HH: date.getHours(), // 获取小时
    mm: date.getMinutes(), // 获取分
    ss: date.getSeconds(), // 获取秒
  };
  for (const key in config) {
    format = format.replace(key, config[key]);
  }
  return format;
});
</script>
```

然后是Left.vue的代码，这里面需要借助inject、computed以及ref，注意在timeClickfn()的回调函数中eventDescribe等经过ref包裹的属性设置值时一定要通过proxy.value属性进行：

```
<script setup>
import { inject } from "vue";
import { computed } from "vue";
import { ref } from "vue";

const registerEvent = inject("registerEvent");
const eventDescribe = ref("");
const eventTime = ref(null);

const timeList = computed(() => {
  let time = [];
  for (let i = 0; i < 24; i++) {
    if (String(i).length === 1) {
      time.push(String.prototype.concat("0", i, ":", "00"));
    } else {
      time.push(String.prototype.concat(i, ":", "00"));
    }
  }
  return time;
});

const timeClickfn = (time) => {
  if (!eventDescribe.value) {
    alert("您还未输入日程信息");
    return;
  }
  eventTime.value = time;
  registerEvent(eventTime.value, eventDescribe.value);
  eventDescribe.value = "";
};
</script>
```

最后是Right.vue，它的改造最简单，只需要将props接收的属性通过inject接收即可：

```
<script setup>
import { inject } from "vue";
const eventList = inject("eventList");
const unRegisterEvent = inject("unRegisterEvent");
const changeEventStatus = inject("changeEventStatus");
</script>
```

