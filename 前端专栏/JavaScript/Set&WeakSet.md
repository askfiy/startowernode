# Set

## 基本介绍

Set是一种具有去重特性的容器，它可以存入任意类型的数据。它在ES6中被引入。

Set常用来操纵数据，而非存储数据。

- 你可以将Set理解为只有key没有value的map
- Set中的元素会进行严格类型检测存储，即数字字符串不等于数字
- Set中存储的值具有唯一性，即Set本身能对元素进行去重
- Set中存入的元素是有序的，即先存入的元素在遍历时会先被遍历到



## 声明定义

使用Set()时传入一个数组即可声明出一个Set容器：

```
"use strict";

let set = new Set([
    new Number(1),
    new String("1"),
    new Boolean(true),
    new Array("A", "B", "C"),
    new Map([["k1", "v1", "k2", "v2"]])
]);

console.log(set instanceof Set);
console.log(set);

// true
// Set {
//   [Number: 1],
//   [String: '1'],
//   [Boolean: true],
//   [ 'A', 'B', 'C' ],
//   Map { 'k1' => 'v1' }
// }
```



## 去重特性

如果一个相同内存地址的对象多次被存入Set中，则只会保留一个。

1）ary1和ary2的内存地址不同，故不会触发Set的去重特性：

```
"use strict";

let ary1 = [1, 2, 3];
let ary2 = [1, 2, 3];

let set = new Set([
    ary1,
    ary2,
]);

console.log(set);

// Set { [ 1, 2, 3 ], [ 1, 2, 3 ] }
```

2）ary1和ary2的内存地址相同，将会触发Set的去重特性：

```
"use strict";

let ary1 = [1, 2, 3];
let ary2 = ary1;

let set = new Set([
    ary1,
    ary2,
]);

console.log(set);

// Set { [ 1, 2, 3 ] }
```





## 获取数量 size

使用size属性可获取当前Set容器中的元素数量：

```
"use strict";

let set = new Set([
    new Number(1),
    new String("1"),
    new Boolean(true),
    new Array("A", "B", "C"),
    new Map([["k1", "v1", "k2", "v2"]])
]);

console.log(set.size);

// 5
```



## 新增元素 add()

使用add()方法为Set容器新增元素项：

```
"use strict";

let set = new Set();

for (let i = 1; i < 4; i++) {
    set.add(`v${i}`);
}

console.log(set);

// Set { 'v1', 'v2', 'v3' }
```



## 元素检测 has()

使用has()方法可检测当前Set容器中是否存在某个元素项，返回布尔值。

- 注意，如果元素是引用类型则必须要使用原本元素对象的引用才行，单纯的形式相似总会返回false

```
"use strict";

let ary = [1, 2, 3];

let set = new Set([
    ary,
    "v2",
    "v3",
]);

// 同一引用，返回true
console.log(set.has(ary));

// 非同一引用，返回false
console.log(set.has([1, 2, 3]));

// true
// false
```



## 删除元素 delete()

使用delete()方法根据键来删除Set容器中的某个元素项。

若删除成功则返回true，否则返回false：

```
"use strict";

let set = new Set([
    "v1",
    "v2",
    "v3",
]);

console.log(set.delete("v8"));
console.log(set.delete("v1"));

// false
// true
```



## 清空容器 clear()

使用clear()方法可清空Set容器：

```
"use strict";

let set = new Set([
    "v1",
    "v2",
    "v3",
]);

set.clear();

console.log(set);

// Set {}
```



## 数组转换

可使用…语法或者Array.form()将Set容器转换为数组。

我们可以利用Set的去重特性来剔除掉数组中的重复数据项，如下示例：

```
"use strict";

let ary = [1, 1, 2, 2, 3, 3];

ary = Array.from(new Set(ary));

console.log(ary);

// [ 1, 2, 3 ]
```



## 交差并集

通过集合和数组的转换，可实现多集合之间求交差并集操作。

1）交集，求出2个集合之间共有的部分：

