# 基本声明

字符串是JavaScript中非常常见的一种数据类型。

以下是使用类实例化的形式进行对象声明，注意typeof会返回object：

```
"use strict";

let str = new String("hello world");
console.log(`value : ${str}\ntype : ${typeof str}`);

// value : hello world
// type : object
```

也可以选择使用更方便的字面量形式进行对象声明，用英文状态下的单引号、双引号将数据项进行包裹即可：

```
"use strict";

let str = "hello world";
console.log(`value : ${str}\ntype : ${typeof str}`);

// value : hello world
// type : string
```

若new String()时没有传入参数，则生成空字符串：

```
"use strict";

let str = new String();
console.log(`value : ${str}\ntype : ${typeof str}`);

// value :
// type : object
```



# 模板字面量

## ${} \`

JavaScript中没有format方法进行字符串格式化，但是可以通过模板字面量来达到相同的功能。

语法如下，使用字符串2侧使用`进行包裹，并使用${变量 or 函数 or 表达式}来进行格式化。

```
"use strict";

let userName = prompt("please enter your name:").trim();
console.log(`hello ${userName}, welcome home`);

// 用户输入: Jack
// 打印结果: hello Jack, welcome home
```

此外，模板字面量中可以直接使用换行操作，这里不再举例。



## 模板拆解

模板拆解是指定义一个函数，该函数可以提取出模板字符串中的普通字符串与格式化变量。

代码示例：

```
"use strict";

let userName = "Jack";
let userAge = 18;

function func(s, ...f) {
    // s:普通字符串
    // f:格式化变量
    console.log(s);  // (3) ["name:", ",age:", "", raw: Array(3)]
    console.log(f);  // (2) ["Jack", 18]
}

func`name:${userName},age:${userAge}`;
```



# 特殊的\

在普通的声明字符串中，\后面一般都会跟上一个特殊字符。

该字符具有特殊的意义，如\n代表换行，\t代表制表符等，这种具有特殊意义的\char组合被称为转义字符。

常见的转义字符如下表所示：

| 转义字符 | 意义                                | ASCII码值（十进制） |
| -------- | ----------------------------------- | ------------------- |
| \a       | 响铃(BEL)                           | 007                 |
| \b       | 退格(BS) ，将当前位置移到前一列     | 008                 |
| \f       | 换页(FF)，将当前位置移到下页开头    | 012                 |
| \n       | 换行(LF) ，将当前位置移到下一行开头 | 010                 |
| \r       | 回车(CR) ，将当前位置移到本行开头   | 013                 |
| \t       | 水平制表(HT) （跳到下一个TAB位置）  | 009                 |
| \v       | 垂直制表(VT)                        | 011                 |
| \        | 代表一个反斜线字符'''               | 092                 |
| \'       | 代表一个单引号（撇号）字符          | 039                 |
| \"       | 代表一个双引号字符                  | 034                 |
| ?        | 代表一个问号                        | 063                 |
| \0       | 空字符(NUL)                         | 000                 |
| \ddd     | 1到3位八进制数所代表的任意字符      | 三位八进制          |
| \xhh     | 十六进制所代表的任意字符            | 十六进制            |



# 类型转换

string支持与boolean、number进行转换，这是最常见的操作：

```
"use strict";

let str = "string";

console.log(
    !!str,
    Number(str)
);

// true NaN
```

string也支持转换为array：

```
"use strict";

let str = "string";

console.log(
    Array(str),
);

// ["string"]
```



# 四则运算

四则运算中用的比较多的就是string与string之间的运算以及string与number之间的运算。

1）string与string的四则运算只有+得到的结果是string，其他得到的结果均为number或者NaN：

```
"use strict";

console.log(
    "3" + "3",
    "3" - "3",
    "3" * "3",
    "3" / "3",
);

// 33 0 9 1
// string number number number
```

2）string与number的四则运算只有+得到的结果是string，其他得到的结果均为number或者NaN：

```
"use strict";

console.log(
    "3" + 3,
    "3" - 3,
    "3" * 3,
    "3" / 3,
);

