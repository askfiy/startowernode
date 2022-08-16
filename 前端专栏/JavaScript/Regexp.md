# RegEXP

正则表达式其本身就是一种小型的，高度专业化的编程语言，能够非常方便的对字符串进行处理。

正则语法在各个语言中都是通用的，所以掌握它显得十分有必要。



# 创建正则

## 字面量创建

使用//包裹的字面量创建方式是推荐的作法，但它不能在其中使用变量：

```
"use strict";

let str = "hello,world";

let reg = /^h\w+,\w+$/; // 创建正则对象

console.log(reg.test(str));

// true
```

可以使用eval()将字符串转换为Js语法来实现将变量解析到正则表达式中，但是比较麻烦：

```
"use strict";

let str = "hello,world";

let a = "hello";

console.log(eval(`/${a}/`).test(str));  // 验证字符串中是否有hello

// true
```



## 实例化创建

使用构造函数创建对象：

```
"use strict";

let str = "hello,world";

let reg = new RegExp(/^h\w+,\w+$/)

console.log(reg.test(str));

// true
```

当正则需要动态创建时使用这种方式将变量作为匹配规则：

```
"use strict";

let str = "hello,world";

let a = "hello";

let reg = new RegExp(a);

console.log(reg.test(str));

// true
```





# 基础字符

## . 通配符

代表匹配除\n外的任意字符。如果想匹配\n可使用s模式：

```
"use strict";

let str = "hello,world\nJavaScript-RegExp";

// 匹配十个除开\n外的任意字符
let reg = /........../g;

console.log(str.match(reg));

// (2) ["hello,worl", "JavaScript"]
```



## ^ 开始符

^开始符会匹配以特定字符开始的字符串，在检测时只会检测开头第一个字符并立即返回结果：

```
"use strict";

let str = "hello,world\nJavaScript-RegExp";

// 必须以h开始，后面必须是ello,  然后匹配五个除开\n的任意字符。
let reg = /^hello,...../g;

console.log(str.match(reg));

// ["hello,world"]
```



## $ 结束符

$结束符会匹配以特定字符结尾的字符串，在检测时只会检测结尾最后一个字符并立即返回结果：

```
"use strict";

let str = "hello,world\nJavaScript-RegExp";

// 必须以p结束，前面两个字符必须是E与x
let reg = /Exp$/g;

console.log(str.match(reg));

// ["Exp"]
```



# 重复字符

## \* 可有重复

\*代表可以取0-∞位\*号前面的字符（默认贪婪取值，可通过?取消贪婪模式）：

```
"use strict";

let str_1 = "hello,world\nJavaScript-RegExp";

let str_2 = "h\np";

// 必须是以h开头，可以有0个也可以有多个除\n外的任意字符，紧接\n，可以有0个也可以有多个除\n外的任意字符，必须以p结尾。
let reg = /^h.*\n.*p$/g;

console.log(str_1.match(reg));
console.log(str_2.match(reg));
// 上面条件只有三个是必须的 h开头，p结束，中间必须有换行\n。所以str_2也能匹配出来

// ["hello,world↵JavaScript-RegExp"]
// ["h↵p"]
```



## + 必有重复

+代表可以取1-∞位+号前面的字符（默认贪婪取值，可通过?取消贪婪模式）:

```
"use strict";

let str_1 = "hello,world\nJavaScript-RegExp";

let str_2 = "h\np";

// 必须是以h开头，可以有0个也可以有多个除\n外的任意字符，紧接\n，可以有0个也可以有多个除\n外的任意字符，必须以p结尾。
let reg = /^h.*\n.*p$/g;
// 上面条件只有三个是必须的 h开头，p结束，中间必须有换行\n。所以str_2也能匹配出来

console.log(str_1.match(reg));
console.log(str_2.match(reg));

// ["hello,world↵JavaScript-RegExp"]
// ["h↵p"]
```



## ? 单一重复

?代表可以取1-∞位?号前面的字符（默认贪婪取值，可通过?取消贪婪模式）：

```
"use strict";

let str_1 = "123456789";

let str_2 = "23456789";

// 可以以1开头，也可以不以，后面是23456789，必须是9结尾
let reg = /^1?23456789$/g;

console.log(str_1.match(reg));
console.log(str_2.match(reg));

// ["123456789"]
// ["23456789"]
```



## {n,m} 区间重复

