# class

## 基本介绍

严格来说，JavaScript中是没有类这一个概念的。

ES6之前，我们可以通过构造函数模拟出其他语言中的类。

ES6之后，JavaScript引入了关键词class，使用它来书写面向对象的程序比用构造函数来书写面向对象的程序更加直观也更加方便。

class仅作为语法糖参与代码编写，它不是其他语言中你所认为的类，它归根结底其实还是函数。

- 使用class语法糖定义类，其书写的方法会全部写入到原型对象中，这个非常方便
- class语法糖中书写的方法是不会被for/in遍历出来的，但是构造函数的书写方法for/in是能遍历出方法的
- class代码块中默认使用严格模式进行代码执行



## 简单使用

使用class来定义类，命名应采用大驼峰式风格。

此外你还需要在类中书写constructor()构造方法，以接收实例对象时外部所传递的值。

类中的方法不必加上function前缀且会自动写入原型对象中，也不用使用逗号进行分割。

如下示例：

```
"use strict";

class Person {

    constructor(userName, userAge, userGender) {
        this.userName = userName;
        this.userAge = userAge;
        this.userGender = userGender;
    }

    getInfo() {
        return `name : ${this.userName}\nage : ${this.userAge}\ngender : ${this.userGender}`
    }

}

let ins = new Person("Jack", 18, "male");
console.log(ins);

// Person { userName: 'Jack', userAge: 18, userGender: 'male' }
```







## 构造方法

constructor()构造方法不是必须要写的，只有当你需要为实例对象传递参数时才需要写上他。

即使你不写它系统内部也会通过super在原型链上找到该方法并执行。

相当于下面这条代码：

```
constructor(...args){
    super(...args);
}
```





## 遍历差异

class中书写的方法是不会被遍历出来的，而构造函数的原型对象中书写的方法却可以被遍历出来。

这也是2者之间比较大的一个差异：

```
"use strict";

class Person {

    constructor(userName, userAge, userGender) {
        this.userName = userName;
        this.userAge = userAge;
        this.userGender = userGender;
    }

    getInfo() {
        return `name : ${this.userName}\nage : ${this.userAge}\ngender : ${this.userGender}`
    }

}

let ins = new Person("Jack", 18, "male");

for(let attr in ins){
    console.log(ins[attr]);
}

// Jack
// 18
// male
```



## 严格模式

class中书写的代码默认使用strict严格模式进行执行：

```
class A {
    show() {
        (function () { console.log(this); }())  // 严格模式，闭函数指向undefined
    }
}

let ins = new A();
ins.show();
```



## 原理剖析

JavaScript中没有类，所以class归根结底还是函数。

如下，这是class语法编写的类，获取它的类型结果为function：

```
"use strict";

class Person {

    constructor(userName, userAge, userGender) {
        this.userName = userName;
        this.userAge = userAge;
        this.userGender = userGender;
    }

    getInfo() {
        return `name : ${this.userName}\nage : ${this.userAge}\ngender : ${this.userGender}`
    }

}

console.log(typeof Person);

// function
```

上面这个定义与下面的构造函数定义相同：

```
"use strict";

function Person(userName, userAge, userGender) {
    this.userName = userName;
    this.userAge = userAge;
    this.userGender = userGender;
}

Person.prototype.getInfo = function () {
    "use strict";
    return `name : ${this.userName}\nage : ${this.userAge}\ngender : ${this.userGender}`
};

Object.defineProperties(Person.prototype, {
    getInfo:{
        enumerable: false, // 不可被遍历
    }
});

let ins = new Person("Jack", 18, "male");

for(let attr in ins){
    console.log(ins[attr]);
}

// Jack
// 18
// male
```

两者一对比就会发现，class语法糖确实比构造函数+原型对象设置的方式书写代码要好很多。





# 类成员

## 类属性

类属性在JavaScript中其实应该称作静态属性，它不是为实例对象提供的而是为类本身提供的。

使用static进行声明即可：

```
"use strict";

class Person {

    // 如果不加static，则会变成实例属性
    static desc = "Humanity";

    constructor(userName, userAge, userGender) {
        this.userName = userName;
        this.userAge = userAge;
        this.userGender = userGender;
    }

    getInfo() {
        return `name : ${this.userName}\nage : ${this.userAge}\ngender : ${this.userGender}`
    }

}

console.log(Person.desc);

// Humanity
```

实现原理：

```
"use strict";

function Person(userName, userAge, userGender) {
    this.userName = userName;
    this.userAge = userAge;
    this.userGender = userGender;
}

Person.desc = "Humanity"

// ...

console.log(Person.desc);
```



## 类方法

类方法和类属性都差不多，同样使用static进行声明：