// 33 0 9 1
// string number number number
```





# 索引切片

string底层是以一种连续的顺序结构存储数据项，故可以使用索引（index）单个数据项进行获取等操作。

- JavaScript中只有正向索引，没有负向索引
- 正向索引起始值从0开始

如下所示：

```
----------------------------|
| A | B | C | D | E | F | G |
----------------------------|
| 0 | 1 | 2 | 3 | 4 | 5 | 6 |
----------------------------|
```

如下所示，取出第3个元素：

```
"use strict";

let v = "ABCDEFG";
console.log(v[3]);

// D
```

如果想取出最后一个元素，可以搭配length属性：

```
"use strict";

let v = "ABCDEFG";
let len = v.length;
console.log(v[len - 1]);

// G
```





# 长度获取 length

length 获取字符串长度：

```
"use strict";

let str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
let len = str.length;

console.log(len);

// 26
```



# 大小写转换



## 转全大写 toUpperCase()

toUpperCase()  将字符串转换为全大写：

```
"use strict";

let str = "abcdefghijklmnopqrstuvwxyz";

console.log(str.toUpperCase());

// ABCDEFGHIJKLMNOPQRSTUVWXYZ
```



## 转全小写 toLowerCase()

toLowerCase()  将字符串转换为全小写：

```
"use strict";

let str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

console.log(str.toLowerCase());

// abcdefghijklmnopqrstuvwxyz
```



# 空白移除



## 移除两侧空白 trim()

trim()  移除字符串两侧的空白：

```
"use strict";

let str = "  mid  ";

console.log(str.trim());

// mid
```



## 移除左侧空白 trimLeft()

trimLeft()  移除字符串左侧的空白：

```
"use strict";

let str = "  mid  ";

console.log(str.trimLeft());

// mid  
```



## 移除右侧空白 trimRight()

trimRight()  移除字符串右侧的空白:

```
"use strict";

let str = "  mid  ";

console.log(str.trimRight());

//   mid
```



# 元素获取

## 单字符获取 charAt()

charAt()  通过索引值获取单字符，与[idx]效果相同：

```
"use strict";

let str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

console.log(
    // 取出A
    str.charAt(0),
    str[0],
    // 取出Z
    str.charAt(str.length - 1),
    str[str.length - 1],
    // 取出X
    str.charAt(str.length - 3),
    str[str.length - 3],
);

// A A Z Z X X
```



## 字符转Unicode charCodeAt()

charCodeAt()  通过索引值获取单字符的Unicode编码，如果str本身就是单字符，则不用填入索引值：

```
"use strict";

let str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

console.log(
    // 取出A的字符编码 65
    str.charCodeAt(0),
    // 取出Z的字符编码 90
    str.charCodeAt(str.length - 1),
    // 取出X的字符编码 88
    str.charCodeAt(str.length - 3),
);

// 单字符不用填索引值
console.log("Z".charCodeAt());

// 65 90 88
// 90
```



## Unicode转字符 fromCharCode()

fromCharCode() 可将Unicode编码转换为字符，通常对应charCodeAt()使用。

可理解为根据ASCII码值将Number转换为String：

```
"use strict";

console.log(String.fromCharCode(65));

// A
```



# 切片操作

## 截取子串 slice()

slice()  字符串按照索引切片，顾头不顾尾。

- start：索引开始的位置
- end：索引结束的位置，若不指定该值，则全切

```
"use strict";

let str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

console.log(
    // 拿出全部
    str.slice(0),
    // 拿出A-W
    str.slice(0, str.length - 3)
);

// ABCDEFGHIJKLMNOPQRSTUVWXYZ
// ABCDEFGHIJKLMNOPQRSTUVW
```



## 截取子串 substr()

获取从指定位置开始并具有指定长度的子字符串。

- form：从哪个位置开始切
- length：切几个，若不指定该值，则全切

```
"use strict";

let str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

console.log(
    // 拿出全部
    str.substr(0),
    // 从B开始切，向后数5-1个
    str.substr(1, 5)
);

// ABCDEFGHIJKLMNOPQRSTUVWXYZ
// BCDEF
```



## 截取子串 substring()

substring() 字符串按照索引切片，顾头不顾尾。

- start：索引开始的位置
- end：索引结束的位置，若不指定该值，则全切

```
"use strict";

let str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

console.log(
    // 拿出全部
    str.substring(0),
    // 拿出A-W，与slice()相同
    str.substring(0, str.length - 3)
);

