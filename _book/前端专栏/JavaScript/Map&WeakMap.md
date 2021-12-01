# Map

## 基本介绍

Map是一组键值对的结构，用于解决以往不能用对象做键的问题，它在ES6中被引入。

- Map拥有极快的查找速度
- 对比Object，它的键可以是任意类型

其实Map和Object非常相似，但是Object的键只能是String或者Symbol类型，Map在这方面就显得没有这么严苛。

以下是Map和Object的区别对比：

| 对比项   | Map                                                          | Object                                                       |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 意外的键 | Map默认情况下不包含任何键，只包含显式插入的键                | Object有一个prototype对象，原型链上的键名可能和你已有对象中的键名发生冲突，虽然ES5开始可以用object.create(null)来创建一个没有原型的对象，但是这种用法不太常见 |
| 键的类型 | Map的键可以是任意类型，包含函数、对象或任意基本类型          | Object中键的类型必须是String或者Symbol                       |
| 键的顺序 | Map中的键是有序的，先插入的k-v排列在前，后插入的k-v排列在后  | Object中的键是无序的。注意：自ECMAScript 2015规范以来，对象确实保留了字符串和Symbol键的创建顺序； 因此，在只有字符串键的对象上进行迭代将按插入顺序产生键 |
| Size     | Map的键值对个数可以通过size属性轻松的进行获取                | Object的键值对个数只能手动进行计算                           |
| 迭代     | Map遵循了 [可迭代协议](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols)，所以可以直接被迭代 | Object需要手动的创建出迭代器后才能够对其进行迭代             |
| 性能     | 在频繁增删键值对的场景下表现更好                             | 在频繁添加和删除键值对的场景下未作出优化                     |



## 声明定义

使用Map()加上嵌套的二维数组即可声明出一个Map容器：

```
"use strict";

let map = new Map([
    ["k1", "v1"],
    ["k2", "v2"],
    ["k3", "v3"],
]);

console.log(map instanceof Map);
console.log(map);
console.log(map.get("k1"));

// true
// Map { 'k1' => 'v1', 'k2' => 'v2', 'k3' => 'v3' }
// v1
```

Map的键必须是不同的对象，若相同内存地址的对象作为Map中的多个键，将会引发键冲突。

换而言之，Map中的键必须是唯一的。

1）ary1和ary2的内存地址不同，故不会发生键冲突：

```
"use strict";

let ary1 = [1, 2, 3];
let ary2 = [1, 2, 3];

let map = new Map([
    [ary1, "v1"],
    ["k2", "v2"],
    [ary2, "v3"],
]);

console.log(map instanceof Map);
console.log(map);

// true
// Map { [ 1, 2, 3 ] => 'v1', 'k2' => 'v2', [ 1, 2, 3 ] => 'v3' }
```

2）ary1和ary2的内存地址相同，将会引发键冲突：

```
"use strict";

let ary1 = [1, 2, 3];
let ary2 = ary1;

let map = new Map([
    [ary1, "v1"],
    ["k2", "v2"],
    [ary2, "v3"],
]);

console.log(map instanceof Map);
console.log(map);

// true
// Map { [ 1, 2, 3 ] => 'v3', 'k2' => 'v2' }
```



## 获取数量 size

使用size属性可获取当前Map容器中的键值对数量：

```
"use strict";

let map = new Map([
    ["k1", "v1"],
    ["k2", "v2"],
    ["k3", "v3"],
]);

console.log(map.size);

// 3
```



## 新增数据 set()

使用set()方法为Map容器新增一组键值对：

```
"use strict";

let map = new Map();

for (let i = 1; i < 4; i++) {
    map.set(`k${i}`, `v${i}`);
}

console.log(map);

// Map { 'k1' => 'v1', 'k2' => 'v2', 'k3' => 'v3' }
```



## 元素检测 has()

使用has()方法可检测当前Map容器中是否存在某个键，返回布尔值。

- 注意，如果键是引用类型则必须要使用原本键对象的引用才行，单纯的形式相似总会返回false

```
"use strict";

let ary = [1, 2, 3];

let map = new Map([
    [ary, "v1"],
    ["k2", "v2"],
    ["k3", "v3"],
]);

// 同一引用，返回true
console.log(map.has(ary));

// 非同一引用，返回false
console.log(map.has([1, 2, 3]));

// true
// false
```





## 获取元素 get()

使用get()方法根据键获取值，若键不存在则返回undefined：

```
"use strict";

let map = new Map([
    ["k1", "v1"],
    ["k2", "v2"],
    ["k3", "v3"],
]);

console.log(map.get("k1"));
console.log(map.get("k8"));


// v1
// undefined
```





## 删除元素 delete()

使用delete()方法根据键来删除Map容器中的一组键值对。

若删除成功则返回true，否则返回false：

```
"use strict";

let map = new Map([
    ["k1", "v1"],
    ["k2", "v2"],
    ["k3", "v3"],
]);

console.log(map.delete("k8"));
console.log(map.delete("k1"));


// false
// true
```





## 清空容器 clear()

使用clear()方法可清空Map容器：

```
"use strict";

let map = new Map([
    ["k1", "v1"],
    ["k2", "v2"],
    ["k3", "v3"],
]);

map.clear();

console.log(map);

// Map {}
```



## 数组转换

可使用…语法或者Array.form()将Map容器转换为二维数组：

