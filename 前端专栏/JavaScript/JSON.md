# 基本介绍

JSON格式是目前最火热的跨语言交互格式，它十分利于人的阅读和编写。

使用JSON数据格式是替换XML最佳方式，主流语言都很好的支持JSON格式。

JSON标准中要求使用双引号包裹属性，虽然有些语言不强制，但使用双引号可避免多程序间传输发生错误语言错误的发生。







# 序列化

序列化是指将数据转换为JSON字符串，一般是当前语言向其他语言传递信息时使用。

JavaScript中JSON序列化方法为JSON.stringify()。

参数释义：

- value：序列化的对象
- replacer：指定保存的属性，如果为null则代表所有
- space：格式化后对table空格的定义，默认是4空格

代码示例：

```
"use strict";

let obj = {
    username: "Jack",
    age: 18,
    gender: "male"
};

jString = JSON.stringify(
    obj,
    ["username", "age"],  // 仅保留username、age属性
    8                     // table空格为8
);

console.log(jString);

// {
//     "username": "Jack",
//     "age": 18
// }
```





# 反序列化

反序列化是指将JSON字符串转换为JavaScript中的数据类型，一般是当前语言接收其他语言传递信息时使用。

JavaScript中JSON反序列化方法为JSON.parse()。

参数释义：

- text：JSON字符串
- reviver：回调函数，接收key、value参数，用于对反序列化出的数据进行二次处理

代码示例：

```
"use strict";

let obj = {
    username: "Jack",
    age: 18,
    gender: "male"
};

jString = JSON.stringify(obj);

str = JSON.parse(jString, (k, v) => {
    if (k === "gender") {
        return v === "male" ? "男" : "女"
    }
    return v;
})

console.log(str);

// { username: 'Jack', age: 18, gender: '男' }
```