// ABCDEFGHIJKLMNOPQRSTUVWXYZ
// ABCDEFGHIJKLMNOPQRSTUVW
```



# 查找相关



## 左侧搜索子串 indexOf()

indexOf()  从左侧开始向右查找子串，若子串存在则返回子串开始的索引位置，若子串不存在则返回-1，第2参数为从哪个位置开始找。

- searchString：要被搜索的子串
- position：从哪里开始搜索

```
"use strict";

let str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

console.log(
    // 能找到，返回子串开始的索引位置
    str.indexOf("FGH"),
    // 找不到，返回-1
    str.indexOf("abc")
);

// 5
// -1
```



## 右侧搜索子串 lastIndexOf()

lastIndexOf()  从右侧开始向右查找子串，若子串存在则返回子串开始的索引位置，若子串不存在则返回-1，第2参数为从哪个位置开始找。

- searchString：要被搜索的子串
- position：从哪里开始搜索

```
"use strict";

let str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

console.log(
    // 能找到，返回子串开始的索引位置
    str.lastIndexOf("FGH"),
    // 找不到，返回-1
    str.lastIndexOf("abc")
);

// 5
// -1
```



## 是否包含子串 search()

search()  通过正则表达式搜索子串，若子串存在则返回子串开始的索引位置，若子串不存在则返回-1。

```
"use strict";

let str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

console.log(
    // 能找到，返回子串开始的索引位置
    str.search("Z$"),
    // 找不到，返回-1
    str.search("^B")
);

// 25
// -1
```



## 是否包含子串 includes()

includes()  查看字符串中是否包含子串，若子串存在则返回true，若子串不存在则返回false，可指定查找位置。

- searchString：要被搜索的子串
- position：从哪里开始搜索

```
"use strict";

let str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

console.log(
    // 能找到，返回true
    str.includes("ABC"),
    // 找不到，返回false
    str.includes("abc")
);

// true
// false
```



## 特定子串开头 startsWith()

startsWith()  查看字符串是否以特定子串开头，返回true或者false，可指定开头索引的查找位置。

```
"use strict";

let str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

console.log(
    // 是否已ABC开头
    str.startsWith("ABC"),
    // 索引5处是否已FGH开头
    str.includes("FGH", 5)
);

// true
// true
```



## 特定子串结尾 endsWith()

endsWith()  查看字符串是否以特定子串结尾，返回true或者false，可指定结尾索引的查找位置。

```
"use strict";

let str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

console.log(
    // 是否已XYZ结尾
    str.endsWith("XYZ"),
    // 索引5处是否已FGH结尾
    str.includes("FGH", 5)
);

// true
// true
```



# 替换与重复生成



## 替换子串 replace()

replace()  替换字符串中的子串为新串，默认只替换一次，返回新的字符串。

- searchValue：被替换的子串
- replaceValue：替换的子串

```
"use strict";

let str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

console.log(
    // 将XYZ替换为xyz
    str.replace("XYZ", "xyz"),
);

// ABCDEFGHIJKLMNOPQRSTUVWxyz
```



## 重复生成 repeat()

repeat()  重复生成字符串：

```
"use strict";

let str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

console.log(
    // 将str重复生成3次
    str.repeat(3),
);

// ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZ
```

将电话号码中间4位隐藏：

```
"use strict";

let str = "13899320641";

console.log(
    str.replace(str.slice(3, 7), "*".repeat(4))
);

// 138****0641
```



# 拆分与拼接

## 拆分字符串 split()

split()  拆分字符串为数组。

- separator：根据那个子串进行拆分
- limit：指定数组中存入的拆分结果数量

默认数组会将所有拆分结果都进行存入：

```
"use strict";

let str = "hello, world, hello, JavaScript";

console.log(
    // 以,进行拆分
    str.split(",")
);

// ["hello", " world", " hello", " JavaScript"]
```

指定数组仅存入拆分结果的前2个：

```
"use strict";

let str = "hello, world, hello, JavaScript";

console.log(
    str.split(",", 2)
);

// ["hello", " world"]

```



## 字符串拼接 concat()

concat()  拼接字符串，类似于 string + string 的操作：

```
"use strict";

let str = "hello";

console.log(
    // 相当于 str + , + world
    str.concat(",", "world"),
);

// hello,world
```

