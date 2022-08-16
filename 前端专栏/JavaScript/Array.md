# 基本声明

## 数组创建

以下是使用类实例化的形式进行对象声明：

```
"use strict";

let ary = new Array(1, 2, 3);

console.log(ary);

// [ 1, 2, 3 ]
```

也可以选择使用更方便的字面量形式进行对象声明，利用[]对数据项进行包裹，并且使用逗号将数据项之间进行分割：

```
"use strict";

let ary = [1, 2, 3];

console.log(ary);

// [ 1, 2, 3 ]
```

若new Array()时没有传入参数，则生成空数组：

```
"use strict";

let ary = new Array();

console.log(ary);

// []
```



## Array.of()

在使用类实例化的形式进行对象声明时，若数组只想有1个数据项元素，则可通过Array.of()进行创建。

否则创建的是一个长度为填入值的空数组：

```
"use strict";

let ary1 = Array.of(3);  // 创建出只有一个数据项元素3的数组
console.log(ary1);

let ary2 = new Array(3); // 创建出长度为3的空数组
console.log(ary2);

// [ 3 ]
// [ <3 empty items> ]
```



## Array.fill()

若想创建一个数组中具有相同数据项元素的数组，则可通过Array.fill()进行创建。

如下示例，该数组中的数据项元素全部为A：

```
"use strict";

let ary = new Array(5).fill("A");

console.log(ary);

// [ 'A', 'A', 'A', 'A', 'A' ]
```





## 多维数组

当一个数组中嵌套另一个数组时，该数组即为多维数组：

```
"use strict";

let ary = [1, 2, 3, ["A", "B", "C"]];

console.log(ary);

// [ 1, 2, 3, [ 'A', 'B', 'C' ] ]
```



## 不可变数组

数组是引用类型，使用const声明后依旧能改变数组中元素的值：

```
"use strict";

const ary = [1, 2, 3];
ary[0] = "A";

console.log(ary);

// [ 'A', 2, 3 ]
```

你可以使用Object.freeze()方法将数组包裹起来，让该数组变为不可变数组：

```
"use strict";

const ary = [1, 2, 3];
Object.freeze(ary);

ary[0] = "A";

console.log(ary);

// TypeError: Cannot assign to read only property '0' of object '[object Array]'
```



# 类型转换

## 数组转字符串

1）大部分数据类型都可以使用toString()方法将自身转换为字符串：

```
"use strict";

let ary = ["A", "B", "C"];
let strArray = ary.toString();

console.log(strArray);
console.log(typeof strArray);

// A,B,C
// string
```

2）也可以通过String()将数组包裹起来，这样做会将数组转换为字符串：

```
"use strict";

let ary = ["A", "B", "C"];
let strArray = String(ary);

console.log(strArray);
console.log(typeof strArray);

// A,B,C
// string
```

3）join()方法也可以将数组转换为字符串，你需要指定元素之间的链接字符：

```
"use strict";

let ary = ["A", "B", "C"];
let strArray = ary.join("-");

console.log(strArray);
console.log(typeof strArray);

// A-B-C
// string
```



## 类数组转数组

通过Array.form()方法可以将类数组转换为数组。

- 类数组是指具有length属性或者是可迭代的对象

Array.form()具有2个参数：

- arrayLike：待转换为数组的类数组

- callbackfn：回调函数，它会对类数组中的元素挨个挨个的做操作，最终会将return的值加入到新数组中

如下示例，将DOM对象集合的NodeList转换为数组：

```
// HTML代码里6个P标签，内容是 1 2 3 4 5 6

"use strict";

let nodeList = document.querySelectorAll("p");

let nodeInnerTextArray = Array.from(nodeList, (element, index) => {
    return "p -> " + element.innerText
})

console.log(nodeInnerTextArray);

// ["p -> 1", "p -> 2", "p -> 3", "p -> 4 ", "p -> 5 ", "p -> 6"]
```



# 索引使用

## 单元素操作

在String一章节中我们介绍过索引的基本使用。由于String类型是不可变的，所以你只能通过索引进行数据项获取。