{n,m}代表可以取到n-m位{n,m}前面的字符（默认贪婪取值，可通过?取消贪婪模式）:

```
"use strict";

let str_1 = "1111111";

// 可以是3个1，也可以是4个1
let reg = /1{3,4}/g;

console.log(str_1.match(reg));

// ["1111", "111"]
```



## {n} 精确重复

{n}代表可以精确取到n位{n}前面的字符：

```
"use strict";

let str_1 = "1111111";

// 3个1
let reg = /1{3}/g;

console.log(str_1.match(reg));

// ["111", "111"]
```





## 取消贪婪

默认的*，+，?，{n,m}都是贪婪取值。

即有多个就取多个，没有多个才少取。

在它们后面加上?即可取消贪婪匹配，如下示例：

```
"use strict";

let str_1 = "1111111";

let reg_1 = /1*?/g;  // 0个或者多个，取消贪婪后不取

let reg_2 = /1+?/g;  // 1个或者多个，取消贪婪后取1个

let reg_3 = /1??/g;  // 0个或者1个，取消贪婪后不取

let reg_4 = /1{3,4}?/g;  // 3个或者4个，取消贪婪后取3个

console.log(str_1.match(reg_1));  // ["", "", "", "", "", "", "", ""]

console.log(str_1.match(reg_2));  // ["1", "1", "1", "1", "1", "1", "1"]

console.log(str_1.match(reg_3));  // ["", "", "", "", "", "", "", ""]

console.log(str_1.match(reg_4));  // ["111", "111"]
```





# 字符集

## [] 字符集

在[]中多个字符仅能匹配到1个，并且字符集中所有符号失去特殊意义，仅有- ^ \ 这3个符号在字符集中具有特殊意义：

```
"use strict";

let str_1 = "1111111";
let str_2 = "2222222";

// 1或者2，取多个
let reg_1 = /[12]+/g;

console.log(str_1.match(reg_1));
console.log(str_2.match(reg_1));

// ["1111111"]
// ["2222222"]
```



## [-] 区间符

字符集中的-号代表可以取从多少到多少区间的值，按照ASCII码排序，比如[a-z0-9A-Z]代表这1位取全部的英文字母和数字：

```
"use strict";

let str_1 = "12345abcde";

// 字符可以是a-z或者0-9，取多个
let reg_1 = /[a-z0-9]+/g;

console.log(str_1.match(reg_1));
// ["12345abcde"]
```



## \[^\] 排除符

字符集中的^号代表非的作用，比如\[^0-9\]代表这1位并非是数字：

```
"use strict";

let str_1 = "12345abcde";
let str_2 = "x2345abcde";

// 不能以0-9开头，紧跟着取处\n外的任意字符
let reg_1 = /^[^0-9].+/g;

console.log(str_1.match(reg_1));
console.log(str_2.match(reg_1));

// null
// ["x2345abcde"]
```



## [\\] 转义符

除开在字符集中使用还可以在外部使用，它可以使所有具有特殊意义的字符失去特殊意义。并且还可以为特定的字符指定意义：

```
"use strict";

let str_1 = "\t\n\t\n";

// 取\t或\n
let reg_1 = /[\t\n]+/g;

console.log(str_1.match(reg_1));

// [" ↵   ↵"]
```





# 转义字符

## \ 转义符

转义符如果在字符集外使用，可以为特定的字符赋予特殊的意义，并且还可以让特定的字符失去特殊的意义。

如失去特殊意义：

- .本来是通配符，如果是\.就是普通的.，再也没有任何意义了。

如下示例，匹配一个url：

```
"use strict";

let str = "www.google.com www.biying.com";

let rule = /w{3}\.\w+\.com/g;

console.log(str.match(rule));

// [ 'www.google.com', 'www.biying.com' ]
```

以下是常用加上\后就拥有特殊意义的字符：

| 符号 | 中文名称 | 释义                                                         |
| ---- | -------- | ------------------------------------------------------------ |
| \d   | ...      | 匹配任何十进制数，它相当于在字符集中使用[0-9]                |
| \D   | ...      | 匹配任何非十进制数，它相当于在字符集中使用[^0-9]             |
| \s   | ...      | 匹配任何空白字符，它相当于在字符集中使用[\t\n\r\f\v]         |
| \S   | ...      | 匹配任何非空白字符，它相当于在字符集中使用[^\t\n\r\f\v]      |
| \w   | ...      | 匹配任何字母数字下划线字符,它相当于在字符集中使用[a-z A-Z 0-9] |
| \W   | ...      | 匹配任何非字母数字下划线字符,它相当于在字符集中使用[a-z A-Z 0-9] |
| \b   | ...      | 匹配一个特殊字符边界，比如空格,&.#等(不常用)                 |