```
"use strict";

class Person {

    // 如果不加static，则会变成实例属性
    static desc = "Humanity";

    static help(){
        return "This is about class Person help information"
    }

    constructor(userName, userAge, userGender) {
        this.userName = userName;
        this.userAge = userAge;
        this.userGender = userGender;
    }

    getInfo() {
        return `name : ${this.userName}\nage : ${this.userAge}\ngender : ${this.userGender}`
    }

}

console.log(Person.help());

// This is about class Person help information
```

实现原理：

```
"use strict";

function Person(userName, userAge, userGender) {
    this.userName = userName;
    this.userAge = userAge;
    this.userGender = userGender;
}

Person.desc = "Humanity"
Person.help = function () {
    return "This is about class Person help information"
}

// ...

console.log(Person.help());

// This is about class Person help information
```





# 属性代理

## Getter/Setter

使用get和set关键字定义用户在访问某一属性、设置某一属性时做的操作。

如下示例，我们将实例属性userAge进行封装，并为其配置属性代理器set和get。

在设置时新的userAge值时类型必须是number，在获取时总是返回一个年龄段而不返回具体年龄：

```
"use strict";

class Person {

    // 定义绝对私有属性
    #userAge;

    constructor(userName, userAge, userGender) {
        this.userName = userName;
        // 绝对私有，不允许外部访问或继承
        this.#userAge = userAge;
        this.userGender = userGender;
    }

    set userAge(value) {
        if (typeof value === "number" && (value <= 100 && value > 0)) {
            this.#userAge = value;
            return
        }
        throw new Error("error! type or age range error");
    }

    get userAge() {
        let age = this.#userAge;
        switch (true) {
            case age < 18: return "juvenile";
            case age < 30: return "youth";
            case age < 60: return "middle";
            case age <= 100: return "elderly";
            default: return "unknown";
        }
    }

}

// 这里不会触发代理
let ins = new Person("Jack", 10, "male");

// 触发get
console.log(ins.userAge);

// 触发set
ins.userAge = 1001;

// youth
// Uncaught Error: error! type or age range error
```



## 静态属性

BMI指数是用来衡量一个人的体重与身高对健康影响的一个指标，计算公式为：

- BMI指数计算公式: BMI = 体重(kg) / (身高m**2)

- BMI正常值在20至25之间，超过25为超重，30以上则属肥胖

身高或体重是不断变化的，因而每次想查看BMI值都需要通过计算才能得到，但很明显BMI听起来更像是一个特征而非功能。

所以我们可以利用get关键字来定义一个伪属性，它其实是一个方法：

```
"use strict";

class Person {
    constructor(userName, userHight, userWeight) {
        this.userName = userName;
        this.userHight = userHight;
        this.userWeight = userWeight;
    }
    get bmi() {
        let v = this.userWeight / (this.userHight ** 2);
        switch (true) {
            case 20 < v < 25:
                return "normal"
            case v > 25:
                return "overweight"
            default:
                return "obesity"
        }
    }
}

// 这里不会触发代理
let ins = new Person("Jack", 1.75, 77);

// 触发get
console.log(ins.bmi);

// normal
```



# 私有公有

## public

public指不受保护的、公开的属性。类的内部或外部均可访问到该属性。

默认所有的属性都是public：

```
"use strict";

class Person {

    constructor(userName, userAge, userGender) {
        this.userName = userName;
        this.userAge = userAge;
        this.userGender = userGender;
    }

    getInfo() {
        return `name : ${this.userName}\nage : ${this.userAge}\ngender : ${this.userGender}`
    }

}

let ins = new Person("Jack", 18, "male");
console.log(ins);

// Person { userName: 'Jack', userAge: 18, userGender: 'male' }
```



## private

private指受保护的、私有的属性。仅能在类的内部访问，外部是访问不到的，且不允许被继承。

private属性经常与属性代理一起使用，使用#关键字定义私有属性即可：

```
"use strict";

class Person {
    // 1.声明私有属性
    #userAge;
    #userGender;

    constructor(userName, userAge, userGender) {
        this.userName = userName;

        // 2.使用私有属性
        this.#userAge = userAge;
        this.#userGender = userGender;
    }

    getInfo() {
        if (this.#check){
            return `name : ${this.userName}\nage : ${this.#userAge}\ngender : ${this.#userGender}`
        }
        // 如果信息不全，则抛出异常
        throw new Error("Incomplete information")
    }


    // 3.私有方法不需要声明 验证信息是否齐全
    #check() {
        return this.userName && this.#userAge && this.#userGender
    }

}

let ins = new Person("Jack", 18, "male");
console.log(ins.getInfo());

// ins.#check()
// Uncaught SyntaxError: Private field '#check' must be declared in an enclosing class

// ins.#userAge
// Uncaught SyntaxError: Private field '#userAge' must be declared in an enclosing class
```



## protected

protected是指受保护的，半私有的属性。仅能在类的内部访问，外部是访问不到的，但它允许被继承

下面介绍三种半私有封装方式。



### 下划线封装法

这是一种君子约定的封装法，使用单下划线或者双下划线进行封装。

它其实并不会有任何强制性措施，只是告诉使用者，请不要在外部使用该属性。

它仅作为提示功能，就像吸烟时烟盒上出现的吸烟有害健康，但还是可以抽：

```
"use strict";