而Array是可变类型，所以你可以利用索引对元素进行增删改查。

- JavaScript中的[]索引语法仅支持单元素操作，不能进行切片

1）获取单元素，直接使用ary[idx]即可：

```
"use strict";

let ary = ["A", "B", "C", "D", "E"];

// 获取第一个
console.log(ary[0]);

// 获取最后一个
console.log(ary[ary.length - 1]);

// A
// E
```

2）增加单元素，如果增加单元素的索引位置大于数组已有的最大索引位置，则之前的位置都会用empty进行占位。其实empty就是undefined：

```
"use strict";

let ary = ["A", "B", "C", "D", "E"];
// ary -> current max index = 4
ary[8] = "I";

console.log(ary);
// empty = undefined
console.log(ary[6]);

// [ 'A', 'B', 'C', 'D', 'E', <3 empty items>, 'I' ]
// undefined
```

3）修改单元素，如果要修改的索引位置大于数组已有的最大索引位置，则相当于增加单元素。

```
"use strict";

let ary = ["A", "B", "C", "D", "E"];
ary[0] = "a";

console.log(ary);

// [ 'a', 'B', 'C', 'D', 'E' ]
```

4）删除单元素，使用delete配合[idx]来删除单个元素，被删除的元素位置会用undefined进行占位，即整个数组不会缩容。若要删除的索引位置大于数组已有的最大索引位置，则无效：

```
"use strict";

let ary = ["A", "B", "C", "D", "E"];

// 索引1处变为undefined
delete ary[1];

// 并不会抛出异常
delete ary[8];

console.log(ary);

// [ 'A', <1 empty item>, 'C', 'D', 'E' ]
```



## 实现元素位置调整

通过索引的操作，实现一个元素位置调整函数：

```
"use strict";

function move(array, beforePosition, toPosition) {

    if (beforePosition < 0 || toPosition >= array.length) {
        throw new Error("move position error");
    }
    let beforeValue = array[beforePosition];
    let toValue = array[toPosition];

    array[beforePosition] = toValue;
    array[toPosition] = beforeValue;
}

let ary = ["C", "B", "A", "D"];

move(ary, 0, 2);
console.log(ary);

// [ 'A', 'B', 'C', 'D' ]
```





# ...语法

## 数组合并

…语法可用于拆解一个数组。

如下所示，新数组由2个旧的数组合并而成：

```
"use strict";

let ary1 = ["A", "B", "C"], ary2 = ["D", "E", "F"];

let newArray = [...ary1, ...ary2];

console.log(newArray);

// [ 'A', 'B', 'C', 'D', 'E', 'F' ]
```



## 解构赋值

对一个数组进行解构赋值时，变量名外部必须由[]进行包裹，且变量名与元素一一对应：

```
"use strict";

let ary = ["Jack", "18", "male"];

let [userName, age, gender] = ary;

console.log(userName);
console.log(age);
console.log(gender);

// Jack
// 18
// male
```

若你想一个变量接收多个被解构的元素，可使用...语法接收它们：

```
"use strict";

let ary = ["Jack", "18", "male", "basketball", "football", "volleyball"];

let [userName, age, gender, ...hobby] = ary;

console.log(userName);
console.log(age);
console.log(gender);
console.log(hobby);

// Jack
// 18
// male
// [ 'basketball', 'football', 'volleyball' ]
```

某些变量对你来说没用，可使用_作为匿名变量进行占位，或者直接使用,放弃接收它：

```
"use strict";

let ary = ["Jack", "18", "male", "basketball", "football", "volleyball", "1.92m", "80kg"];

// basketball使用匿名变量接收， football和volleyball直接舍弃
let [userName, age, gender, _, , , height, weight] = ary;

console.log(userName);
console.log(age);
console.log(gender);
console.log(height);
console.log(weight);

// Jack
// 18
// male
// 1.92m
// 80kg
```



## 类数组转数组

使用…语法也可以实现类数组转数组，如下所示：