```
"use strict";

let map = new Map([
    ["k1", "v1"],
    ["k2", "v2"],
    ["k3", "v3"],
]);

let ary1 = new Array(...map);
let ary2 = Array.from(map);

console.log(ary1);
console.log(ary2);

// [ [ 'k1', 'v1' ], [ 'k2', 'v2' ], [ 'k3', 'v3' ] ]
// [ [ 'k1', 'v1' ], [ 'k2', 'v2' ], [ 'k3', 'v3' ] ]
```



# 遍历操作

## 迭代器方法

使用以下3个方法都可创建出Map迭代器。

- kes()
- values()
- entries()

示例如下：

```
"use strict";

let map = new Map([
    ["k1", "v1"],
    ["k2", "v2"],
    ["k3", "v3"],
]);

console.log(map.keys());
console.log(map.values());
console.log(map.entries());

// [Map Iterator] { 'k1', 'k2', 'k3' }
// [Map Iterator] { 'v1', 'v2', 'v3' }
// [Map Iterator] { [ 'k1', 'v1' ], [ 'k2', 'v2' ], [ 'k3', 'v3' ] }
```



## for/of

由于Map本身属于可迭代对象，所以你可以直接使用for/of对其进行遍历操作。

这么做将拿到k-v，它们会以数组的形式返回：

```
"use strict";

let map = new Map([
    ["k1", "v1"],
    ["k2", "v2"],
    ["k3", "v3"],
]);

for(let kv of map){
    console.log(kv);
}

// [ 'k1', 'v1' ]
// [ 'k2', 'v2' ]
// [ 'k3', 'v3' ]
```



## forEach()

使用forEach()方法也可对Map容器进行遍历操作。

它与Array的forEach()方法使用基本相同。

```
"use strict";

let map = new Map([
    ["k1", "v1"],
    ["k2", "v2"],
    ["k3", "v3"],
]);

map.forEach((key, value) => {
    console.log(key, value);
})

// v1 k1
// v2 k2
// v3 k3
```



# WeakMap

## 基本介绍

WeakMap和Map非常相似，它也是根据键值对的形式来进行数据的存取。

但它和Map也有一些不同之处，如下所示：

- WeakMap中的键必须是对象（引用类型）
- WeakMap中的键是无序的
- WeakMap中的键是弱引用的，也就是说WeakMap中的键并不会增加键源对象的引用计数
- 在GC机制运行时，如果键源对象在其他地方被删除，WeakMap中也会删除掉该键值对
- 因为WeakMap中的键是弱引用的，故WeakMap不包含keys()、values()、entries()等方法和size属性，同时WeakMap是不可被迭代的

由于WeakMap中的键是弱引用的特性，故我们可以用它来保存一些经常存取的数据。

当键源对象在外部被删除后，WeakMap中的键值对也会删除，以此达到动态视图的功能。



## 声明定义

使用WeakMap()加上嵌套的二维数组即可声明出一个WeakMap容器。

- 注意！键必须是引用类型，若是值类型则会抛出异常

```
"use strict";

let ary = [1, 2, 3];
let obj = { k1: "v1", k2: "v2" };
let map = new Map([["k1", "v1"], ["k2", "v2"]]);

let wMap = new WeakMap([
    [ary, "v1"],
    [obj, "v2"],
    [map, "v3"],
])

console.log(wMap);

// WeakMap {{…} => "v2", Map(2) => "v3", Array(3) => "v1"}
```

WeakMap的键必须是不同的对象，若相同内存地址的对象作为WeakMap中的多个键，将会引发键冲突。

换而言之，WeakMap中的键必须是唯一的。

1）ary1和ary2的内存地址不同，故不会发生键冲突：

```
"use strict";

let ary1 = [1, 2, 3];
let ary2 = [1, 2, 3];
let obj = { k1: "v1", k2: "v2" };

let wMap = new WeakMap([
    [ary1, "v1"],
    [obj, "v2"],
    [ary2, "v3"],
])

console.log(wMap);

// WeakMap {{…} => "v2", Array(3) => "v1", Array(3) => "v3"}
```

2）ary1和ary2的内存地址相同，将会引发键冲突：

```
"use strict";

let ary1 = [1, 2, 3];
let ary2 = ary1;
let obj = { k1: "v1", k2: "v2" };

let wMap = new WeakMap([
    [ary1, "v1"],
    [obj, "v2"],
    [ary2, "v3"],
])

console.log(wMap);

// WeakMap {Array(3) => "v3", {…} => "v2"}
```





## 常用操作

WeakMap容器仅包含3个可用的方法：

- set()：新增一组键值对，其中键必须是引用类型
- delete()：删除一组键值对
- has()：根据键判断键值对是否存在于WeakMap容器中



## 弱引用

对于GC机制的引用计数法来说，存在于WeakMap中的对象键并不会为其增加引用计数，因此这种特性也被称为弱引用。

当WeakMap中对象键在外部的引用计数为0后，WeakMap内部中该键值对也会被清除，如下所示：

```
"use strict";

let ary = [1, 2, 3];
let obj = { k1: "v1", k2: "v2" };
let map = new Map([["k1", "v1"], ["k2", "v2"]]);

let wMap = new WeakMap([
    [ary, "v1"],
    [obj, "v2"],
    [map, "v3"],
])

console.log(wMap);

// 引用计数归0
ary = undefined;

// 等待GC机制运行，3s后打印结果
setTimeout(() => {
    console.log(wMap);
}, 3000);

// WeakMap {{…} => "v2", Map(2) => "v3", Array(3) => "v1"}
// WeakMap {{…} => "v2", Map(2) => "v3"}
```

