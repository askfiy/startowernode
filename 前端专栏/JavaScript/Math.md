# 基本介绍

Math我们常叫它数学工具包，它没有构造函数所以不用new，直接使用其下所定义的数学或方法即可。

如：

```
"use strict";

console.log(Math.PI);

// 3.141592653589793
```



# 属性一览

以下是Math中提供的常用属性：

| 属性                                                         | 描述                                            |
| ------------------------------------------------------------ | ----------------------------------------------- |
| [E](https://www.w3school.com.cn/jsref/jsref_e.asp)           | 返回算术常量 e，即自然对数的底数（约等于2.718） |
| [LN2](https://www.w3school.com.cn/jsref/jsref_ln2.asp)       | 返回 2 的自然对数（约等于0.693）                |
| [LN10](https://www.w3school.com.cn/jsref/jsref_ln10.asp)     | 返回 10 的自然对数（约等于2.302）               |
| [LOG2E](https://www.w3school.com.cn/jsref/jsref_log2e.asp)   | 返回以 2 为底的 e 的对数（约等于 1.414）        |
| [LOG10E](https://www.w3school.com.cn/jsref/jsref_log10e.asp) | 返回以 10 为底的 e 的对数（约等于0.434）        |
| [PI](https://www.w3school.com.cn/jsref/jsref_pi.asp)         | 返回圆周率（约等于3.14159）                     |
| [SQRT1_2](https://www.w3school.com.cn/jsref/jsref_sqrt1_2.asp) | 返回返回 2 的平方根的倒数（约等于 0.707）       |
| [SQRT2](https://www.w3school.com.cn/jsref/jsref_sqrt2.asp)   | 返回 2 的平方根（约等于 1.414）                 |





# 方法一览

以下是Math中提供的常用方法：

| 方法                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [abs(x)](https://www.w3school.com.cn/jsref/jsref_abs.asp)    | 返回数的绝对值                                               |
| [acos(x)](https://www.w3school.com.cn/jsref/jsref_acos.asp)  | 返回数的反余弦值                                             |
| [asin(x)](https://www.w3school.com.cn/jsref/jsref_asin.asp)  | 返回数的反正弦值                                             |
| [atan(x)](https://www.w3school.com.cn/jsref/jsref_atan.asp)  | 以介于 -PI/2 与 PI/2 弧度之间的数值来返回 x 的反正切值       |
| [atan2(y,x)](https://www.w3school.com.cn/jsref/jsref_atan2.asp) | 返回从 x 轴到点 (x, y) 的角度（介于 -PI/2 与 PI/2 弧度之间） |
| [ceil(x)](https://www.w3school.com.cn/jsref/jsref_ceil.asp)  | 对数进行上舍入                                               |
| [cos(x)](https://www.w3school.com.cn/jsref/jsref_cos.asp)    | 返回数的余弦                                                 |
| [exp(x)](https://www.w3school.com.cn/jsref/jsref_exp.asp)    | 返回 e 的指数                                                |
| [floor(x)](https://www.w3school.com.cn/jsref/jsref_floor.asp) | 对数进行下舍入                                               |
| [log(x)](https://www.w3school.com.cn/jsref/jsref_log.asp)    | 返回数的自然对数（底为e）                                    |
| [max(x,y)](https://www.w3school.com.cn/jsref/jsref_max.asp)  | 返回 x 和 y 中的最高值                                       |
| [min(x,y)](https://www.w3school.com.cn/jsref/jsref_min.asp)  | 返回 x 和 y 中的最低值                                       |
| [pow(x,y)](https://www.w3school.com.cn/jsref/jsref_pow.asp)  | 返回 x 的 y 次幂                                             |
| [random()](https://www.w3school.com.cn/jsref/jsref_random.asp) | 返回 0 ~ 1 之间的随机数                                      |
| [round(x)](https://www.w3school.com.cn/jsref/jsref_round.asp) | 把数四舍五入为最接近的整数                                   |
| [sin(x)](https://www.w3school.com.cn/jsref/jsref_sin.asp)    | 返回数的正弦                                                 |
| [sqrt(x)](https://www.w3school.com.cn/jsref/jsref_sqrt.asp)  | 返回数的平方根                                               |
| [tan(x)](https://www.w3school.com.cn/jsref/jsref_tan.asp)    | 返回角的正切                                                 |
| [valueOf()](https://www.w3school.com.cn/jsref/jsref_valueof_math.asp) | 返回 Math 对象的原始值                                       |



# 取最大值 max()

max()方法用于求出众多参数中的最大值：

```
"use strict";

console.log(Math.max(1, 2, 3, 4));

// 4
```





# 取最小值 min()

min()方法用于求出众多参数中的最小值：

```
"use strict";

console.log(Math.min(1, 2, 3, 4));

// 1
```



# 对象求值 apply()

通过apply()搭配其他的一些方法，我们可以为数组、对象等进行求值。

如，求数组中的最大值：

```
"use strict";

let ary = [1, 2, 3, 4];

console.log(Math.min.apply(Math, ary));

// 1
```





# 向上取整 ceil()

使用ceil()方法进行向上取整，它针对小数，即小数位只要不为0都向上进一位：

```
"use strict";

let v = 33.01;
console.log(Math.ceil(v));


// 34
```



# 向下取整 floor()

使用floor()方法进行向下取整，它针对小数，即抹除掉小数位：

```
"use strict";

let v = 33.99;
console.log(Math.floor(v));


// 33
```



# 四舍五入 round()

使用round()方法进行四舍五入：

```
"use strict";

let v1 = 33.5;
let v2 = 33.4;
console.log(Math.round(v1));
console.log(Math.round(v2));

// 34
// 33
```



# 幂运算 pow()

使用pow()方法进行幂运算：

```
"use strict";

let v = 3;

// 等同于 3 ** 3，即 3*3*3
console.log(Math.pow(3, 3));

// 27
```





# 随机数 random()

1）直接使用random()方法会返回大于0小于1的浮点数：

```
"use strict";

let v = Math.random();

console.log(v);

// 0.20942996211521936
```

2）使用floor()方法搭配random()进行取整：

```
"use strict";

let v = Math.floor(Math.random() * 5);
console.log(v);
```

3）生成10 - 20 之间的整数，包括10，包括20：

```
// 公式 Math.floor(Math.random() * (max - min + 1)) + min

"use strict";

let v = Math.floor(Math.random() * (20 - 10 + 1)) + 10;
console.log(v);
```

4）生成10 - 20 之间的整数，包括10，不包括20：

```
// 公式 Math.floor(Math.random() * (max - min)) + min


"use strict";

let v = Math.floor(Math.random() * (20 - 10)) + 10;
console.log(v);
```

5）从数组中随机抽取一个元素项：

```
"use strict";

let ary = [1, 2, 3, 4, 5];

const idx = Math.floor(Math.random() * ary.length);

console.log(ary[idx]);
```

6）从数组中随机抽取索引 1 - 3 之间的元素，包括3：

```
"use strict";

let ary = [1, 2, 3, 4, 5];


const idx = Math.floor(Math.random() * 3) + 1;

console.log(ary[idx]);
```

7）从数组中随机抽取索引 1 - 3 之间的元素，不包括3：

```
"use strict";

let ary = [1, 2, 3, 4, 5];


const idx = Math.floor(Math.random() * 2) + 1;

console.log(ary[idx]);
```