class Person {

    constructor(userName, userAge, userGender) {
        this.userName = userName;
        this._userAge = userAge;
        this.__userGender = userGender;
    }

    getInfo() {
        return `name : ${this.userName}\nage : ${this._userAge}\ngender : ${this.__userGender}`
    }

}

let ins = new Person("Jack", 18, "male");

// 外部依然可以访问，君子约定而已
console.log(ins._userAge);
console.log(ins.__userGender);

// 18
// male
```





### Symobl封装法

由于我们的代码都是在一个模块中进行封装的，所以使用Symbol()来进行私有封装非常的方便。

除非使用者打开源代码找到Symbol键，否则他只能通过提供的接口来拿到数据。

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





### WeakMap封装法

WeakMap有弱引用的特性，所以我们可以利用WeakMap来做私有封装。

它和Symbol封装法一样，使用者若想在外部访问被封装的实例变量，只能扒源码来找WeakMap容器：

```
"use strict";

let wMap = new WeakMap();

class Person {

    constructor(userName, userAge, userGender) {
        this.userName = userName;
        wMap.set(this, {
            userAge,
            userGender,
        })
    }

    getInfo() {
        // 仅能内部访问
        return `name : ${this.userName}\nage : ${wMap.get(this).userAge}\ngender : ${wMap.get(this).userGender}`
    }

}

let ins = new Person("Jack", 18, "male");

console.log(ins.getInfo());

// 外部若想访问，只能扒源码来找wMap容器
console.log(wMap.get(ins).userGender);

// name : Jack
// age : 18
// gender : male

// male
```





# 类的继承

## extends

class通过extends关键字进行原型继承，JavaScript中只支持单继承不支持多继承。

注意，如果子类中有构造方法，那么一定要使用super()方法来调用父类的构造方法，否则将会抛出异常。

1）错误示例：

```
"use strict";

class A {
    constructor(param) {
        this.param = param;
    }
}

class B extends A {
    // 未使用super调用父类构造方法，抛出异常
    constructor(param) {
        this.param = param
    }
}

let ins = new B("x");
console.log(ins);

// ReferenceError: Must call super constructor in derived class before accessing 'this' or returning from derived constructor
```

2）正确示例：

```
"use strict";

class A {
    constructor(param) {
        this.param = param;
    }
}

class B extends A {
    constructor(param) {
        super(param);
    }
}

let ins = new B("x");
console.log(ins);

// B {param: "x"}
```

3）或者你也可以不为super()方法传参：

```
"use strict";

class A {
    show() {
        console.log("run show...");
    }
}

class B extends A {
    constructor(param) {
        super();
        this.param = param;
    }
}

let ins = new B("x");
console.log(ins);

// B {param: "x"}
```

内部原理如下：

```
"use strict";

function A() { };

A.prototype.show = function () {
    console.log("run show...");
}

function B(param) {
    A.call(this);
    this.param = param;
}

let ins = new B("x");
console.log(ins);

// B {param: "x"}
```



## super()

super表示从当前原型中执行方法。

- super一直指向当前对象
- 在构造方法中，super()一定要放在this声明的前面
- super只能在类或对象的方法中使用，而不能在函数中使用，也就是说你应当使用ES6的简写语法表示该函数是一个方法，而不是利用函数关键字function进行声明



1）正确的super使用案例：

```
"use strict";

class A {
    show() {
        console.log("run show...");
    }
}

class B extends A {
    constructor(param) {
        // 在构造方法中，super()一定要放在this声明的前面
        super();
        this.param = param;
    }
    run() {
        // super只能在类或对象的方法中使用，而不能在函数中使用
        return super.show();
    }
}

let ins = new B("x");
ins.run();

// run show...
```

2）其他的super使用案例：

```
"use strict";

let obj_1 = {
    show() {
        console.log("run show...");
    }
};

let obj_2 = {
    __proto__: obj_1,
    
    // 由于run采用简写形式，故可以使用super
    run() {
        return super.show();
    }
}

obj_2.run();

// run show...
```

3）失败的使用案例：

```
"use strict";

let obj_1 = {
    show() {
        console.log("run show...");
    }
};

let obj_2 = {
    __proto__: obj_1,
    
    // 出现问题，利用function定义了该方法就不能使用super
    run: function () {
        return super.show();
    }
}

obj_2.run();

// SyntaxError: 'super' keyword unexpected here
```





## 继承类成员

extends关键字也可以继承类成员：

```
"use strict";

class A {
    // 定义类属性
    static desc = "class A";
}

class B extends A{}

console.log(B.desc);

// class A
```

