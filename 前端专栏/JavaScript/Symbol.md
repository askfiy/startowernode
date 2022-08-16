# Symbol介绍

Symbol对象是ES6中新引进的一种数据类型，主要作为对象的键而使用。

在ES6之前，对象的键必须是String类型，且不能重复。

在ES6之后，对象的键多了一种选择即Symbol对象，Symbol对象最大的特点就是具有唯一性。



# 唯一型Symbol

## 基本声明

Symbol没有构造函数，这使得我们不能new它，而是直接使用Symbol()即可。

Symbol对象具有唯一性，每个创建出的Symbol对象都是独一无二的：

```
"use strict";

let first = Symbol();
let last = Symbol();

console.log(typeof first, typeof last);
console.log(first === last);

// symbol symbol
// false
```

如图所示：

![image-20210730162145531](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210730162145531.png)



## 描述信息

在声明唯一型Symbol对象时，可以添加一段描述信息。

后面可以对唯一型Symbol对象使用description属性来获取这段描述信息，若没有描述信息时返回undefined。

如下所示，2个Symbol对象是不同的，但他们的描述信息是相同的：

```
"use strict";

let first = Symbol("this is a description information");
let last = Symbol("this is a description information");


console.log(first === last);
console.log(first.description === last.description);

// false
// true
```



# 引用型Symbol

## 基本声明

Symbol.for()方法用于创建一个全局引用的Symbol对象，后续再通过该方法创建的Symbol对象会引用第一个Symbol对象。

```
"use strict";

let first = Symbol.for();
let last = Symbol.for();

console.log(typeof first, typeof last);
console.log(first === last);

// symbol symbol
// true
```

如图所示：

![image-20210730163011602](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210730163011602.png)

若2个Symbol.for()创建出的对象描述信息不同，则会认为是不同的Symbol对象：

```
"use strict";

let first = Symbol.for("X");
let last = Symbol.for("Y");

console.log(typeof first, typeof last);
console.log(first === last);

// symbol symbol
// false
```

如图所示：

![image-20210730163949459](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210730163949459.png)

## 描述信息

在声明引用型Symbol对象时，可以添加一段描述信息。

后面可以对引用型Symbol对象使用Symbol.keyFor()方法来获取这段描述信息，若没有描述信息时返回undefined。

注意，如果使用description属性获取引用型Symbol的描述信息的话，得到的结果永远是undefined：

```
"use strict";

let first = Symbol.for("this is a first symbol description information");

console.log(first.description);
console.log(Symbol.keyFor(first));

// undefined
// this is a first symbol description information
```



# 对象与Symbol

## 对象的Symbol键

JavaScript中对象的键如果直接声明内部会将其转变为String类型，这在某种程度上可能会引起键冲突问题。

对象键最好是唯一的，因此唯一型Symbol对象是最好的选择。

当我们想使用Symbol作为对象键时，需要注意1点：

- Symbol声明的对象键在访问使用时都不能利用.点语法进行操作，而是用利用[]语法进行操作，因为.点语法是操作字符串属性的

如下所示：

```
"use strict";

"use strict";

let userNameKey = Symbol("userNameKey");
let userAgeKey = Symbol("userAgeKey");
let userGenderKey = Symbol("userGenderKey");

let userMessage = {
    [userNameKey] : "Jack",
    [userAgeKey] : 18,
    [userGenderKey] : "male"
}

console.log(userMessage[userNameKey]);
console.log(userMessage[userAgeKey]);
console.log(userMessage[userGenderKey]);

// Jack
// 18
// male
```



## 遍历获取

Symbol类型的值不能被for/in、for/of遍历到。

普通的for/in、for/of、迭代器方法仅能获取对象下的String键：

```
"use strict";

let userNameKey = Symbol("userNameKey");
let userAgeKey = Symbol("userAgeKey");
let userGenderKey = Symbol("userGenderKey");

let userMessage = {
    [userNameKey] : "Jack",
    [userAgeKey] : 18,
    [userGenderKey] : "male",
    "hobby" : ["basketball", "football", "volleyball"]
}

for(let k in userMessage){
    console.log(userMessage[k]);
}

for(let v of Object.values(userMessage)){
    console.log(v);
}

// [ 'basketball', 'football', 'volleyball' ]
// [ 'basketball', 'football', 'volleyball' ]
```



## 仅获取Symbol键

使用Object.getOwnPropertySymbols()获取对象下的所有Symbol键，注意仅是获取Symbol键而已：

```
"use strict";

let userNameKey = Symbol("userNameKey");
let userAgeKey = Symbol("userAgeKey");
let userGenderKey = Symbol("userGenderKey");

let userMessage = {
    [userNameKey]: "Jack",
    [userAgeKey]: 18,
    [userGenderKey]: "male",
    "hobby": ["basketball", "football", "volleyball"]
}

for (let k of Object.getOwnPropertySymbols(userMessage)) {
    console.log(userMessage[k]);
}

// Jack
// 18
// male
```



## 获取所有的键

使用Reflect.ownKeys()获取对象下的所有键，注意仅是获取键而已：

```
"use strict";

let userNameKey = Symbol("userNameKey");
let userAgeKey = Symbol("userAgeKey");
let userGenderKey = Symbol("userGenderKey");

let userMessage = {
    [userNameKey]: "Jack",
    [userAgeKey]: 18,
    [userGenderKey]: "male",
    "hobby": ["basketball", "football", "volleyball"]
}

for (let k of Reflect.ownKeys(userMessage)) {
    console.log(userMessage[k]);
}

// [ 'basketball', 'football', 'volleyball' ]
// Jack
// 18
// male
```





## 半私有属性

我们可以使用Symbol不能被for/in以及for/of访问的特性，为类制作半私有属性以及提供访问接口。

```
"use strict";

let userAgeKey = Symbol("user userAge key");
let userGenderKey = Symbol("user gender key");

class Person {

    constructor(userName, userAge, userGender) {
        this.userName = userName;
        this[userAgeKey] = { userAge };
        this[userGenderKey] = { userGender };
    }

    getInfo() {
        // 仅能内部访问
        return `name : ${this.userName}\nage : ${this[userAgeKey].userAge}\ngender : ${this[userGenderKey].userGender}`
    }

}

let ins = new Person("Jack", 18, "male");

console.log(ins.getInfo());

// 外部若想访问，只能扒源码来找Symbol键
console.log(ins[userGenderKey].userGender);

// name : Jack
// age : 18
// gender : male

// male
```

