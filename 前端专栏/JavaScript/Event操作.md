# 基础知识

## 事件作用

某些时候，我们希望用户在页面上做出一些行为后JavaScript能够自动的运行一段特定的代码。

如用户鼠标单击后放大图片、用户鼠标双击后改变文本颜色等需求均可使用事件来完成。

事件的使用总体分为2大部分：

- 如何给元素绑定事件
- 如何编写事件的处理逻辑

在JavaScript中对于事件的处理采用异步驱动形式进行，所以它的效率是非常高的，不必担心出现任何性能问题。





## 事件类型

事件拥有多种类型，比如文档事件、鼠标事件、键盘事件等。

不同类型的事件是不同的对象，他们会提供不同的属性来供开发人员使用。



## 事件对象

事件本身是一个对象，当一个事件发生时会自动的创建出该类型的事件对象，大多数情况下它都会被存储在window对象中。

我们可以通过window.event属性来调用出当前的事件对象。

所有的事件对象原型都继承了Event，Event为所有类型的事件对象提供了一些通用的属性及方法，如下所示：

| 属性                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [Event.type](https://developer.mozilla.org/zh-CN/docs/Web/API/Event/type) | 事件的类型，不区分大小写                                     |
| [Event.target](https://developer.mozilla.org/zh-CN/docs/Web/API/Event/target) | 对事件原始目标的引用，这里的原始目标指最初派发（dispatch）事件时指定的目标 |
| [Event.currentTarget](https://developer.mozilla.org/zh-CN/docs/Web/API/Event/currentTarget) | 当前正在执行事件的对象，如果有事件冒泡发生，它将是不断变化的 |
| [Event.cancelable](https://developer.mozilla.org/zh-CN/docs/Web/API/Event/cancelable) | 一个布尔值，表示事件是否可以取消                             |
| [Event.defaultPrevented](https://developer.mozilla.org/zh-CN/docs/Web/API/Event/defaultPrevented) | 一个布尔值，表示 [event.preventDefault()](https://developer.mozilla.org/zh-CN/docs/Web/API/Event/preventDefault) 方法是否取消了事件的默认行为 |
| [Event.bubbles](https://developer.mozilla.org/zh-CN/docs/Web/API/Event/bubbles) | 一个布尔值，用来表示该事件是否会在 DOM 中冒泡                |
| [Event.eventPhase](https://developer.mozilla.org/zh-CN/docs/Web/API/Event/eventPhase) | 表示事件流正被处理到了哪个阶段                               |



## 事件目标

事件目标分为2种，一种是引发事件的目标，一种是正在执行事件的目标。

- event.target：它指向引发此处事件的源目标，是不会改变的
- event.currentTarget：它指向事件正在执行的目标，是允许改变的，只有在冒泡行为产生后才能体现该属性的作用

为什么会有2种目标呢？

这个就和事件执行周期有关了，因为元素是可以嵌套的，所以大多数事件在执行时都会产生多个目标对象。



## 事件周期

当一个事件被绑定且执行时，一定会有3个大阶段，为了方便后续探讨，我将它分为4个小阶段，分别是：

- 捕获阶段：事件从Window对象开始派发给源目标
- 目标阶段：事件到达源目标
- 传播阶段：事件开始在源目标上主动执行处理逻辑的回调函数，此时事件的执行目标等同于源目标 （这是一个自我划分的阶段）
- 冒泡阶段：事件从源目标上进行冒泡，被动执行父级相同事件的回调函数，此时事件的执行目标发生改变，源目标不变

如下图所示：

![image-20210805170419286](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210805170419286.png)





# 事件绑定

## 回调函数

一般来说我们在为一个元素添加一个事件监听后会为其绑定一个回调函数。

该回调函数会在事件发生后自动进行调用。



## HTML on

直接在HTML代码中绑定事件回调函数，在绑定时必须添加括号。

它会在事件发生后自动进行执行，可传入参数this和event：

- this：事件源目标对象，即标签本身
- event：当前的事件对象

需要注意this的执行是会根据回调函数形态的不同而发生变化的，具体可参照下面this指向一章节。

示例如下：

```
<style>

</style>

<body>
    <button onclick="callbackfn(this, event)">click me</button>
</body>

<script>

    "use strict";

    function callbackfn(target, event) {
        console.log("run");
    }

</script>
```



## DOM on

也可以将事件回调函数绑定到DOM节点对象的属性中，有2点注意事项：

- 不能使用setAttribute()方法进行属性设置，因为它针对的是标签的普通、特征属性，而非Node节点本身的属性
- 属性名必须区分大小写

在绑定时，我们可指定形参event，它将在回调函数运行时自动传入当前的事件对象。

示例如下：

```
<style>

</style>

<body>
    <button>click me</button>
</body>

<script>

    "use strict";

    let btnNode = document.querySelector("button");

    btnNode.onclick = function (event) {
        console.log("run");
    };

</script>
```



## 监听绑定

### addEventListener()

不论是HTML绑定还是DOM绑定都具有一定的缺陷，因此推荐使用ES6新增的addEventListener()方法进行事件绑定。

它的优点如下：

- 支持transtionend和DOMContentLoaded等事件类型的绑定
- 同一节点对象支持多次绑定相同事件，它会按照绑定顺序进行依次执行
- 支持对未来元素添加绑定事件

参数说明：

| 参数     | 描述                                                    |
| -------- | ------------------------------------------------------- |
| type     | 需要绑定的监听事件类型                                  |
| listener | 需要绑定的回调处理程序，如果是回调函数，可传入形参event |
| options  | 定制选项                                                |

着重说明options的可定制选项：

- once：布尔值，如果为true代表只执行一次事件处理程序，默认是false
- capture：布尔值，表示listener会在该类型的事件捕获阶段传播到该元素时触发，默认是false
- passive：布尔值，如果为true代表用于不要在回调函数中使用event.preventDefault()，否则将抛出异常，默认是false

示例如下，添加一个只执行一次的事件，指定options的once为true即可：

```
<style>

</style>

<body>
    <button>click me</button>
</body>

<script>

    "use strict";

    let btnNode = document.querySelector("button");

    btnNode.addEventListener("click", function (event) {
        console.log("run");
        event.target.innerText = "don't click me";
    }, { once: true });

</script>
```



### removeEventListener()

使用removeEventListener()方法删除某个节点所绑定的事件及回调函数。

示例如下，左键单击&lt;button&gt;按钮会令&lt;span&gt;内容+1，右键单击&lt;button&gt;按钮会令左键功能失效：

```
<style>

</style>

<body>
    <div><span>0</span></div>
    <button title="click + 1, double click cancellation click + 1 function">click me</button>
</body>

<script>

    "use strict";

    let btnNode = document.querySelector("button");
    let spanNode = document.querySelector("div span");

    function click(event) {
        spanNode.innerText = Number(spanNode.innerHTML) + 1;
    }

    function contextmenu(event) {
        btnNode.removeEventListener("click", click);
        btnNode.disabled = true;
        // 禁止右键的弹出菜单
        event.preventDefault();
    }

    btnNode.addEventListener("click", click);
    btnNode.addEventListener("contextmenu", contextmenu);

</script>
```



### handleEvent()

如果事件处理的回调程序不是一个函数，而是一个对象的话，那么对象中的handleEvent()方法会在事件触发后自动进行执行：

```
<style>

</style>

<body>
    <button>click me</button>
</body>

<script>

    "use strict";

    let btnNode = document.querySelector("button");

    class BTNEventHandle {

        constructor(node) {
            node.addEventListener("mousedown", this);
            node.addEventListener("mouseup", this);
        }

        handleEvent(event) {
            // 寻找同名方法 this.类型方法
            // 并传递进事件对象和源目标对象
            this[event.type](event, event.target);
        }

        mousedown(event, target) {
            console.log("按下了鼠标左键");
        }

        mouseup(event, target) {
            console.log("松开了鼠标左键");
        }

    }

    new BTNEventHandle(btnNode);

</script>
```



## this指向

如果回调函数是普通函数，那么this即指向事件的源目标对象。

如果回调函数是箭头函数，那么this即指向window对象。

示例如下：

```
<style>

</style>

<body>
    <button>ordinary</button>
    <button>arrow</button>
</body>

<script>

    "use strict";

    let btnNodeOrdinary = document.querySelector("button:first-of-type");
    let btnNodeArrow = document.querySelector("button:last-of-type");

    btnNodeOrdinary.addEventListener("click", function (event) {
        console.log(this);           // <button>ordinary</button>
    });

    btnNodeArrow.addEventListener("click", event => {
        console.log(this);           // Window
    });


</script>
```



## event.target

如果想在箭头函数中拿到事件源目标对象，可使用event.target属性。

```
<style>

</style>

<body>
    <button>ordinary</button>
    <button>arrow</button>
</body>

<script>

    "use strict";

    let btnNodeOrdinary = document.querySelector("button:first-of-type");
    let btnNodeArrow = document.querySelector("button:last-of-type");

    btnNodeOrdinary.addEventListener("click", function (event) {
        console.log(this);           // <button>ordinary</button>
        console.log(event.target);   // <button>ordinary</button>
    });

    btnNodeArrow.addEventListener("click", event => {
        console.log(this);           // Window
        console.log(event.target);   // <button>ordinary</button>
    });


</script>
```



## 事件传播

HTML on和DOM on的方式都不支持为同一监听事件绑定多个回调函数：

```
<style>

</style>

<body>
    <button>ordinary</button>
    <!-- 总会执行第1个 -->
    <button onclick="arrowFunc01(this, event)" onclick="arrowFunc02(this, event)">arrow</button>
</body>

<script>

    "use strict";

    // 总会执行第2个
    let btnNodeOrdinary = document.querySelector("button:first-of-type");
    btnNodeOrdinary.onclick = ordinaryFunc01;
    btnNodeOrdinary.onclick = ordinaryFunc02;

    function arrowFunc01(target, event) {
        console.log("run arrow function 01");
    }
    function arrowFunc02(target, event) {
        console.log("run arrow function 02");
    }

    function ordinaryFunc01(event) {
        console.log("run ordinary function 01");
    }
    function ordinaryFunc02(event) {
        console.log("run ordinary function 02");
    }

</script>
```

但是addEventListener()支持为同一监听事件绑定多个回调函数，当事件发生时他会依次进行处理。

整个执行的过程我将其称之为事件传播过程：

```
<style>

</style>

<body>
    <button>click me</button>
</body>

<script>

    "use strict";

    let btnNode = document.querySelector("button");

    btnNode.addEventListener("click", event => {
        console.log("1");
    });

    btnNode.addEventListener("click", event => {
        console.log("2");
    });

    btnNode.addEventListener("click", event => {
        console.log("3");
    });


</script>
```

如下图所示：

![image-20210805163028957](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210805163028957.png)





## on是什么意思

on其实就是开始监听的意思。

通过HTML属性或者DOM属性绑定回调函数时，需要在事件名前加入on，比如click就变为onclick，copy变为oncopy。

而使用事件监听的方式则不需要加上on，因为addEventListener()本身就有监听的功能。



# 默认行为

## 默认事件

某些元素会有一些默认监听的行为。

比如：

- &lt;a&gt;标签点击后会自动进行页面跳转
- &lt;input type=“submit”&gt;点击后会自动搜集表单项并提交

在某些场景下我们需要对这些具有默认行为的元素绑定事件，并且还要让其阻止默认行为的发生。



## 阻止默认

阻止默认行为一共有2个方案：

- 适用于任何场景：在回调函数中加上event.preventDefault()即可
- 仅适用于DOM on：在回调函数中return false即可

1）适用于任何场景：在回调函数中加上event.preventDefault()即可：

```
<style>

</style>

<body>

    <a href="#" onclick="linkClick(this, event)"> click me </a>

</body>

<script>

    "use strict";

    function linkClick(target, event) {
        event.preventDefault();
    };

</script>
```



2）仅适用于DOM on：在回调函数中return false即可：

```
<style>

</style>

<body>

    <a href="#"> click me </a>

</body>

<script>

    "use strict";

    let linkNode = document.querySelector("a");

    linkNode.onclick = function(event){
        return false
    };

</script>
```



## 左键双击选中文本

左键双击选中文本是一个常见的默认行为，我们只需要在元素身上添加上属性onselectstart="return false"即可阻止选中：

```
<style>

</style>

<body>

    <div onselectstart="return false">content</div>

</body>

<script>

    "use strict";

    let divNode = document.querySelector("div");

    divNode.addEventListener("dblclick", event=>{
        console.log("run");
    });

</script>
```





## 右键单击打开菜单

右键单击打开菜单是一个常见的默认行为，我们只需要在元素身上添加上属性oncontextmenu="return false"即可阻止选中：

```
<style>

</style>

<body oncontextmenu="return false">
    <div>点击鼠标右键试试?</div>
</body>

<script>

    "use strict";

    let divNode = document.querySelector("div");

    document.addEventListener("contextmenu", event => {
        divNode.innerText = "是不是发现不好使了?"
    });

</script>
```



# 冒泡行为

## 冒泡行为

元素是支持嵌套的，当子元素与父元素都绑定了相同的一个事件时，子元素事件触发后会进行事件的冒泡传递，也就是说父元素也会自动的执行这一事件，冒泡会一直到达HTML根元素标签。

- 大部分事件都会冒泡，但有的事件不会，如focus事件
- event.target属性是对事件原始目标的引用，这里的原始目标指最初派发（dispatch）事件时指定的目标
- event.currentTarget属性是当前执行事件的目标对象，如果有事件冒泡发生，它将是不断变化的
- event.target和event.currentTarget的区别在于，谁引起了本次事件的产生，谁正在执行本次事件的处理

如下图所示：

![image-20210805170538011](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210805170538011.png)

示例演示：

```
<style>

</style>

<body>
    <main>
        <div>
            <button>click me</button>
        </div>
    </main>
</body>

<script>

    "use strict";

    // 传播阶段
    document.querySelector("main div button").addEventListener("click", event => {
        console.log("button click event callbackfn run 1");
        console.log(`current target : ${event.currentTarget.tagName}, target : ${event.target.tagName}`);
    });

    document.querySelector("main div button").addEventListener("click", event => {
        console.log("button click event callbackfn run 2");
        console.log(`current target : ${event.currentTarget.tagName}, target : ${event.target.tagName}`);
    });

    document.querySelector("main div button").addEventListener("click", event => {
        console.log("button click event callbackfn run 3");
        console.log(`current target : ${event.currentTarget.tagName}, target : ${event.target.tagName}`);
    });

    // 冒泡阶段
    document.querySelector("main div").addEventListener("click", event => {
        console.log("div click event callbackfn run");
        console.log(`current target : ${event.currentTarget.tagName}, target : ${event.target.tagName}`);
    });

    document.querySelector("main").addEventListener("click", event => {
        console.log("main click event callbackfn run");
        console.log(`current target : ${event.currentTarget.tagName}, target : ${event.target.tagName}`);
    });


</script>
```

执行结果：

```
button click event callbackfn run 1
current target : BUTTON, target : BUTTON

button click event callbackfn run 2
current target : BUTTON, target : BUTTON

button click event callbackfn run 3
current target : BUTTON, target : BUTTON

div click event callbackfn run
current target : DIV, target : BUTTON

main click event callbackfn run
current target : MAIN, target : BUTTON
```



## 阻止冒泡

阻止事件冒泡的方法有2个：

- 在处理程序中添加event.stopPropagation()
- 在处理程序中添加event.stopImmediatePropagation()

event.stopPropagation()用于阻止父元素的同事件监听回调不再执行。

event.stopImmediatePropagation()用于阻止父元素的同事件监听回调不再执行，并且相同元素的相同事件类型的监听事件也不再执行。

如下图所示：

![image-20210805172629810](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210805172629810.png)

![image-20210805172614319](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210805172614319.png)

根据不同的情况，选用不同的冒泡阻止方案。

一般来说使用event.stopPropagation()情况会更多一点：

```
<style>

</style>

<body>
    <main>
        <div>
            <button>click me</button>
        </div>
    </main>
</body>

<script>

    "use strict";

    // 传播阶段
    document.querySelector("main div button").addEventListener("click", event => {
        console.log("button click event callbackfn run 1");
        console.log(`current target : ${event.currentTarget.tagName}, target : ${event.target.tagName}`);
        event.stopPropagation();
    });

    document.querySelector("main div button").addEventListener("click", event => {
        console.log("button click event callbackfn run 2");
        console.log(`current target : ${event.currentTarget.tagName}, target : ${event.target.tagName}`);
    });

    document.querySelector("main div button").addEventListener("click", event => {
        console.log("button click event callbackfn run 3");
        console.log(`current target : ${event.currentTarget.tagName}, target : ${event.target.tagName}`);
    });

    // 冒泡阶段
    document.querySelector("main div").addEventListener("click", event => {
        console.log("div click event callbackfn run");
        console.log(`current target : ${event.currentTarget.tagName}, target : ${event.target.tagName}`);
        event.stopPropagation();
    });

    document.querySelector("main").addEventListener("click", event => {
        console.log("main click event callbackfn run");
        console.log(`current target : ${event.currentTarget.tagName}, target : ${event.target.tagName}`);
        event.stopPropagation();
    });


</script>
```

执行结果：

```
button click event callbackfn run 1
current target : BUTTON, target : BUTTON

button click event callbackfn run 2
current target : BUTTON, target : BUTTON

button click event callbackfn run 3
current target : BUTTON, target : BUTTON
```



# 事件捕获

## captrue

事件执行周期由捕获 > 目标 > 传播 > 冒泡这4个阶段组成。

我们可以设置addEventListener()的options参数为true或{ capture: true } 让其在捕获阶段执行事件处理程序。

怎么样表述这一行为呢？其实同冒泡一样，它也必须建立在元素嵌套的基础上进行，当父元素和子元素都监听了同一个事件后，父元素会先执行第一个监听回调，然后再进行正常的逻辑，如下图所示：

![image-20210805172536893](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210805172536893.png)



事件捕获在实际使用中频率不高。

如下所示：

```
<style>

</style>

<body>
    <main>
        <div>
            <button>click me</button>
        </div>
    </main>
</body>

<script>

    "use strict";

    document.querySelector("main").addEventListener("click", event => {
        console.log("main click event callbackfn run");
    }, { capture: true });

    document.querySelector("main div").addEventListener("click", event => {
        console.log("div click event callbackfn run");
    }, { capture: true });

    document.querySelector("main div button").addEventListener("click", event => {
        console.log("button click event callbackfn run");
    }, { capture: true });

</script>
```

执行结果：

```
main click event callbackfn run 1
current target : MAIN, target : BUTTON

button click event callbackfn run
current target : BUTTON, target : BUTTON

div click event callbackfn run
current target : DIV, target : BUTTON

main click event callbackfn run 2
current target : MAIN, target : BUTTON

main click event callbackfn run 3
current target : MAIN, target : BUTTON
```



# 事件代理

## 未来元素

我们想让所有已有的，包括未来可能会创建的&lt;li&gt;元素都监听click事件，该怎么做呢？

目前能掌握到最简单的方法就是先将已有&lt;li&gt;的元素都添加上click事件，未来创建的&lt;li&gt;元素在创建时也添加上click事件：

```
<style>

</style>

<body>
    <ul>
        <li>65</li>
        <li>66</li>
        <li>67</li>
    </ul>
    <button>add new li</button>
</body>

<script>

    "use strict";

    let ulNode = document.querySelector("ul");
    let btnNode = document.querySelector("button");

    // 先将所有已有的li添加上事件
    Array.prototype.forEach.call(ulNode.children, (element, index, lst) => {
        element.addEventListener("click", liCallbackfn)
    })

    // 添加新的标签，并为其绑定事件
    btnNode.addEventListener("click", event => {
        let lastLiNode = ulNode.lastElementChild;
        let lastText = lastLiNode.innerText;

        let newNode = document.createElement("li");
        newNode.innerText = Number.isNaN(Number.parseInt(lastText)) ? lastText.charCodeAt() + 1 : Number(lastText) + 1;
        newNode.addEventListener("click", liCallbackfn);

        lastLiNode.after(newNode);
    })

    function liCallbackfn(event) {
        event.target.innerText = String.fromCharCode(event.target.innerText);
    }

</script>
```



## 委托代理

上述的实现方案非常笨，我们可以借助事件冒泡的思路来完成一个事件委托的操作。

实现过程如下：

- 不为具体的子元素设置监听事件
- 将监听事件设置在父元素上，让父元素监听这一个事件
- 在子元素上点击时，这一行为必定发生在父元素的区域内，此时将触发父元素监听回调
- 父元素通过event.target判定事件发生源是否是指定的某子元素即可，如果是则开始正常执行回调逻辑，如果不是则不执行回调逻辑

实现代码：

```
<style>

</style>

<body>
    <ul>
        <li>65</li>
        <li>66</li>
        <li>67</li>
    </ul>
    <button>add new li</button>
</body>

<script>

    "use strict";

    let ulNode = document.querySelector("ul");
    let btnNode = document.querySelector("button");

    // 直接给ul绑定事件
    ulNode.addEventListener("click", event => {
        // 如果事件目标是li，执行li的回调
        if (event.target.tagName === "LI") {
            liCallbackfn(event);
        }
    })

    // 添加新的标签
    btnNode.addEventListener("click", event => {
        let lastLiNode = ulNode.lastElementChild;
        let lastText = lastLiNode.innerText;
        let newNode = document.createElement("li");
        newNode.innerText = Number.isNaN(Number.parseInt(lastText)) ? lastText.charCodeAt() + 1 : Number(lastText) + 1;
        lastLiNode.after(newNode);
    })

    function liCallbackfn(event) {
        event.target.innerText = String.fromCharCode(event.target.innerText);
    }

</script>
```

将事件代理进行完善，将它做成一个on方法并添加进Element原型对象中，使所有的DOM节点都能进行调用：

```
<style>

</style>

<body>
    <ul>
        <li>65</li>
        <li>66</li>
        <li>67</li>
    </ul>
    <button>add new li</button>
</body>

<script>

    "use strict";


    Element.prototype.on = function (eventType, eventTarget, callbackfn) {
        this.addEventListener(eventType, event => {
            if (event.target.tagName === eventTarget.toUpperCase()) {
                callbackfn(event);
            }
        })
    }

    let ulNode = document.querySelector("ul");
    let btnNode = document.querySelector("button");


    // 委托的DOM节点  代理的事件  事件目标  事件目标的回调函数
    ulNode.on("click", "li", (event) => {
        event.target.innerText = String.fromCharCode(event.target.innerText);
    })

    // 添加新的标签
    btnNode.addEventListener("click", event => {
        let lastLiNode = ulNode.lastElementChild;
        let lastText = lastLiNode.innerText;
        let newNode = document.createElement("li");
        newNode.innerText = Number.isNaN(Number.parseInt(lastText)) ? lastText.charCodeAt() + 1 : Number(lastText) + 1;
        lastLiNode.after(newNode);
    })

</script>
```



# 文档事件

## 常用事件

文档常用事件如下所示，它们一般绑定给window对象。

| 事件             | 描述                                   |
| ---------------- | -------------------------------------- |
| load             | 文档解析完成，且所有外部资源加载完成时 |
| DOMContentLoaded | 文档解析完成时                         |
| beforeunload     | 文档刷新或关闭时                       |
| unload           | 文档卸载时                             |



## load

load事件会在文档解析完成，且所有外部资源加载完成时触发。

一般我们会将其作为JavaScript的入口函数使用：

```
<style>

</style>

<body>
    <!-- 必须等待图片也加载到位后 -->
    <img src="https://www.google.com/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png" alt="">
</body>

<script>

    "use strict";

    window.onload = function (event) {
        console.log("文档加载完成,外部资源加载完成");
    }

</script>
```



## DOMContentLoaded

DOMContentLoaded事件会在文档解析完成时触发，它不需要等待所有外部资源加载完成。

- 该事件只能使用addEventListener()进行设置。

相较于load事件，它更适合作为JavaScript的入口函数，在jQuery库中就是采用该事件作为入口函数的：

```
<style>

</style>

<body>
    <img src="https://www.google.com/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png" alt="">
</body>

<script>

    "use strict";

    window.addEventListener("DOMContentLoaded", function (event) {
        console.log("文档加载完成");
    })

</script>
```



## beforeunload

beforeunload事件会在文档刷新或关闭时触发。

- 在某些浏览器上，该事件的回调函数返回值将作为提示信息弹出，或者你可以通过event.returnValue进行设置
- 部分浏览器无法使用addEventListener()绑定该事件
- 必须在网页上做出更改后，刷新或关闭时才会弹出提示
- 所设定的值并不是一定会显示的，这与浏览器版本有关

该事件常见于一些小网站上，当然我印象最深的地方是拼多多：

```
<style>

</style>

<body>
    <input type="text">
</body>

<script>

    "use strict";

    window.onbeforeunload = function (event) {
        // 兼容IE8和Firefox 4之前的版本
        event.returnValue = '关闭提示';

        // Chrome, Safari, Firefox 4+, Opera 12+ , IE 9+
        return '关闭提示';
    };

</script>
```



## unload

unload事件会在文档卸载时触发，它位于beforeunload事件之后。

- 此时不能使用alert()、confirm()等交互窗指令
- 发生错误也不会阻止页面关闭或刷新

如下示例：

```
<style>

</style>

<body>
    <input type="text">
</body>

<script>

    "use strict";

    window.onunload = function (event) {
        console.log("刷新或关闭了");
    };

</script>
```





# 表单事件

## 常用事件

可用于监听的表单事件如下表所示：

| 事件   | 描述                               |
| ------ | ---------------------------------- |
| focus  | 元素获取焦点后触发                 |
| blur   | 元素失去焦点后触发                 |
| input  | 元素内容发生改变后触发             |
| click  | 单击了某个表单项                   |
| change | 元素内容发生并且失去焦点改变时触发 |
| submit | 提交表单时触发                     |

注意，node.focus()和node.blur()可强制的让元素获取、失去焦点。

另外，如果input的type是submit，则可以调用submit()方法进行表单提交。

此外，如果input的type是file，则可以调用click()方法来选择文件。





# 鼠标事件

## 常用事件

可用于监听的鼠标事件如下表所示：

| 事件       | 描述                                         |
| ---------- | -------------------------------------------- |
| click      | 鼠标左键单击（同时触发mousedown以及mouseup） |
| dblclick   | 鼠标左键双击                                 |
| mousedown  | 鼠标左键、右键按下                           |
| mouseup    | 鼠标左键、右键松开                           |
| mouseover  | 鼠标移入时                                   |
| mousemove  | 鼠标移动时                                   |
| mouseout   | 鼠标移出时                                   |
| mouseenter | 鼠标移入时，不产生冒泡行为                   |
| mouseleave | 鼠标移出时，不产生冒泡行为                   |
| copy       | 拷贝时                                       |
| scroll     | 滚动时                                       |

测试代码：

```
<style>
    div:nth-child(1) {
        height: 200px;
        width: 200px;
        background-color: #ddd;
        overflow: scroll;
    }
</style>

<body>
    <div>content
        <div style="height: 1000px;width: 1000px;"></div>
    </div>
</body>

<script>

    "use strict";

    class MouseEventTest {
        constructor(node) {
            // 添加测试项
            node.addEventListener("copy", this);
            node.addEventListener("scroll", this);
        }

        handleEvent(event) {
            this[event.type](event.target, event);
        }

        click(target, event) {
            console.log("左键单击");
        }

        dblclick(target, event) {
            console.log("左键双击");
        }

        mousedown(target, event) {
            console.log("某个键按下");
        }

        mouseup(target, event) {
            console.log("某个键松开");
        }

        mouseover(target, event) {
            console.log("鼠标移入");
        }

        mousemove(target, event) {
            console.log("鼠标移动");
        }
        mouseout(target, event) {
            console.log("鼠标移出");
        }
        mouseenter(target, event) {
            console.log("鼠标移动入 不冒泡");
        }
        mouseleave(target, event) {
            console.log("鼠标移出, 不冒泡");
        }
        copy(target, event) {
            console.log("拷贝");
        }
        scroll(target, event) {
            console.log("滚动");
        }
    }

    new MouseEventTest(document.querySelector('div'))

</script>
```



## 事件属性

鼠标事件对象是MouseEvent，它派生自UIEvent，而UIEvent派生自Event。

该事件对象可调用属性如下：

| 属性                     | 描述                                                     |
| ------------------------ | -------------------------------------------------------- |
| MouseEvent.which         | 鼠标事件触发时按下的按键                                 |
| MouseEvent.button        | 鼠标事件触发时是否按下了某个鼠标按键，将返回1个数值      |
| MouseEvent.buttons       | 鼠标事件触发时是否按下了多个鼠标按键，将返回1个或N个数值 |
| MouseEvent.altKey        | 鼠标事件触发时是否按下了alt键，返回Boolean值             |
| MouseEvent.ctrlKey       | 鼠标事件触发时是否按下了ctrl键，返回Boolean值            |
| MouseEvent.shiftKey      | 鼠标事件触发时是否按下了shift键，返回Boolean值           |
| MouseEvent.metaKey       | 鼠标事件触发时是否按下了window键，返回Boolean值          |
| MouseEvent.offsetX       | 鼠标位于当前目标元素中的X坐标                            |
| MouseEvent.offsetY       | 鼠标位于当前目标元素中的Y坐标                            |
| MouseEvent.pageX         | 鼠标位于当前整个文档中的X坐标                            |
| MouseEvent.pageY         | 鼠标位于当前整个文档中的Y坐标                            |
| MouseEvent.clientX       | 鼠标位于当前整个窗口中的X坐标                            |
| MouseEvent.clientY       | 鼠标位于当前整个窗口中的Y坐标                            |
| MouseEvent.screenX       | 鼠标位于当前整个屏幕中的X坐标                            |
| MouseEvent.screenY       | 鼠标位于当前整个屏幕中的Y坐标                            |
| MouseEvent.relatedTarget | 鼠标事件的次要目标（如果有的话）                         |

relatedTarget属性主要针对mouseover、mouseout事件：

- mouseover：鼠标移入到当前元素之前在哪个元素身上
- mouseout：鼠标移出到当前元素之后在哪个元素身上
- 当无来源，或移动到窗口外时值为null



which属性的按键代号如下所示：

- 0：左键
- 1：中键
- 2：右键

在IE8之前版本中代号有所不同：

- 0：没有按下任何按键
- 1：按下左键
- 2：按下右键
- 3：同时按下左、右键
- 4：按下中键
- 5：同时按下左、中键
- 6：同时按下右、中键
- 7：3个按键同时按下



# 键盘事件

## 常用事件

可用于监听的键盘事件如下表所示：

| 事件     | 描述                               |
| -------- | ---------------------------------- |
| keydown  | 按下某个键、长按时将重复触发       |
| keyup    | 松开某个键                         |
| keypress | 按下并松开某个键、长按时将重复触发 |

测试代码：

```
<style>
    div:nth-child(1) {
        height: 200px;
        width: 200px;
        background-color: #ddd;
        overflow: scroll;
    }
</style>

<body>
    <textarea name="" id="" cols="30" rows="10"></textarea>
</body>

<script>

    "use strict";

    class MouseEventTest {
        constructor(node) {
            // 添加测试项
            node.addEventListener("keydown", this);
            node.addEventListener("keyup", this);
            node.addEventListener("keypress", this);
        }

        handleEvent(event) {
            this[event.type](event.target, event);
        }

        keydown(target, event) {
            console.log("按下或长按了");
        }

        keyup(target, event) {
            console.log("松开了");
        }

        keypress(target, event) {
            console.log("按下并松开了");
        }
    }

    new MouseEventTest(document.querySelector('textarea'))

</script>
```



## 事件属性

鼠标事件对象是KeyboardEvent，它派生自UIEvent，而UIEvent派生自Event。

该事件对象可调用属性如下：

| 属性                   | 描述                                            |
| ---------------------- | ----------------------------------------------- |
| KeyboardEvent.key      | 返回按键字符                                    |
| KeyboardEvent.code     | 返回按键编码                                    |
| KeyboardEvent.keyCode  | 返回按键ASCII码                                 |
| KeyboardEvent.altKey   | 键盘事件触发时是否按下了alt键，返回Boolean值    |
| KeyboardEvent.ctrlKey  | 键盘事件触发时是否按下了ctrl键，返回Boolean值   |
| KeyboardEvent.shiftKey | 键盘事件触发时是否按下了shift键，返回Boolean值  |
| KeyboardEvent.metaKey  | 键盘事件触发时是否按下了window键，返回Boolean值 |

KeyboardEvent.key释义：

- 按键字符，按键的字符含义表示，大小写不同。不能区分左右ALT等。不同语言操作系统下值会不同

KeyboardEvent.code释义：

- 按键编码，字符以Key开始，数字以Digit开始，特殊字符有专属名字。左右ALT键字符不同。 不同布局的键盘值会不同