## \d和\D

\d可以匹配任何十进制数，它相当于在字符集中使用[0-9]：

```
"use strict";

let reg = /\d+/g;

let str = "123 abc 1a2b3c";

console.log(str.match(reg));

// [ '123', '1', '2', '3' ]
```

\D可以匹配任何非十进制数，它相当于在字符集中使用\[^0-9\]：

```
"use strict";

let reg = /\D+/g;

let str = "123 abc 1a2b3c";

console.log(str.match(reg));

// [ ' abc ', 'a', 'b', 'c' ]
```



## \s和\S

\s可以匹配任何空白字符，它相当于在字符集中使用[\t\n\r\f\v]：

```
"use strict";

let reg = /\s/g;

let str = "123 abc 1a2b3c";

console.log(str.match(reg));

// [ ' ', ' ' ]
```

\S匹配任何非空白字符，它相当于在字符集中使用\[^\t\n\r\f\v\]：

```
"use strict";

let reg = /\S/g;

let str = "123 abc 1a2b3c";

console.log(str.match(reg));

// [ '1', '2', '3', 'a', 'b', 'c', '1', 'a', '2', 'b', '3', 'c' ]
```



## \w和\W

\w可以匹配任何字母数字下划线字符,它相当于在字符集中使用[a-z A-Z 0-9]：

```
"use strict";

let reg = /\w+/g;

let str = "user_name123 \t\n\f";

console.log(str.match(reg));

// [ 'user_name123' ]
```

\W可以匹配任何非字母数字下划线字符,它相当于在字符集中使用[a-z A-Z 0-9]：

```
"use strict";

let reg = /\W+/g;

let str = "user_name123 \t\n\f";

console.log(str.match(reg));

// [ ' \t\n\f' ]
```





# 管道

## | 管道符

|管道符相当于或，注意与字符集里的区别，管道符将前后分为2段，左右看做1个整体，而字符集中的或仅仅代表从众多选项中拿出1个。

```
"use strict";

let str_1 = "abcdefabcccde";

// 取abc或者def
let reg_1 = /abc|def/g;

console.log(str_1.match(reg_1));

// ["abc", "def", "abc"]
```



# 分组

## () 分组符

()分组符将多个元素字符看做一个整体，也就是将它们当做一个元素字符进行匹配。

若整个匹配规则中无子分组，则默认的匹配结果为一组：

```
"use strict";

let str_1 = "asdfghjkl";
let str_2 = "asbfghjkl";

// 必须以asd开头，必须以l结尾
let reg_1 = /^(asd)\w+l$/g;

console.log(str_1.match(reg_1));
console.log(str_2.match(reg_1));

// ["asdfghjkl"]
// null
```



## 分组别名

组别名使用 (?<别名>匹配规则) 形式定义。

在使用match检测时需要取消g模式的全局匹配。

```
"use strict";

let str_1 = "http://www.google.com/";

let reg_1 = /^https?:\/\/w{3}\.(?<name>\w+)\.(com|cn|org|hk)/;


console.log(str_1.match(reg_1));
console.dir(str_1.match(reg_1).groups.name);

// ["http://www.google.com", "google", "com", index: 0, input: "http://www.google.com/", groups: {name: "google"}]
// google
```



## 分组引用

每一个分组都有一个编号，如1,2,3,4,5。

\num在匹配时引用原子组（匹配到的具体内容，注意不是规则），$num指在替换时使用匹配的组数据。

如下所示，将所有的\<h1-6\>标签替换为\<p\>标签，使用编号替换。

```
"use strict";

let str_1 = `
<h1>hello,world</h1>
<h2>hello,JavaScript</h2>
<h3>PHP,no.1!!!</h3>
        `;

// 最后的\1代表引用第一个分组的内容，即grade组  [\s\S]代表匹配任意
let reg = /<(?<grade>h[1-6])>(?<content>[\s\S]*?)\/\1/gi;

// $2 代表拿到第二个分组的内容，即content分组。
let new_str = str_1.replace(reg, `<p>$2</p>`);
console.log(new_str);


// <p>hello,world<</p>>
// <p>hello,JavaScript<</p>>
// <p>PHP,no.1!!!<</p>>
```