```
"use strict";

let nodeList = document.querySelectorAll("p");

let nodeInnerTextArray = new Array(...nodeList).map((element, index) => {
    return "p -> " + element.innerText
})

console.log(nodeInnerTextArray);

// ["p -> 1", "p -> 2", "p -> 3", "p -> 4 ", "p -> 5 ", "p -> 6"]
```



# 迭代器方法

## keys()

返回所有元素的索引值：

```
"use strict";

let ary = [ 'A', 'B', 'C', 'D', 'E', 'F' ];
let aryIdx = ary.keys();

while (1){
    let iterItem = aryIdx.next();
    if(iterItem.done === false){
        console.log(iterItem.value);
        continue
    }
    break
}

// 0
// 1
// 2
// 3
// 4
// 5
```



## values()

返回所有元素的值本身：

```
"use strict";

let ary = [ 'A', 'B', 'C', 'D', 'E', 'F' ];
let aryEle = ary.values();

while (1){
    let iterItem = aryEle.next();
    if(iterItem.done === false){
        console.log(iterItem.value);
        continue
    }
    break
}

// A
// B
// C
// D
// E
// F
```



## entries()

以数组形式返回所有元素的索引值与值本身：

```
"use strict";

let ary = ['A', 'B', 'C', 'D', 'E', 'F'];
let aryIE = ary.entries();


while (1) {
    let iterItem = aryIE.next();
    if (iterItem.done === false) {
        console.log(iterItem.value);
        continue
    }
    break
}

// [ 0, 'A' ]
// [ 1, 'B' ]
// [ 2, 'C' ]
// [ 3, 'D' ]
// [ 4, 'E' ]
// [ 5, 'F' ]
```





# 长度获取 length

length 获取数组长度：

```
"use strict";

let ary = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L'];
let len = ary.length;

console.log(len);

// 12
```



# 循环遍历

## for

根据数组长度结合for循环来遍历整个数组：

```
"use strict";

let ary = ['A', 'B', 'C', 'D', 'E', 'F'];

for (let i = 0; i < ary.length; i++) {
    console.log(ary[i]);
}

// A
// B
// C
// D
// E
// F
```



## for/in

for/in迭代时的迭代变量是数组的索引值：

```
"use strict";

let ary = ['A', 'B', 'C', 'D', 'E', 'F'];

for (let idx in ary) {
    console.log(ary[idx]);
}

// A
// B
// C
// D
// E
// F
```



## for/of

for/of迭代时的迭代变量是数组的值本身：

```
"use strict";

let ary = ['A', 'B', 'C', 'D', 'E', 'F'];

for (let item of ary) {
    console.log(item);
}

// A
// B
// C
// D
// E
// F
```

配合迭代器方法entries()和解构赋值，可同时拿到索引和值：

```
"use strict";

let ary = ['A', 'B', 'C', 'D', 'E', 'F'];

for (let [idx, item] of ary.entries()) {
    console.log(idx, item);
}

// 0 'A'
// 1 'B'
// 2 'C'
// 3 'D'
// 4 'E'
// 5 'F'
```



## forEach()

forEach()用于递归的操作元素，并根据操作修改原数组中的元素。

使用该方法时需要绑定一个回调函数，该回调函数没有返回结果。

注意与map()方法的区别，该方法是对数组进行原地操作，不会生成新的数组。

回调函数共有3个参数：

- value：被遍历的元素值本身
- index：被遍历的元素索引值
- array：被遍历的数组本身

如下示例，对原数组每个数据项+100：

```
"use strict";

let ary = [1, 2, 3, 4, 5];

ary.forEach((element, idx, ary)=>{
    ary[idx] += 100;
})

console.log(ary);

// [ 101, 102, 103, 104, 105 ]
```





## 反向遍历

for循环也可用于反向遍历，如下所示：

```
"use strict";

let ary = ['A', 'B', 'C', 'D', 'E', 'F'];

for(let i=ary.length; i>=0; i--){
    console.log(ary[i]);
}

// F
// E
// D
// C
// B
// A
```



