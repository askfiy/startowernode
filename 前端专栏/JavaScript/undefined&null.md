# 两者关系

在ECMA中认为null与undefined是相等的。

它们均代表没有、未定义、未初始的意思。

它们等同于Go语言的nil，Python的None。

但是2者之间还是有一些语义上的区别的。



# undefined

undefined常用于代表未定义。

1）定义好变量，未进行赋值，此时如果使用该变量得到的结果是undefined：

```
"use strict";

let userName;
console.log(userName);

// undefined
```

2）函数无返回值：

```
"use strict";

function demo(){}

let result = demo();
console.log(result);

// undefined
```



# null

null代表未引用任何对象，常在初始化变量中使用。

某些情况下你可能还没想好这个变量存什么值但确定它后续能存值的时候下可用null进行变量初始化。

- null本身是属于object，也就是引用类型

```
"use strict";

let userName = null;
let userAge = null;

// 请求后端获得数据

userName = "Jack";
userAge = 18;
```

