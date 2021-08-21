# 基本声明

boolean的值被称为布尔值，常用于分支流程中，仅有2种表现形式：

- true：代表条件为真
- false：代表条件为假

以下是使用类实例化的形式进行对象声明：

```
"use strict";

let bool = new Boolean(true);
console.log(`value : ${bool}\ntype : ${typeof bool}`);

// value : true
// type : boolean
```

也可以选择使用更方便的字面量形式进行对象声明：

```
"use strict";

let bool = true;
console.log(`value : ${bool}\ntype : ${typeof bool}`);

// value : true
// type : boolean
```

若new Boolean()时没有传入参数，则生成false：

```
"use strict";

let bool = new Boolean();
console.log(`value : ${bool}\ntype : ${typeof bool}`);

// value : false
// type : boolean
```



# 类型转换

## 显式转换

使用 !! 或Boolean()将任意类型对象包裹均能获得其布尔值的表现形式。

- !代表取反，2个!!刚好获得布尔值对象

如下所示：

```
"use strict";

console.log(!![]);
console.log(!!{});
console.log(!!"");
console.log(!!"0");
console.log(!!"1");
console.log(!!0);
console.log(!!1);

// true
// true
// false
// true
// true
// false
// true
```



## 隐式转换

基本上所有类型都可以隐式转换为Boolean类型。

1）boolen与number比较时，true等同于1，false等同于0：

```
"use strict";

console.log(3 > true);
console.log(0 > false);

// true
// false
```

2）boolen与数字串比较时，两边都会转换为number后再进行比较：

```
"use strict";

console.log("3" > true);
console.log("0" > false);

// true
// false
```

3）boolen与array比较时，两边都会转换为number后再进行比较：

```
"use strict";

console.log([3] > true);
console.log([0] > false);

// true
// false
```

4）NaN和布尔值的比较结果都是false：

```
"use strict";

console.log(NaN == true);
console.log(NaN == false);

// false
// false
```

5）非空的string布尔值为true、空的string布尔值为false：

```
"use strict";

console.log(!!"str");
console.log(!!"");

// true
// false
```

6）引用类型的布尔值总为true，无论它其中是否有值，如array和object，以及map和set：

```
"use strict";

console.log(!![]);
console.log(!![1, 2, 3]);
console.log(!!{});
console.log(!!{ x: 1, y: 2 });
console.log(!!new Map());
console.log(!!new Map([["x", 1], ["y", 2]]));
console.log(!!new Set());
console.log(!!new Set([1, 1, 2, 2, 3, 3]));

// true
// true
// true
// true
// true
// true
// true
// true
```

7）要判断容器类型是否为空，则需要通过length或者size以及其他方法进行判断，如下所示：

```
"use strict";

console.log(!![].length);
console.log(!![1, 2, 3].length);
console.log(!!Object.keys({}).length);
console.log(!!Object.keys({ x: 1, y: 2 }).length);
console.log(!!new Map().size);
console.log(!!new Map([["x", 1], ["y", 2]]).size);
console.log(!!new Set().size);
console.log(!!new Set([1, 1, 2, 2, 3, 3]).size);

// false
// true
// false
// true
// false
// true
// false
// true
```