# 元素管理

## 压入尾部 push()

将元素追加至数组尾部：

```
"use strict";

let ary = ['A', 'B', 'C', 'D', 'E', 'F'];

ary.push("G", "H", "I", "J", "K")

console.log(ary);

// [ 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K' ]
```



## 弹出尾部 pop()

将数组尾部最后一个元素弹出并返回，它将引起数组的缩容，并不会使用undefined进行占位：

```
"use strict";

let ary = ['A', 'B', 'C', 'D', 'E', 'F'];

let lastElement = ary.pop();
console.log(lastElement);

console.log(ary);

// F
// [ 'A', 'B', 'C', 'D', 'E' ]
```



## 压入头部 unshift()

将元素插入至数组头部：

```
"use strict";

let ary = ['A', 'B', 'C', 'D', 'E', 'F'];

ary.unshift("1", "2", "3", "4", "5")

console.log(ary);

// [ '1', '2', '3', '4', '5', 'A', 'B', 'C', 'D', 'E', 'F' ]
```



## 弹出头部 shift()

将数组头部第一个元素弹出并返回，它将引起数组的缩容，并不会使用undefined进行占位：

```
"use strict";

let ary = ['A', 'B', 'C', 'D', 'E', 'F'];

let firstElement = ary.shift();
console.log(firstElement);

console.log(ary);

// A
// [ 'B', 'C', 'D', 'E', 'F' ]
```





## 切片方法 slice()

使用slice()方法对数组进行切片，顾头不顾尾。

- start：索引开始的位置
- end：索引结束的位置，若不指定该值，则全切

```
"use strict";

let ary = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z'];

console.log(
    // 拿出全部
    ary.slice(0),
    // 拿出A-W
    ary.slice(0, ary.length - 3)
);

```



## 万能方法 splice()

使用splice()方法对原数组中的元素进行添加、删除、替换操作。

1）删除元素，返回值为被删除的元素数组：

- 第一个参数：从哪里开始删除
- 第二个参数：删除几个元素

```
"use strict";

let ary = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L'];

let delElementArray = ary.splice(0, 3); // 从第0个元素开始删，删3个

console.log(delElementArray);
console.log(ary);

// [ 'A', 'B', 'C' ]
// [ 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L' ]
```

2）向末尾添加元素，配合length属性进行操作：

```
"use strict";

let ary = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L'];

// 从最后一个位置删，删除0个，并新增M、N、O元素至数组尾部
ary.splice(ary.length, 0, "M", "N", "O")

console.log(ary);

// [ 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O' ]
```

3）向头部插入元素：

```
"use strict";

let ary = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L'];

// 从第一个位置删，删除0个，并插入1, 2, 3, 4, 5元素至数组尾部
ary.splice(0, 0, "1", "2", "3", "4", "5");

console.log(ary);

// [ '1', '2', '3', '4', '5', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L' ]
```

4）先删除元素，再添加元素到头部：

```
"use strict";

let ary = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L'];

// 从第一个位置删，删3个，并且新增元素到头部
let delElementArray = ary.splice(0, 3, "1", "2", "3", "4", "5"); 

console.log(delElementArray);
console.log(ary);

// [ 'A', 'B', 'C' ]
// [ '1', '2', '3', '4', '5', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L' ]
```



## 数组清空

数组清空的方法有很多。

1）将数组的length属性改为0即可：

```
"use strict";

let ary = ["A", "B", "C"];
ary.length = 0;

console.log(ary);

// []
```

2）改变变量的标识符指向，旧的数组会自动被GC机制清理掉：

```
"use strict";

let ary = ["A", "B", "C"];
ary = []

console.log(ary);

// []
```

3）使用pop()或者shift()方法来清空数组：

```
"use strict";

let ary = ["A", "B", "C"];

while(ary.pop()){};

console.log(ary);

// []
```

4）使用splice()方法来清空数组：

```
"use strict";

let ary = ["A", "B", "C"];

ary.splice(0, ary.length);

console.log(ary);

// []
```



