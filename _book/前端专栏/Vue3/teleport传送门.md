# 需求场景

在某些情况下，我们需要让一个组件的作用发生到另一个组件上，此时就需要teleport传送门进行实现。

举个例子，我们有3个组件：

- 登录框组件（子组件）
- 主内容组件（父组件）
- 遮罩层组件（全局组件）

我们希望在主内容组件上点击登录按钮时，会触发登录框组件以及遮罩层组件的显示，同时遮罩层组件将作用在主内容组件上。

整体逻辑如下图所示：

![image-20210919183508094](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210919183508094.png)

这个过程就是将遮罩层组件作用在主内容组件身上。





# 代码实现

通过teleport来实现这一个需求：

```
<link rel="stylesheet" href="./demo.css">

<body>
    <div id="app">
        <content-cpn></content-cpn>
    </div>

    <template id="content-tpl">
        <div id="content">
            <!-- 主体内容区域 -->
            <header><button @click="status = !status">登录</button></header>
            <main>
                <!-- 监视取消登录 -->
                <login-cpn v-show="status" @close="status = !status"></login-cpn>
            </main>
            <!-- 遮罩层只有点击登录才生效，点击取消后失效 -->
            <mask-cpn v-show="status"></mask-cpn>
        </div>
    </template>

    <template id="login-tpl">
        <div id="login">
            <!-- 登录框 -->
            <div>
                <h3>欢迎登录</h3>
            </div>
            <div><input type="text"></div>
            <div><input type="text"></div>
            <div><button>提交</button><button @click="close">取消</button></div>
        </div>
    </template>

    <template id="mask-tpl">
        <div id="mask">
            <!-- to：生效在主内容组件上 -->
            <teleport to="#content">
                <!-- 遮罩层 -->
                <aside></aside>
            </teleport>
        </div>
    </template>

    <script src='https://cdn.bootcdn.net/ajax/libs/vue/3.1.5/vue.global.js'></script>
    <script>
        "use strict";
        const maskCpn = {
            template: "#mask-tpl"
        }
        const loginCpn = {
            template: "#login-tpl",
            methods: {
                // 监听退出事件按钮
                close(event) {
                    this.$emit("close");
                }
            }
        }
        const contentCpn = {
            template: "#content-tpl",
            data() {
                return {
                    status: false
                }
            },
            components: {
                loginCpn
            }
        }
        const app = Vue.createApp({
            data() {
                return {
                    maskStatus: false
                }
            },
            components: {
                contentCpn
            }
        })
        app.component("maskCpn", maskCpn);
        app.mount("#app");
    </script>

</body>
```

CSS代码如下：

```
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body #content {
  width: 400px;
  height: 400px;
  border: 1px solid #ddd;
  display: flex;
  flex-flow: column nowrap;
  position: relative;
}
body #content > header {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  padding-right: 0.2rem;
  height: 10%;
  background-color: #aaa;
}
body #content > main {
  background: #bbb;
  height: 90%;
}
body #content > main #login {
  background: #fff;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  padding: 0.5rem;
  z-index: 1;
}
body #content > main #login div {
  margin: 0.2rem;
}
body #content > #mask {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
}
```