　也可以使用别名替换。$<别名>：

```
let new_str = str_1.replace(reg,`<p>$<content></p>`);  // 拿到content分组
console.log(new_str);
```



## 记录取消

通常情况下，我们的分组会记录编号方便后面使用。

如果一个分组的内容编号不想被记录，可使用(?:)进行处理：

```
"use strict";

let str_1 = "123,abc,456";

let reg_1 = /(?:\d+),([a-z]+),(\w+)/;
let reg_2 = /(\d+),([a-z]+),(\w+)/;

console.log(str_1.match(reg_1));  // (3) ["123,abc,456", "abc", "456", index: 0, input: "123,abc,456", groups: undefined]
console.log(str_1.match(reg_2));  // (4) ["123,abc,456", "123", "abc", "456", index: 0, input: "123,abc,456", groups: undefined]

// 可以看到。reg_2的长度多了一位，记录了第一个分组。
```





# 断言匹配

断言匹配虽然都有1个括号，但它们并不是子分组。

故括号中的内容也不会当做结果进行保持，可以将它们理解为匹配时的条件。



## (?=exp)

零宽先行断言匹配后面为exp的内容。

```
"use strict";

let str_1 = "hello,JavaScript";

// 匹配出后面是,JavaScript的内容
let reg_1 = /\w+(?=,JavaScript)/g;

console.log(str_1.match(reg_1));

// ["hello"]
```



## (?<=exp)

零宽后行断言，匹配前面为exp的内容。

```
"use strict";

let str_1 = "hello,JavaScript";

// 匹配出前面是hello,的内容
let reg_1 = /(?<=hello,)\w+/g;

console.log(str_1.match(reg_1));

// ["JavaScript"]
```



## (?!exp)

零宽负向先行断言，匹配后面不能是exp的内容。

```
"use strict";

let str_1 = "_123";
let str_2 = "yun23";
let str_3 = "123";

// 匹配开始不能是下划线的
let reg_1 = /^(?!_)[\w]+/g;

console.log(str_1.match(reg_1));
console.log(str_2.match(reg_1));
console.log(str_3.match(reg_1));

// null
// ["yun23"]
// ["123"]
```



## (?<!exp)

(?<!exp) 零宽负向后行断言匹配前面不能是exp的内容。

```
"use strict";

let str_1 = "abc123   456";

// 前面不能是空格的三位数字
let reg_1 = /(?<!\s+)\d{3}/g;

console.log(str_1.match(reg_1));

// ["123"]
```





# 匹配模式

## 模式介绍

正则表达式在执行时会按他们的默认执行方式进行，但有时候默认的处理方式总不能满足我们的需求，所以可以使用模式修正符更改默认方式。

如下表所示：

| 匹配模式 | 描述                                                |
| -------- | --------------------------------------------------- |
| i        | 不分大小写字母的匹配模式                            |
| g        | 全局搜索所有匹配内容                                |
| m        | 视为多行的匹配模式，以\n作为行分割符                |
| s        | 视为单行的匹配模式，使用.可以匹配所有字符           |
| y        | 从 regexp.lastInde 开始匹配                         |
| u        | 宽字符匹配模式，即支持匹配占4个字符的UTF-16匹配模式 |

注意，JavaScript中regexp的匹配模式可同时应用多个。



## i

i模式匹配时不区分大小写：

```
"use strict";

let str_1 = "HELLO,WORLD";

let reg_1 = /hello,world/i;

console.log(str_1.match(reg_1));

// ["HELLO,WORLD", index: 0, input: "HELLO,WORLD", groups: undefined]
```





## g

g模式不会在第一次匹配成功后就停止，而是继续向后匹配，直到匹配完成：

```
"use strict";

let str_1 = "HELLO,WORLD";

let reg_1 = /./g;

console.log(str_1.match(reg_1));

// (11) ["H", "E", "L", "L", "O", ",", "W", "O", "R", "L", "D"]
```





## m

m模式会将每一行单独匹配，用于将内容视为多行匹配，主要是对 ^和 $的修饰：