# 查找元素

## 左侧搜索元素 indexOf()

从左侧开始向右查找元素，若元素存在则返回该元素首次出现的索引位置，若元素不存在则返回-1，第2参数为从哪个位置开始找。

- searchElement：要被搜索的元素
- fromIndex：从哪里开始搜索

```
"use strict";

let ary = ["A", "B", "C"];


console.log(
    // 能找到，返回元素的索引位置
    ary.indexOf("B"),
    // 找不到，返回-1
    ary.indexOf("Z")
);

// 1
// -1
```



## 右侧搜索元素 lastIndexOf()

从右侧开始向右查找元素，若元素存在则返回该元素首次出现的索引位置，若元素不存在则返回-1，第2参数为从哪个位置开始找。

- searchElement：要被搜索的元素
- fromIndex：从哪里开始搜索

```
"use strict";

let ary = ["A", "B", "C"];


console.log(
    // 能找到，返回元素的索引位置
    ary.lastIndexOf("B"),
    // 找不到，返回-1
    ary.lastIndexOf("Z")
);

// 1
// -1
```



## 是否包含元素 includes()

查看数组中是否包含某个元素，若该元素存在则返回true，若该元素不存在则返回false，可指定查找位置。

- searchElement：要被搜索的元素
- fromIndex：从哪里开始搜索

```
"use strict";

let ary = ["A", "B", "C"];


console.log(
    // 能找到，返回true
    ary.includes("B"),
    // 找不到，返回false
    ary.includes("Z")
);

// true
// false
```



## 元素查找 find()

查看数组中是否包含某个元素，若该元素存在则返回该元素，若该元素不存在则返回undefined。

- predicate：回调函数，当该函数的return结果为true时，将返回被遍历的元素值本身
- thisArg：如果提供，它将用作每次调用predicate的this值，如果未提供，则使用undefined

find()常用于查找引用类型的数据，比如我们要查找多维数组时，includes()是查找不到的，此时就可以使用find()进行查找：

```
"use strict";

let ary = [1, 2, 3, ["A", "B", "C"]];

let result = ary.find((element, index, array) => {
    return ["A", "B", "C"].toString() == element
})

console.log(result);

// [ 'A', 'B', 'C' ]
```



## 索引查找 findIndex()

查看数组中是否包含某个元素，若该元素存在则返回该元素在数组中的索引，若该元素不存在则返回undefined。

- predicate：回调函数，当该函数的return结果为true时，将返回被遍历的元素在数组中的索引
- thisArg：如果提供，它将用作每次调用predicate的this值，如果未提供，则使用undefined

findIndex()常用于查找引用类型的数据，比如我们要查找多维数组时，includes()是查找不到的，此时就可以使用find()进行查找：

```
"use strict";

let ary = [1, 2, 3, ["A", "B", "C"]];

let result = ary.findIndex((element, index, array) => {
    return ["A", "B", "C"].toString() == element
})

console.log(result);

// 3
```



## find原理

以下是find()和findIndex()的原理：

```
"use strict";

function find(array, callbackfn) {
    for (let item of array) {
        // 如果被可迭代对象返回true，则返回被遍历的值
        if (callbackfn(item) === true) {
            return item
        }
    }
    return undefined
}

let ary = [1, 2, 3, ["A", "B", "C"]];

let result = find(ary, element => {
    return ["A", "B", "C"].toString() == element
})

console.log(result);

// [ 'A', 'B', 'C' ]
```

我们可以将该函数稍作修改后添加进Array的原型中，这样所有的Array实例都能调用该方法了：

```
"use strict";

Array.prototype.findItem = function (callbackfn) {
    // this = 调用该方法的数组
    for (let item of this) {
        if (callbackfn(item) === true) {
            return item
        }
    }
    return undefined
}

let ary = [1, 2, 3, ["A", "B", "C"]];

let result = ary.findItem(element => {
    return ["A", "B", "C"].toString() == element
})

console.log(result);

// [ 'A', 'B', 'C' ]
```



