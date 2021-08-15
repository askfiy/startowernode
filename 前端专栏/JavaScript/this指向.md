# this是什么

this是JavaScript中的一个关键字。

一般我们会在函数或者方法中来对它进行使用。

this和Python中的self非常像，但是又不大一样，它使用范围更广，并且自身也代表多重含义。

对于新手来说this可能是非常难迈过的一道坎，因为它的变化实在太多了，但是当你真正理解它后其实不难掌握它变化的规律。





# 常见this指向情况

## 全局环境下的this

1）严格模式中，全局环境下this指向为window对象：

```
"use strict";

console.log(this);

// window
```

2）非严格模式中，全局环境下this指向为window对象：

```
console.log(this);

// window
```



## 普通函数的this

1）严格模式中，普通函数下this指向为undefined：

```
"use strict";

function show(){
    console.log(this);
}

show();

// undefined
```

2）非严格模式中，普通函数下this指向为window对象：

```
function show(){
    console.log(this);
}

show();

// window
```



## 箭头函数的this

箭头函数没有this，大部分情况下它会继承上层作用域的this。

1）严格模式中，箭头函数下this指向为window，因为它的上层是全局作用域：

```
"use strict";

let show = () => {
    console.log(this);
}

show();

// window
```

2）非严格模式中，箭头函数下this指向为window，因为它的上层是全局作用域：

```
let show = () => {
    console.log(this);
}

show();

// window
```



## 对象中的this

1）严格模式中，对象下this指向当前对象：

```
"use strict";

let obj = {
    show : function(){
        console.log(this);
    }
};

obj.show();

// { show: [Function: show] }
```

2）非严格模式中，对象下this指向当前对象：

```
let obj = {
    show : function(){
        console.log(this);
    }
};

obj.show();

// { show: [Function: show] }
```



## 构造函数的this

1）严格模式中，构造函数下this指向构造完成后的实例对象：

```
"use strict";

function A(param){
    this.param = param;
    console.log(this);
}

new A("x");
new A("y");

// A { param: 'x' }
// A { param: 'y' }
```

2）非严格模式中，构造函数下this指向构造完成后的实例对象：

```
function A(param){
    this.param = param;
    console.log(this);
}

new A("x");
new A("y");

// A { param: 'x' }
// A { param: 'y' }
```



## 普通方法的this

1）严格模式中，对象的普通方法下this指向当前对象：

```
"use strict";

let obj = {
    show : function(){
        console.log(this);
    }
};

obj.show();

// { show: [Function: show] }
```

2）非严格模式中，对象的普通方法下this指向当前对象：

```
let obj = {
    show : function(){
        console.log(this);
    }
};

obj.show();

// { show: [Function: show] }
```



## 箭头方法的this

1）严格模式中，对象的箭头方法下this指向window对象，实际上它是跟随的obj指向，即跟随上层：

```
"use strict";

let obj = {
    show: () => {
        console.log(this);
    }
};

obj.show();

// window
```



2）非严格模式中，对象的箭头方法下this指向window对象，实际上它是跟随的obj指向，即跟随上层：

```
let obj = {
    show: () => {
        console.log(this);
    }
};

obj.show();

// window
```



## 普通闭函数的this

在包函数是个普通函数的情况下，若闭函数是普通函数则可能发生如下情况。

1）严格模式中，普通闭函数的this指向undefied：

```
"use strict";

let obj = {
    show: function () {
        (function(){
            console.log(this);
        }())
    }
};

obj.show();

// undefined
```

2）非严格模式中，普通闭函数的this指向window：

```
let obj = {
    show: function () {
        (function(){
            console.log(this);
        }())
    }
};

obj.show();

// window
```



## 箭头闭函数的this

在包函数是个普通函数的情况下，若闭函数是箭头函数则可能发生如下情况。

1）严格模式中，箭头闭函数的this跟随上层包函数的this指向：