```
"use strict";

let str_1 = `

                # 1.HTML5 #
                # 2.CSS #
                # 3.JavaScript #

        `;

// g模式，全局，m模式，多行。每一行必须是多个空格开头，#号结尾。
let reg_1 = /^\s+# \d\.\w+ #$/gm;

console.log(str_1.match(reg_1));

//  ["↵↵                # 1.HTML5 #", "                # 2.CSS #", "
```





## s

s模式将多行以一行匹配，忽略换行符\n。这代表使用.可匹配到任意字符：

```
"use strict";

let str_1 = `123\n456\n789`;

let reg_1 =/.*/gs;

console.log(str_1.match(reg_1));

//  ["123↵456↵789", ""]
```





## u

每个字符都有属性，如L属性表示是字母，P表示标点符号，需要结合 u 模式才有效。

其他属性简写可以访问 [属性的别名](https://www.unicode.org/Public/UCD/latest/ucd/PropertyValueAliases.txt) 网站查看。

如果是一些特殊的生僻字符，那么它的字节宽度可能是4字节或更多，此时就无法正确匹配出来，使用u模式可解决该问题。

```
"use strict";

let str_1 = "𝒳𝒴";

let reg_1 = /𝒳𝒴/g;
let reg_2 = /𝒳𝒴/gu;

console.log(str_1.match(reg_1));  //   结果为乱字符"�"
console.log(str_1.match(reg_1));  //   ["𝒳𝒴"]
```

字符也有Unicode文字系统，属性Script=文字系统。

下面是使用 \p{sc=Han}获取中文字符 han为中文系统，其他语言请查看 [文字语言表](http://www.unicode.org/standard/supported.html)

```
"use strict";

let str_1 = "Mr. Yun 云崖先生 C";

let reg_1 = /\p{sc=Han}+/gu;

console.log(str_1.match(reg_1));

// ["云崖先生"]
```



## lastIndex

RegExp对象lastIndex属性可以返回或者设置正则表达式开始匹配的位置。

- 必须结合g修饰符使用
- 仅对exec()方法有效
- 匹配完成时，lastIndex会被重置为0

如下所示：

```
"use strict";

let str_1 = "Hello,My name is YunYa,YunYa age is 19";

let reg_1 = /YunYa.+/g;

// 从索引23处开始匹配。
reg_1.lastIndex = 23;

console.log(reg_1.exec(str_1));

// ["YunYa age is 19", index: 23, input: "Hello,My name is YunYa,YunYa age is 19",
```



## y

g模式无论成功与否都会一直向后匹配，直到lastIndex等于length。

而y模式也是一直向后匹配，但是只要匹配不成功就会停止。

```
"use strict";

let str_1 = "1211111";

let reg_1 = /1/g;

let reg_2 = /1/y;

console.log(reg_1.exec(str_1));  // ["1", index: 0, input: "1211111", groups: undefined]
console.log(reg_2.exec(str_1));  // ["1", index: 0, input: "1211111", groups: undefined]


console.log(reg_1.lastIndex);  // 1
console.log(reg_2.lastIndex);  // 1


console.log(reg_1.exec(str_1)); // ["1", index: 2, input: "1211111", groups: undefined]
console.log(reg_2.exec(str_1)); // null


console.log(reg_1.lastIndex);  // 3  没找到，g模式继续
console.log(reg_2.lastIndex);  // 0  没找到，y模式归零
```





# 字符串方法

下面例举一些字符串中支持正则匹配的方法。



## search()

search() 方法用于检索字符串中指定的子字符串的索引值，也可以使用正则表达式搜索，返回值为索引位置:

```
"use strict";

let str_1 = "YunYaSir c2323182108@gmail.com";
let reg_1 = /c\d+@\w+.?\w+?\.com/;

console.log(str_1.search(reg_1));

// 9
```



## match()

match()方法可以返回出匹配到的子字符串：

```
"use strict";

let str_1 = "YunYaSir c2323182108@gmail.com";
let reg_1 = /c\d+@\w+.?\w+?\.com/;

console.log(str_1.match(reg_1));

// ["c2323182108@gmail.com", index: 9, input: "YunYaSir c2323182108@gmail.com", groups: undefined]
// 查找到的内容  索引位置  被查找的字符串  分组信息
```

如果使用g模式，返回的内容没那么详细了：

```
"use strict";

let str_1 = "YunYaSir c2323182108@gmail.com";
let reg_1 = /c\d+@\w+.?\w+?\.com/g;

console.log(str_1.match(reg_1));

// ["c2323182108@gmail.com"]
```



## matchAll()

matchAll()和match()使用相同，必须在g模式下使用。

但matchAll()返回一个可迭代对象，而match()返回一个数组：

```
"use strict";

let str_1 = "YunYaSir c2323182108@gmail.com c238923874@qq.com";
let reg_1 = /c\d+@\w+.?\w+?\.com/g;

let resultIter = str_1.matchAll(reg_1);

for (let result of resultIter) {
    console.log(result);
}
```





## split()

用于使用字符串或正则表达式分隔字符串。

如下示例，时间的分隔符不确定是那个，此时可以使用正则匹配。

```
"use strict";

let str_1 = "2020-08/09";

let reg_1 = /[-/]/g;

console.log(str_1.split(reg_1));

// (3) ["2020", "08", "09"]
```



## replace()

replace() 方法不仅可以执行基本字符替换，也可以进行正则替换，下面替换日期连接符：

```
"use strict";

let str_1 = "2020-08/09";

let reg_1 = /[-/]/g;

console.log(str_1.replace(reg_1,"-"));

// 2020-08-09
```



### 字符串替换

替换字符串可以插入下面的特殊变量名：

| 变量 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| $$   | 插入一个 "$"。                                               |
| $&   | 插入匹配的子串。常用                                         |
| `$`` | 插入当前匹配的子串左边的内容。                               |
| $'   | 插入当前匹配的子串右边的内容。                               |
| $n   | 假如第一个参数是 RegExp对象，并且 n 是个小于100的非负整数，那么插入第 n 个括号匹配的字符串。提示：索引是从1开始 |

将百度一下四个文字加上链接：

```
<body>

    <div>百度一下，你就知道</div>

    <script>

    	"use strict";

        let div = document.querySelector("div");
        div.innerHTML = div.innerHTML.replace("百度一下","<a href='http://www.baidu.com'>$&</a>")

    </script>

</body>
```



### 回调函数

replace()支持回调函数操作，用于处理复杂的替换逻辑：

回调函数参数释义：

- match：匹配的子串内容，相当于特殊变量$&
- p1, p2, … ：假如replace()方法的第一个参数是一个 RegExp 对象，则代表第n个括号匹配的字符串。（对应于上述的$1，$2等。）例如，如果是用 /(\a+)(\b+)/ 来进行匹配，p1 就是匹配的 \a+，p2 就是匹配的 \b+
- offset：匹配到的子字符串在原字符串中的偏移量，即子串开始时的索引位置（比如，如果原字符串是 abcd，匹配到的子字符串是 bc，那么这个参数将会是 1）
- string：被匹配的原字符串
- NamedCaptureGroup：命名捕获组匹配的对象

使用回调函数将百度一下四个字加上链接：

```
<body>

    <div>百度一下，你就知道</div>

    <script>

        "use strict";

        let div = document.querySelector("div");
        div.innerHTML = div.innerHTML.replace("百度一下", (match, defaultGroup, source) => `<a href='http://www.baidu.com'>${match}</a>`)

    </script>

</body>
```



# 正则对象方法

下面是RegExp正则对象提供的操作方法

## test()

返回布尔值，是否验证成功。

检测输入的邮箱是否合法：

```
<body>

    <input type="text" name="email" />

    <script>

        "use strict";

        let email = document.querySelector(`[name="email"]`);
        email.addEventListener("keyup", e => {
            console.log(/^\w+@\w+\.\w+$/.test(e.target.value));
        });

    </script>

</body>
```



## exec()

不使用 g 修饰符时与 match方法使用相似，使用 g 修饰符后可以循环调用直到全部匹配完。

- 使用 g修饰符多次操作时使用同一个正则，即把正则定义为变量使用
- 使用 g修饰符最后匹配不到时返回null

示例如下，计算JavaScript共出现了几次：

```
<body>

    <div>JavaScript的事件循环模型与许多其他语言不同的一个非常有趣的特性是，它永不阻塞。JavaScript非常优秀！</div>
    
    <script>

        "use strict";

        let txt = document.querySelector("div").innerHTML;
        let reg = /(?<tag>JavaScript)/gi;
        let num = 0;

        while (reg.exec(txt)) {
            num++;
        }

        console.log(`JavaScript共出现:${num}次。`);

    </script>
    
</body>
```