# 反转排序

## 数组翻转 reverse()

reverse()方法用于将数组进行翻转，它并不会生成新的数组，而是在原数组基础上做操作：

```
"use strict";

let ary = ["A", "B", "C", "D", "E", "F"];

ary.reverse();

console.log(ary);

// [ 'F', 'E', 'D', 'C', 'B', 'A' ]
```



## 元素排序 sort()

sort()方法可对数组中的元素项进行排序，默认按照从小到大的升序排列进行排序，它并不会生成新的数组，而是在原数组基础上做操作：

```
"use strict";

let ary = [3, 2, 5, 1, 4];

ary.sort();

console.log(ary);

// [ 1, 2, 3, 4, 5 ]
```

配合reverse()方法，实现倒序排列：

```
"use strict";

let ary = [3, 2, 5, 1, 4];

ary.sort().reverse();

console.log(ary);

// [ 5, 4, 3, 2, 1 ]
```

为sort()方法指定回调函数，实现自定义规则排序。

- 回调函数接收2个参数，a和b
- 若回调函数返回的是负数，则a排在b之前
- 若回调函数返回的是正数，则a排在b之后
- 若回调函数返回的是0，则代表a和b相等，不进行位置调整

如下所示：

```
"use strict";

let ary = [3, 2, 5, 1, 4];

ary.sort((a, b)=>{
    // a - b 升序排列
    // b - a 降序排列
    return b - a;
});

console.log(ary);

// [ 5, 4, 3, 2, 1 ]
```



## 实例演示

按照薪资待遇从大到小进行排序：

```
"use strict";

let ary = [
    { name: "Jack", salary: 2000 },
    { name: "Tom", salary: 5000 },
    { name: "Mary", salary: 3000 },
    { name: "Ken", salary: 8000 },
];

ary.sort((a, b) => {
    // a - b 升序排列
    // b - a 降序排列
    return b.salary - a.salary;
});

console.log(ary);

//  [
//   { name: 'Ken', salary: 8000 },
//   { name: 'Tom', salary: 5000 },
//   { name: 'Mary', salary: 3000 },
//   { name: 'Jack', salary: 2000 }
//  ]
```



## 排序原理

以下是排序原理实现：

```
"use strict";

Array.prototype.sortItem = function (callback) {
    // this = 调用该方法的数组
    for (let a in this) {
        for (let b in this) {
            // 如果调用结果小于0，则意味着a小于b，需要调整位置
            if (callback(this[a], this[b]) < 0) {
                let temp = this[a];
                this[a] = this[b];
                this[b] = temp;
            }
        }
    }
    return this;
}

let ary = [3, 2, 5, 1, 4];

ary.sortItem((a, b) => {
    return a - b
})

console.log(ary);

// [ 5, 4, 3, 2, 1 ]
```



# 合并拆分

## 数组合并为字符串 join()

使用join()方法，将数组合并为字符串，它会产生一个新的字符串并返回：

你需要指定元素之间的链接字符：

```
"use strict";

let ary = ["www", "google", "com"];
let str = ary.join(".");

console.log(str);

// www.google.com
```



## 字符串拆分为数组 split()

使用split()方法，将字符串合并为数组，它会产生一个新的数组并返回。

你需要指定依据那个字符进行拆分，并且指定拆分次数。

具体示例参见String一章：

```
"use strict";

let str = "www.google.com";
let ary = str.split(".");

console.log(ary);

// [ 'www', 'google', 'com' ]
```



## 链接多个数组 concat()

使用concat()方法，将多个数组进行链接合并，它会产生一个新的数组并返回。

```
"use strict";

let ary = [1, 2, 3];
let newAry = ary.concat([4, 5, 6], [7, 8, 9]);
console.log(newAry);

// [ 1, 2, 3, 4, 5, 6, 7, 8, 9 ]
```





# 扩展方法

## 全部为真 every()

every()用于递归的检测元素，所有检测为true结果才为true，若有一个检测结果为false则直接返回false。