```
"use strict";

let obj = {
    show: function () {
        let inner = () => {
            console.log(this);
        }
        inner();
    }
};

obj.show();

// { show: [Function: show] }
```

2）非严格模式中，箭头闭函数的this跟随上层包函数的this指向：

```
let obj = {
    show: function () {
        let inner = () => {
            console.log(this);
        }
        inner();
    }
};

obj.show();

// { show: [Function: show] }
```



## 普通事件回调函数的this

1）严格模式中，普通事件回调函数的this指向事件源本身，即元素本身：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <button>click me</button>

    <script>
        "use strict";
        document.querySelector("button").addEventListener("click", function (event) {
            console.log(this);  // <button>click me</button>
        })
    </script>
</body>

</html>
```

2）非严格模式中，普通事件回调函数的this指向事件源本身，即元素本身：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <button>click me</button>

    <script>
        document.querySelector("button").addEventListener("click", function (event) {
            console.log(this);  // <button>click me</button>
        })
    </script>
</body>

</html>
```



## 箭头事件回调函数的this

1）严格模式中，箭头事件回调函数的this指向window对象，实际上它是跟随的addEventListener指向，即跟随上层：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <button>click me</button>

    <script>
        "use strict";
        document.querySelector("button").addEventListener("click", event => {
            console.log(this);  // window
        })
    </script>
</body>

</html>
```

2）非严格模式中，箭头事件回调函数的this指向window对象，实际上它是跟随的addEventListener指向，即跟随上层：：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <button>click me</button>

    <script>
        document.querySelector("button").addEventListener("click", event => {
            console.log(this);  // window
        })
    </script>
</body>

</html>
```

若想在箭头事件回调函数中获取事件源本身，可使用event.target属性：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <button>click me</button>

    <script>
        "use strict";
        document.querySelector("button").addEventListener("click", event => {
            console.log(event.target);  // <button>click me</button>
            console.log(this);  // window
        })
    </script>
</body>

</html>
```



## 根据上下文改变指向

我们谨记一个规律即可，对象下的this永远指向当前对象（特殊情况下除外，如代理器）。

换个简单的说法，我们可以看谁调用了个方法，那么就是谁就是this。

示例1：

```
"use strict";

function show(){
    console.log(this);
}

// 等同于 window.show(); 故this为window; 但是严格模式下普通函数的this不能为window，所以是undefined

show();

// undefined
```

示例2：

```
"use strict";

let obj = {
    show(){
        console.log(this);
    }
}

// obj.show() 那么show中的this就指向obj

obj.show();

// { show: [Function: show] }
```

示例3：

```
"use strict";

let obj = {
    show: () => {
        console.log(this);
    }
}

// 等同于 window.obj.show()，show是箭头函数，跟随上层obj的指向，故指向window

obj.show();

// window
```

示例4：

```
"use strict";

let obj = {
    show: function(){
        let inner = function(){
            console.log(this);
        }
        // 虽然let声明的对象不会存放到window中，但若inner()前面没有任何调用者，则默认还是window
        // 但是严格模式下普通函数的this不能为window，所以是undefined
        inner();
    }
}

obj.show();

// undefined
```

示例5：

```
"use strict";

let obj = {
    show: function(){
        let self = this;
        let inner = function(){
            // 向上查找，this为obj对象
            self.func();
        }
        inner();
    },
    func:function(){
        // 等同于obj.func() 故结果是obj对象
        console.log(this);
    }
}

obj.show();

// { show: [Function: show], func: [Function: func] }
```

示例6：

```
"use strict";

function show() {
    console.log(this);
}

let obj = { show };

// 等同于 window.show(); 故this为window; 但是严格模式下普通函数的this不能为window，所以是undefined
show(); // undefined

// obj.show() 那么show中的this就指向obj
obj.show(); // { show: [Function: show] }
```

相信这6个例子看完后，你对this的指向已经了如指掌了。



# 改变this指向

## call()

通过call()方法，让原本函数的this指向发生变化。

- thisArg：新的this指向对象
- argArray：需要为函数传递的参数

特点：立即执行。

如下示例，通过call()方法改变函数setAttribute()的this指向，让它为obj增添属性：

```
"use strict";

