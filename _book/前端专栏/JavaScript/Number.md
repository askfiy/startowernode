# 基本声明

JavaScript中的Number类型包含整数和浮点数。

以下是使用类实例化的形式进行对象声明：

```
"use strict";

let age = new Number(12);
console.log(`value : ${age}\ntype : ${typeof age}`);

// value : 12
// type : number
```

也可以选择使用更方便的字面量形式进行对象声明：

```
"use strict";

let age = 12;
console.log(`value : ${age}\ntype : ${typeof age}`);

// value : 12
// type : number
```

若new Number()时没有传入参数，则生成0：

```
"use strict";

let age = new Number();
console.log(`value : ${age}\ntype : ${typeof age}`);

// value : 0
// type : number
```



# NaN

NaN表示一个无效的数值，它不能使用==与其他值做对比。

你可以通过Number.isNaN()方法或者Object.is()方法判定一个值是否属于NaN：

```
"use strict";

let v = 123.4;
console.log(Number.isNaN(v));
console.log(Object.is(v, NaN));

// false
// false
```



# 类型转换

## 基本转换

基本上所有类型都可以隐式的转换为Number类型，区别在于有的会转换为有效数值，有的会转换为无效数值NaN。

1）非数字串的string类型转换为number得到的结果是NaN，即无效数字：

```
"use strict";

console.log(
    Number("string")
);

// NaN
```

2）如果是纯数字串的string类型，则可以转换为number：

```
"use strict";

console.log(
    Number("1000"),
);

// 1000
```

3）true会转换为1，false会转换为0：

```
"use strict";

console.log(
    Number(true),
    Number(false)
);

// 1 0
```

4）空的array会转换为0，空的object会转换为NaN：

```
"use strict";

console.log(
    Number([]),
    Number({})
);

// 0 NaN
```

5）如果array的长度为1，且数据项为number或者数字字符串类型，会直接进行提取，否则将转换为NaN：

```
"use strict";

console.log(
    Number([100]),   // 长度为1，且数据项类型为number
    Number(["100"]), // 长度为1，且数据项类型为数字串
    Number(["100", 100]),
);

// 100 100 NaN
```

6）JavaScript是弱类型语言，故number类型可以和str的纯数字串做四则运算，加法运算得到的结果是string，其他的运算得到的结果都是number或NaN：

```
"use strict";

console.log(
    100 + "3",
    100 - "3",
    100 * "3",
    100 / "3"
);

// 1003 97 300 33.333333333333336
// string number number number
```







## 强转整数 parseInt()

如果一个string类型即包含数字又包含其他字符，则可通过parseInt()方法进行强制转换。

- parseInt()：提取字符串中的整数部分并将其转换为number类型

示例如下：

```
"use strict";

let digitalString = "123.4this is string"
console.log(parseInt(digitalString));

// 123
```



## 强转浮点数 parseFloat()

如果一个string类型即包含数字又包含其他字符，则可通过parseFloat()方法进行强制转换。

- parseFloat()：提取字符串中的浮点数部分并将其转换为number类型

示例如下：

```
"use strict";

let digitalString = "123.4this is string"
console.log(parseFloat(digitalString));

// 123.4
```



# 类型判断

## 整数判定 isInteger()

使用isInteger()方法判断一个值是否是整数。

返回一个布尔值。

```
"use strict";

let v = 123.4;
console.log(Number.isInteger(v));

// false
```



## NaN判定 isNaN()

使用isNaN()方法判断一个值是否有效。

返回一个布尔值。

```
"use strict";

let v = 123.4;
console.log(Number.isNaN(v));

// false
```



# 浮点舍入 toFixed()

使用toFixed()方法对数值进行四舍五入的操作，可指定保留小数点后的位数：

```
"use strict";

let v = 123.4344;
console.log(v.toFixed());
console.log(v.toFixed(2));

// 123
// 123.43
```



# 浮点精度

基本上所有的编程语言中浮点数都有精度问题，当然JavaScript也不例外：

```
"use strict";

let n1 = 0.1;
let n2 = 0.2;
let v = n1 + n2;

console.log(v);

// 0.30000000000000004
```

这是因为计算机以二进制处理值类型，上面的0.1和0.2转换为2进制后是无穷的：

```
"use strict";

let n1 = 0.1;
let n2 = 0.2;
let v = n1 + n2;

console.log(n1.toString(2));
console.log(n2.toString(2));

// 0.0001100110011001100110011001100110011001100110011001101
// 0.001100110011001100110011001100110011001100110011001101
```

我们可以使用toFixed()方法来进行小数截取：

```
"use strict";

let n1 = 0.1;
let n2 = 0.2;
let v = (n1 + n2).toFixed(2);

console.log(v);

// 0.30
```

或者用一些第三方库进行操作，如decimal.js：

```
// HTML部分
    <script src="https://cdn.bootcss.com/decimal.js/10.2.0/decimal.min.js" defer></script>
    <script src="./demo.js" defer></script>
    
// Js部分
"use strict";

let n1 = 0.1;
let n2 = 0.2;
let v = Decimal.add(n1, n2).valueOf();

console.log(v);

// 0.3
```