使用该方法时需要要绑定一个回调函数，该回调函数必须返回true或者false。

回调函数共有3个参数：

- value：被遍历的元素值本身
- index：被遍历的元素索引值
- array：被遍历的数组本身

如下示例，检测班级中所有JavaScript成绩都及格的同学：

```
"use strict";

let ary = [
    {name : "Tom", js : 98},
    {name : "Jack", js : 67},
    {name : "Tom", js : 58},
];

let result = ary.every((item, idx, array)=>{
    return item.js >= 60
})

console.log(result);

// false
```

内部实现原理：

```
"use strict";

Array.prototype.everyItem = function (callbackfn) {
    // this = 调用该方法的数组
    for (let [index, item] of this.entries()) {
        if (!callbackfn(item, index, this)) {
            return false
        }
    }
    return true
}

let ary = [
    { name: "Tom", js: 98 },
    { name: "Jack", js: 67 },
    { name: "Tom", js: 59 },
];

let result = ary.everyItem((item, idx, array) => {
    return item.js >= 60
})

console.log(result);

// false
```



## 一个为真 some()

some()用于递归的检测元素，所有检测为false结果才为false，若有一个检测结果为true则直接返回true。

使用该方法时需要绑定一个回调函数，该回调函数必须返回true或者false。

回调函数共有3个参数：

- value：被遍历的元素值本身
- index：被遍历的元素索引值
- array：被遍历的数组本身

如下示例，检测用户输入的内容是否包含敏感词汇：

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
    <input type="text" name="username" placeholder="请输入您的昵称">
    
    <script>
        "use strict";
        const detect = ["蛤蟆", "维尼熊", "跳跳虎"];
        let inputElement = document.querySelector("input[name=username]");
        
        inputElement.addEventListener("keyup", function (event) {
            let content = event.target.value;
            let result = detect.some((item, index, array) => {
                return detect.indexOf(content) >= 0;
            })
            if (result) {
                inputElement.value = "";
                inputElement.placeholder = "请不要输入敏感词汇";
            }
        })
    </script>
</body>

</html>
```

内部实现原理：

```
"use strict";

Array.prototype.someItem = function (callbackfn) {
    // this = 调用该方法的数组
    for (let [index, item] of this.entries()) {
        if (callbackfn(item, index, this)) {
            return true
        }
    }
    return false
}

let ary = [
    { name: "Tom", js: 98 },
    { name: "Jack", js: 67 },
    { name: "Tom", js: 59 },
];

let result = ary.someItem((item, idx, array) => {
    return item.js < 60
})

console.log(result);

// true
```



## 元素过滤 filter()

filter()用于递归的检测元素，所有检测结果为true的元素添加至新的数组中，检测结果为false的元素丢弃。

当检测完毕后会返回新的数组。

使用该方法时需要绑定一个回调函数，该回调函数必须返回true或者false。

回调函数共有3个参数：

- value：被遍历的元素值本身
- index：被遍历的元素索引值
- array：被遍历的数组本身

如下示例，筛选出班级中所有JavaScript成绩都及格的同学：

```
"use strict";

let ary = [
    { name: "Tom", js: 98 },
    { name: "Jack", js: 67 },
    { name: "Tom", js: 59 },
];

let result = ary.filter((item, idx, array) => {
    return item.js >= 60
})

console.log(result);

// [ { name: 'Tom', js: 98 }, { name: 'Jack', js: 67 } ]
```

内部实现原理：

```
"use strict";

Array.prototype.filterItem = function (callbackfn) {
    // this = 调用该方法的数组
    let newArray = []
    for (let [index, item] of this.entries()) {
        if (callbackfn(item, index, this)) {
            newArray.push(item);
        }
    }
    return newArray
}

let ary = [
    { name: "Tom", js: 98 },
    { name: "Jack", js: 67 },
    { name: "Tom", js: 59 },
];

let result = ary.filterItem((item, idx, array) => {
    return item.js >= 60
})

console.log(result);