function setAttribute(userName, userAge){
    this.userName = userName;
    this.userAge = userAge;
}

let obj = {}

// 将setAttribute的this指向改为obj，达到为obj增添属性的功能
setAttribute.call(obj, "Jack", 19);

console.log(obj);

// { userName: 'Jack', userAge: 19 }
```



## apply()

apply()方法和call()的特性以及参数都一样，唯一不同的是在为函数传递参数时必须传递数组。

- thisArg：新的this指向对象
- argArray：需要为函数传递的参数，必须传递数组，内部会进行解构

特点：立即执行。

如下示例，通过apply()方法改变函数setAttribute()的this指向，让它为obj增添属性：

```
"use strict";

function setAttribute(userName, userAge){
    this.userName = userName;
    this.userAge = userAge;
}

let obj = {}

// 必须传递数组
setAttribute.apply(obj, ["Jack", 19]);

console.log(obj);

// { userName: 'Jack', userAge: 19 }
```



## bind()

bind()方法也可以用来改变this指向，不同的是它会返回一个新的函数，也就是说它不是立即执行的。

bind()方法的参数传递有很多种，我们只需要记住一种即可，如下示例：

```
"use strict";

function setAttribute(userName, userAge){
    this.userName = userName;
    this.userAge = userAge;
}

let obj = {}

// 返回新函数
// 第一步，告诉需要将this对象改变成谁
let newSetAttribute = setAttribute.bind(obj);

// 第二步，调用新函数，参数正常传递即可
newSetAttribute("Jack", 18)

console.log(obj);

// { userName: 'Jack', userAge: 19 }
```





## 它们能做什么

上述介绍的三个方法用处非常大，搭配原型链可做到方法借用。

String本身没有forEach()方法，但是Array却有，我们利用上述3个方法改变this指向做到方法借用：

```
"use strict";

let str = "ABCDEF"

// 借用Array原型对象中的forEach方法，改变this指向为str，并绑定回调即可
Array.prototype.forEach.call(str, (value, index, str) => {
    console.log(value, index);
});

// A 0
// B 1
// C 2
// D 3
// E 4
// F 5
```



# 图表总结

以下是常见this指向情况的表总结，针对实在不能理解this指向的同学，下面的情况可以预防90%的情况：

| 调用环境         | window对象                                | undefined   | 当前对象或未来返回的实例对象 | 上层this指向       | 事件源DOM对象      |
| ---------------- | ----------------------------------------- | ----------- | ---------------------------- | ------------------ | ------------------ |
| 全局环境         | 严格/非严格模式：√                        |             |                              |                    |                    |
| 普通函数         | 非严格模式：√                             | 严格模式：√ |                              |                    |                    |
| 箭头函数         | 因为跟随上层this指向，所以这里是√         |             |                              | 严格/非严格模式：√ |                    |
| 对象             |                                           |             | 严格/非严格模式：√           |                    |                    |
| 构造函数         |                                           |             | 严格/非严格模式：√           |                    |                    |
| 普通方法         |                                           |             | 严格/非严格模式：√           |                    |                    |
| 箭头方法         |                                           |             |                              | 严格/非严格模式：√ |                    |
| 普通闭函数       | 非严格模式：√                             | 严格模式：√ |                              |                    |                    |
| 箭头闭函数       |                                           |             |                              | 严格/非严格模式：√ |                    |
| 普通事件回调函数 |                                           |             |                              |                    | 严格/非严格模式：√ |
| 箭头事件回调函数 | 因为跟随上层this指向，所以这里很大概率是√ |             |                              | 严格/非严格模式：√ |                    |

若想了解更多，可参照官方文档和阮一峰大神的文章。

[官方文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this)

[this指向的原理](https://www.ruanyifeng.com/blog/2018/06/javascript-this.html)