```
"use strict";

let set1 = new Set([1, 2, 3]);
let set2 = new Set([2, 3, 4]);

let intersection = new Set(
    Array.from(set1).filter((item, index, array) => {
        return set2.has(item);
    })
)

console.log(intersection);

// Set { 2, 3 }
```

，2）差集，求出一个集合独有的部分：

```
"use strict";

let set1 = new Set([1, 2, 3]);
let set2 = new Set([2, 3, 4]);

let difference = new Set(
    Array.from(set1).filter((item, index, array) => {
        return !set2.has(item);
    })
)

console.log(difference);

// Set { 1 }
```

3）并集，将多个集合合并成一个集合，共有元素只取一次：

```
"use strict";

let set1 = new Set([1, 2, 3]);
let set2 = new Set([2, 3, 4]);

let union = new Set([...set1, ...set2]);

console.log(union);

// Set { 1, 2, 3, 4 }
```





# 遍历操作

## 迭代器方法

使用以下3个方法都可创建出Set迭代器。

- kes()
- values()
- entries()

示例如下：

```
"use strict";

let set = new Set([
    "v1",
    "v2",
    "v3",
]);

console.log(set.keys());
console.log(set.values());
console.log(set.entries());


// SetIterator {"v1", "v2", "v3"}
// SetIterator {"v1", "v2", "v3"}
// SetIterator {"v1" => "v1", "v2" => "v2", "v3" => "v3"}
```



## for/of

由于Set本身属于可迭代对象，所以你可以直接使用for/of对其进行遍历操作。

```
"use strict";

let set = new Set([
    "v1",
    "v2",
    "v3",
]);

for(let v of set){
    console.log(v);
}

// v1
// v2
// v3
```



## forEach()

使用forEach()方法也可对Set容器进行遍历操作。

它与Array的forEach()方法使用基本相同。

```
"use strict";

let set = new Set([
    "v1",
    "v2",
    "v3",
]);

set.forEach((item, _, set) => {
    console.log(item);
})

// v1
// v2
// v3
```



# WeakSet

## 基本介绍

WeakSet和Set非常相似，它也具有去重的特性。

但它和Set也有一些不同之处，如下所示：

- WeakSet中的元素项必须是对象（引用类型）
- WeakSet中的元素是无序的
- WeakSet中的元素是弱引用的，也就是说WeakSet中的元素并不会增加源对象的引用计数
- 在GC机制运行时，如果源对象在其他地方被删除，WeakSet中也会删除掉该元素
- 因为WeakSet的元素是弱引用的，故WeakSet不包含keys()、values()、entries()等方法和size属性，同时WeakSet是不可被迭代的



## 声明定义

使用WeakSet()时传入一个数组即可声明出一个WeakSet容器：

```
"use strict";

let ary = new Array([1, 2, 3]);
let map = new Map([["k1", "v1"]]);
let obj = new Object({"k1" : "v1"});

let wSet = new WeakSet([
    ary,
    map,
    obj,
]);

console.log(wSet instanceof WeakSet);
console.log(wSet);

// true
// WeakSet {Map(1), {…}, Array(1)}
```



## 常用操作

WeakMap容器仅包含3个可用的方法：

- add()：新增一个元素
- delete()：删除一个元素
- has()：判断某个元素是否存在于WeakSet容器中



## 弱引用

对于GC机制的引用计数法来说，存在于WeakSet中的元素并不会为其增加引用计数，因此这种特性也被称为弱引用。

当WeakSet中元素在外部的引用计数为0后，WeakSet内部中该元素也会被清除，如下所示：

```
"use strict";

let ary = new Array([1, 2, 3]);
let map = new Map([["k1", "v1"]]);
let obj = new Object({"k1" : "v1"});

let wSet = new WeakSet([
    ary,
    map,
    obj,
]);

console.log(wSet);

// 引用计数归0
ary = undefined;

// 等待GC机制运行，3s后打印结果
setTimeout(() => {
    console.log(wSet);
}, 3000);

// WeakSet {Map(1), {…}, Array(1)}
// WeakSet {Map(1), {…}}
```