// [ { name: 'Tom', js: 98 }, { name: 'Jack', js: 67 } ]
```



## 遍历操作 map()

map()用于递归的操作元素，并将操作完成后的元素添加进新数组中，当操作完毕后会返回新的数组。

使用该方法时需要绑定一个回调函数，该回调函数可有返回结果，也可以没有返回结果，若没返回结果新数组则为空。

注意与forEach()方法的区别，该方法不会对原数组造成任何改变，只会生成新的数组。

回调函数共有3个参数：

- value：被遍历的元素值本身
- index：被遍历的元素索引值
- array：被遍历的数组本身

如下示例，对原数组每个数据项+100，生成新的数组并返回，原数组并不受影响：

```
"use strict";

let ary = [1, 2, 3, 4, 5];

let result = ary.map((element, idx, ary) => {
    return element += 100;
})

console.log(result);
console.log(ary);

// [ 101, 102, 103, 104, 105 ]
// [ 1, 2, 3, 4, 5 ]
```

内部实现原理：

```
"use strict";

Array.prototype.mapItem = function (callbackfn) {
    // this = 调用该方法的数组
    let newArray = []
    for (let [index, item] of this.entries()) {
        newArray.push(callbackfn(item, index, this));
    }
    return newArray
}



let ary = [1, 2, 3, 4, 5];

let result = ary.mapItem((element, idx, ary) => {
    return element += 100;
})

console.log(result);
console.log(ary);

// [ 101, 102, 103, 104, 105 ]
// [ 1, 2, 3, 4, 5 ]
```



## 多元素操作 reduce()

reduce()方法有1个初始值，你可以指定该初始值或者不指定。

每次对数组进行遍历后，都会使用回调函数处理该遍历项，并且将该遍历项与初始值进行操作。

常用于累积运算，如累加，累乘等。

除此之外，还有个reduceRight()方法，它是从右向左的遍历，而reduce()方法是从左向右的遍历。

回调函数的参数：

- prev：上次调用回调函数返回的结果
- cur：被遍历的元素值本身
- index：被遍历的元素索引值
- array：被遍历的数组本身

下面代码示例。

1）统计元素在数组中出现的次数：

```
"use strict";

function getCount(ary, item) {
    return ary.reduce((prev, cur, index, array) => {
        return prev += (cur === item ? 1 : 0)
    }, 0)
}

let ary = [1, 2, 3, 4, 4, 5, 6];

console.log(getCount(ary, 4));

// 2
```

2）返回数组中最大元素：

```
"use strict";

function getMax(ary) {
    return ary.reduce((prev, cur, index, array) => {
        return prev > cur ? prev : cur
    })
}

let ary = [1, 2, 3, 4, 4, 5, 6];

console.log(getMax(ary));

// 6
```

3）进行累加操作：

```
"use strict";

function getMax(ary) {
    return ary.reduce((prev, cur, index, array) => {
        return prev > cur ? prev : cur
    })
}

// 创建一个数组，长度是100全是空的，然后获取其下索引值
// 再通过 ...语法展开到新的数组中，这个新数组的元素就
// 变为了0-99
let ary = [...Array(100).keys()];
let result = ary.reduce((prev, cur, index, array) => {
    return prev + cur
})

console.log(result);
// 4950
```

4）在100的基础上进行累加操作：

```
"use strict";

function getMax(ary) {
    return ary.reduce((prev, cur, index, array) => {
        return prev > cur ? prev : cur
    })
}

let ary = [...Array(11).keys()];
let result = ary.reduce((prev, cur, index, array) => {
    return prev + cur
}, 100)

console.log(result);
// 155
```

内部实现原理：

```
"use strict";

Array.prototype.reduceMethod = function (callbackfn, initialValue = undefined) {
    // this = 调用该方法的数组
    for (let [index, item] of this.entries()) {
        initialValue = callbackfn(initialValue, item, index, this);
    }
    return initialValue
}



let ary = [...Array(11).keys()];
let result = ary.reduceMethod((prev, cur, index, array) => {
    return prev + cur
}, 100)

console.log(result)

// 155
```

