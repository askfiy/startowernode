# DOM

DOM的全称为Document Object Model即为文档对象模型。

DOM支持将HTML文档转换为JavaScript的对象进行操作。



# 节点对象

## 基本介绍

HTML文档中的任何内容在JavaScript中统称为DOM节点对象。

每个节点对象都包含若干操纵方法。以下是节点对象的常用知识：

- 节点对象可分为12种类型
- 根节点对象为document，即文档本身
- 所有的节点对象都是Node的实例
- 常见节点对象有根节点、元素节点、文本节点、注释节点

节点类型与编号如下所示：

| 节点类型         | 类型编号 |
| ---------------- | -------- |
| 元素节点         | 1        |
| 属性节点         | 2        |
| 文本节点         | 3        |
| CDATA节点        | 4        |
| 实体引用名称节点 | 5        |
| 实体名称节点     | 6        |
| 处理指令节点     | 7        |
| 注释节点         | 8        |
| 文档节点         | 9        |
| 文档类型节点     | 10       |
| 文档片段节点     | 11       |
| DTD声明节点      | 12       |

代码示例：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>

    <main id="main">

        <!-- 注释节点 -->
    </main>

    <script>

        "use strict";

        console.log(document.body.nodeType); // 1 body标签，元素节点

        let main = document.getElementById("main");

        console.log(main.attributes[0].nodeType);  // 2 main标签的id属性也是一个节点。属性节点
        console.log(main.firstChild.nodeType);  // 3 main标签下的第一个子标签 即空白行。文本节点
        console.log(main.lastChild.nodeType);  // 3 main标签下的第一个子标签 即空白行。文本节点
        console.log(main.COMMENT_NODE);  // 8 main标签下的注释标签。 注释节点

        console.log(document.nodeType);  // 9 document也是一个节点。文档节点
        console.log(document.childNodes.item(0).nodeType);  // 10  文档类型节点

    </script>

</body>
```



## 节点原型链

对每一种类型的节点对象进行原型链的攀升，可得到如下结论：

| 原型               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| Object             | 根对象                                                       |
| EventTarget        | 提供事件支持                                                 |
| Node               | 提供parentNode等节点关系操作方法的支持                       |
| Element            | 提供getElementsByTagName()、querySelector()等节点查找方法的支持 |
| HTMLElement        | 所有元素的基础类，提供className、nodeName等节点标准属性的支持 |
| HTMLHeadingElement | Head标题元素类                                               |

以下以元素节点&lt;main&gt;标签为例，它的最终原型对象为ElementTarget.proptrpy：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>

    <main id="main">

        <!-- 注释节点 -->
    </main>

    <script>

        "use strict";

        let main = document.getElementById("main");
        console.log(main.__proto__);
        console.log(main.__proto__.__proto__);
        console.log(main.__proto__.__proto__.__proto__);
        console.log(main.__proto__.__proto__.__proto__.__proto__);
        console.log(main.__proto__.__proto__.__proto__.__proto__.__proto__);

    </script>

</body>

</html>
```

如图所示：

![image-20200816184329033](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/1881426-20200818152538125-1692964534.png)

# 节点对象属性

## nodeType

每个节点对象都有nodeType属性，它返回节点类型的编号。

| 节点类型         | 类型编号 |
| ---------------- | -------- |
| 元素节点         | 1        |
| 属性节点         | 2        |
| 文本节点         | 3        |
| CDATA节点        | 4        |
| 实体引用名称节点 | 5        |
| 实体名称节点     | 6        |
| 处理指令节点     | 7        |
| 注释节点         | 8        |
| 文档节点         | 9        |
| 文档类型节点     | 10       |
| 文档片段节点     | 11       |
| DTD声明节点      | 12       |

以下是nodeType属性的基本使用：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>

    <h1></h1>

    <script>

        "use strict";

        console.log(document.querySelector("h1").nodeType);

        // 1

    </script>

</body>

</html>
```

使用对象原型进行检测：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>

    <h1></h1>

    <script>

        "use strict";

        console.log(document.querySelector("h1") instanceof Element);

        // true

    </script>

</body>

</html>
```



## nodeName

nodeName属性可供任何类型的节点对象使用，其作用是获取节点名称：

| 节点类型 | 类型编号 | 节点名称                       |
| -------- | -------- | ------------------------------ |
| 元素节点 | 1        | 元素名称（返回值为全大写形式） |
| 属性节点 | 2        | 属性名称                       |
| 文本节点 | 3        | #text                          |
| 注释节点 | 8        | #comment                       |

表格释义：

- 对类型编号为1的元素节点对象使用nodeName属性，将返回元素名称
- 对类型编号为8的注释节点对象使用nodeName属性，将返回#comment

如下所示：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <h1>hello，world</h1>
    <h2 class="show">标题</h2>
    <!-- 注释 -->
    文本
    <script>

        "use strict";

        let h1 = document.querySelector("h1");
        let h2Attr = document.querySelector("h2").attributes[0];
        let comment = document.body.childNodes[5];
        let text = document.body.childNodes[6];

        // 获取为大写形式

        console.log(h1.nodeName);  // H1
        console.log(h2Attr.nodeName);  // class
        console.log(comment.nodeName);  // #comment
        console.log(text.nodeName);  // #text

        console.log(document.body.nodeName);  // BODY

    </script>
</body>

</html>
```



## tagName

tagName属性只供元素节点对象使用，其作用是获取节点名称：

- tagName属性存在于Elment原型中
- 对元素节点使用tagName或nodeName没有任何差异
- 该属性获取的值都是全大写形式的

代码示例：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <h1>hello，world</h1>
    <h2 class="show">标题</h2>
    <!-- 注释 -->
    文本
    <script>

        "use strict";

        console.log(document.querySelector("h1").tagName);  // H1
        console.log(document.body.tagName); // BODY

    </script>
</body>

</html>
```



## nodeValue

使用nodeValue属性或data属性获取节点值。

它仅针对非元素节点使用，元素节点使用该属性总是返回null：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <h1>hello，world</h1>
    <h2 class="show">标题</h2>
    <!-- 注释 -->
    文本
    <script>

        "use strict";

        let h1 = document.querySelector("h1");
        let h2Attr = document.querySelector("h2").attributes[0];
        let comment = document.body.childNodes[5];
        let text = document.body.childNodes[6];

        console.log(h1.nodeValue);  // null  元素节点获取不到值
        console.log(h2Attr.nodeValue);  // show
        console.log(comment.data);  //  注释
        console.log(text.data);  // 文本

    </script>
</body>

</html>
```



# document对象

## 基本介绍

document是window的一个子对象，是由HTMLDocument实现的实例。

document对象的原型链中包含Node，所以可以使用所有节点操作方法与属性，如nodeType/nodeName等。



## 文档信息

系统为document对象提供了一些属性，可以快速获取一些常用信息。

| 属性     | 说明                       |
| -------- | -------------------------- |
| title    | 获取和设置当前HTML文档标题 |
| URL      | 获取当前HTML文档的URL      |
| domain   | 获取当前HTML文档的域名     |
| referrer | 获取当前HTML文档的来源地址 |

1）使用title属性可或获取和设置当前HTML文档标题：

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
    <script>

        "use strict";

        console.log(document.title);
        setTimeout(() => { document.title = "DOM LEARN" }, 1000);

        // 1s后修改页面title

    </script>
</body>

</html>
```

2）使用URL属性可获取当前HTML文档的URL：

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
    <script>

        "use strict";

        console.log(document.URL);

        // http://localhost:5500/

    </script>
</body>

</html>
```

3）使用domain属性可获取当前HTML文档的域名：

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
    <script>

        "use strict";

        console.log(document.domain);

        // localhost

    </script>
</body>

</html>
```

4）使用referrer属性可获取当前HTML文档的来源地址，可用于页面引流统计或者防盗链上：

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
    <script>

        "use strict";

        console.log(document.referrer);

        // http://localhost:5500/

    </script>
</body>

</html>
```



# 元素节点获取

## 手动获取所有元素节点

下面将通过使用节点的nodeType属性判断并获取所有的元素节点。

```
let html = [...document.childNodes].filter((node) => {
  if (node.nodeType === 1) {
    return node
  }
})[0]
console.log(html)
```





## 快速获取元素节点一览

如果只想获取元素节点，可通过下面的属性或方法进行快速获取。

| 属性/方法                       | 描述                            | 返回值         | 普通节点对象是否可调用 |
| ------------------------------- | ------------------------------- | -------------- | ---------------------- |
| document.documentElement        | 获取html元素节点                | node           | 否                     |
| document.body                   | 获取body元素节点                | node           | 否                     |
| document.head                   | 获取head元素节点                | node           | 否                     |
| document.links                  | 获取a元素节点集合               | HTMLCollection | 否                     |
| document.anchors                | 获取具有name属性的a元素节点集合 | HTMLCollection | 否                     |
| document.forms                  | 获取form元素节点集合            | HTMLCollection | 否                     |
| document.images                 | 获取img元素节点集合             | HTMLCollection | 否                     |
| document.getElmentById()        | 根据id属性获取元素节点          | node           | 否                     |
| document.getElmentByName()      | 根据name属性获取元素节点集合    | NodeList       | 否                     |
| document.getElmentByTagName()   | 根据tagName属性获取元素节点集合 | HTMLCollection | 是                     |
| document.getElmentByClassName() | 根据class属性获取元素节点集合   | HTMLCollection | 是                     |
| document.querySelectorAll()     | 根据CSS选择器获取元素节点集合   | NodeList       | 是                     |
| document.querySelector()        | 根据CSS选择器获取元素节点       | node           | 是                     |



## document.documentElement

document.documentElement属性可获取HTML标签。

- 注意！仅能在document对象中使用该属性！

它返回单一被选中的元素节点：

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
    <script>

        "use strict";

        console.log(document.documentElement);

        // html

    </script>
</body>

</html>
```



## document.body

document.body属性可获取body标签。

- 注意！仅能在document对象中使用该属性！

它返回单一被选中的元素节点：

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
    <script>

        "use strict";

        console.log(document.body);

        // body

    </script>
</body>

</html>
```



## document.head

document.head属性可获取head标签。

- 注意！仅能在document对象中使用该属性！

它返回单一被选中的元素节点：

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
    <script>

        "use strict";

        console.log(document.head);

        // head

    </script>
</body>

</html>
```



## document.links

document.links属性可获取所有的a标签。

- 注意！仅能在document对象中使用该属性！

它返回HTMLCollection元素集合对象：

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

    <a href="#">first</a>
    <a href="#">second</a>
    <a href="#">last</a>

    <script>

        "use strict";

        console.log(document.links);

        // HTMLCollection(3) [a, a, a]

    </script>
</body>

</html>
```



## document.anchors

document.anchors属性可获取所有具有name属性的的a标签锚点。

- 注意！仅能在document对象中使用该属性！

它返回HTMLCollection元素集合对象：

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

    <a href="#" name="first">first</a>
    <a href="#" name="second">second</a>
    <a href="#">last</a>

    <script>

        "use strict";

        console.log(document.anchors);

        // HTMLCollection(2) [a, a, first: a, second: a]

    </script>
</body>

</html>
```



## document.forms

document.links属性可获取所有的form标签。

- 注意！仅能在document对象中使用该属性！

它返回HTMLCollection元素集合对象：

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

    <form action="">
        <input type="text">
    </form>
    <form action="">
        <input type="text">
    </form>
    <form action="">
        <input type="text">
    </form>
    <script>

        "use strict";

        console.log(document.forms);

        // HTMLCollection(3) [form, form, form]

    </script>
</body>

</html>
```



## document.images

document.images属性可获取所有的img标签。

- 注意！仅能在document对象中使用该属性！

它返回HTMLCollection元素集合对象：

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

    <img src="" alt="">
    <img src="" alt="">
    <img src="" alt="">

    <script>

        "use strict";

        console.log(document.images);

        // HTMLCollection(3) [img, img, img]

    </script>
</body>

</html>
```



## document.getElmentById()

document.getElmentById()方法可根据id属性获取元素节点。

- 注意！仅能在document对象中使用该方法！

它返回单一被选中的元素节点：

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

    <div id="div-1"></div>
    <div id="div-2"></div>
    <div id="div-3"></div>

    <script>

        "use strict";

        console.log(document.getElementById("div-1"));

        // <div id="div-1"></div>

    </script>
</body>

</html>
```

拥有标准属性id的元素可做为window对象的属性进行访问，但注意不要与变量名进行重合，所以不推荐这种方式。

这也是为什么推荐在为标签id属性取名的时候用-进行分割而不是用_进行分割的原因。

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

    <div id="first_div"></div>
    <div id="last-div"></div>

    <script>

        "use strict";

        console.log(first_div);

        // <div id="first_div"></div>

    </script>
</body>

</html>
```



## document.getElmentByName()

document.getElmentByName()方法可根据name属性获取元素节点集合。

- 注意！仅能在document对象中使用该方法！
- 该方法通常用于获取表单元素标签，但实际上只要有name属性的标签它都能获取到

它返回NodeList元素集合对象：

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

    <form action="#">
        <input type="text" name="username">
        <input type="text" name="password">
    </form>

    <p name="username"></p>

    <script>

        "use strict";

        let nameAttributeElementCollection = document.getElementsByName("username")

        console.log(nameAttributeElementCollection);

        // NodeList(2) [input, p]

    </script>
</body>

</html>
```



## document.getElmentByTagName()

document.getElmentByTagName()方法可根据tagName属性获取元素节点集合。

- 注意！不论是document对象还是普通的节点对象都能使用该方法！
- 元素名不区分大小写
- 可使用通配符\*获取所有元素节点

它返回HTMLCollection元素集合对象：

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

    <div id="div-1">
        <span></span>
    </div>
    <div id="div-2">
        <mark></mark>
    </div>
    <div id="div-3">
        <time></time>
    </div>

    <script>

        "use strict";

        let divCollection = document.getElementsByTagName("div");
        console.log(divCollection);

        let timeCollection = divCollection[2].getElementsByTagName("time");
        console.log(timeCollection);

        // HTMLCollection(3) [div#div-1, div#div-2, div#div-3, div-1: div#div-1, div-2: div#div-2, div-3: div#div-3]
        // HTMLCollection [time]

    </script>
</body>

</html>
```





## document.getElmentByClassName()

document.getElmentByClassName()方法可根据class属性获取元素节点集合。

- 注意！不论是document对象还是普通的节点对象都能使用该方法！

它返回HTMLCollection元素集合对象：

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

    <main>
        <ul class="list">
            <li class="list">1</li>
            <li class="list">2</li>
        </ul>
    </main>

    <ul class="list">
        <li class="list">A</li>
        <li class="list">B</li>
    </ul>

    <script>

        "use strict";

        let nodeMain = document.getElementsByTagName("main")[0];
        let classListCollection = nodeMain.getElementsByClassName("list")
        console.log(classListCollection);

        // HTMLCollection(3) [ul.list, li.list, li.list]

    </script>
</body>

</html>
```









## querySelectorAll()

document.querySelectorAll()方法可根据CSS Selectors获取元素节点集合。

- 注意！不论是document对象还是普通的节点对象都能使用该方法！

它返回NodeList元素集合对象：

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

    <header></header>
    <main>
        <nav>
            <ul>
                <li>1</li>
                <li>2</li>
                <li>3</li>
            </ul>
        </nav>
        <section>
            <article>A</article>
            <article>B</article>
            <article>C</article>
        </section>
    </main>
    <footer></footer>

    <script>

        "use strict";

        console.log(document.querySelectorAll("main section article:first-of-type"));  // NodeList [article]
        console.log(document.querySelectorAll("main nav ul li")); // NodeList(3) [li, li, li]
        console.log(document.querySelectorAll("main section article:nth-child(2)")); // NodeList [article]

    </script>
</body>

</html>
```



## querySelector()

document.querySelector()方法可根据CSS Selectors获取元素节点。

- 注意！不论是document对象还是普通的节点对象都能使用该方法！
- 如果有多个符合条件的元素，则只返回第一个，也就是说它相当于document.querySelectorAll()[0]

它返回单一被选中的元素节点：

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

    <header></header>
    <main>
        <nav>
            <ul>
                <li>1</li>
                <li>2</li>
                <li>3</li>
            </ul>
        </nav>
        <section>
            <article>A</article>
            <article>B</article>
            <article>C</article>
        </section>
    </main>
    <footer></footer>

    <script>

        "use strict";

        console.log(document.querySelector("main:nth-child(2) ul li"));

        // <li><font style="vertical-align: inherit;"><font style="vertical-align: inherit;">1</font></font></li>

    </script>
</body>

</html>
```



# 表单项查找

## 组合查找

JavaScript为表单的查找提供了单独的属性接口：

- 使用document.forms可获取所有的表单元素
- 根据form的name属性获取单独的表单元素
- 使用form.elements可获取表单元素下的所有表单项元素
- 根据表单项的name属性使用form.elements.title可获取单独的表单项元素，它可以直接简写成form.name
- 针对form.name获取到radio或者checkbox，返回的结果将是一个集合而不是单独的表单项元素

示例如下：

```
<body>
    <form action="#" name="register">
        <input type="text" name="username">
        <input type="text" name="pwd">
    </form>
</body>
<script>

    "use strict";

    let formCollection = document.forms;  // 获取所有表单
    let formNode = formCollection.register; // 根据name获取某个表单
    let formChildren = formNode.elements;  // 拿到其下所有表单项
    let username = formNode.elements.username; // 拿到name为username的表单项

    console.log(formChildren);  // HTMLFormControlsCollection(2) [input, input, username: input, pwd: input]
    console.log(username);  // <input type="text" name="username">

</script>
```





## 获取select选中值

如果想获取select中被选中的option，则可使用以下方法：

- 先获取select标签
- 根据select节点的options属性获取其下所有的option的集合
- 循环遍历options集合，判断option是否具有selected属性

如下所示：

```
<body>
    <form action="#" name="register">
        <div>
            <select name="city" id="city" multiple>
                <option value="bj">北京</option>
                <option value="sh">上海</option>
                <option value="sz">深圳</option>
                <option value="gz">广州</option>
            </select>
        </div>
        <div>
            <button type="button">检测</button>
        </div>
    </form>
</body>
<script>

    "use strict";

    let btnNode = document.querySelector("button");
    let selectNode = document.forms.register.city;
    btnNode.addEventListener("click", (event) => {
        Array.from(selectNode.options, (option, index, array) => {
            if (option.selected) {
                console.log(option.value, option.innerText, "被选中");
            }
        });
    });

</script>
```



## 获取file的文件对象

如果想获取一个文件选择框中的文件对象，则可使用以下方法：

- 先获取input标签
- 根据input节点的files属性获取其下所有的已上传文件的集合
- 根据已上传文件集合的索引操作，获取你想要的文件对象

如下所示：

```
<body>
    <form action="#" name="register">
        <div>
            <input type="file" name="avatar" id="avatar" multiple>
        </div>
        <div>
            <button type="button">检测</button>
        </div>
    </form>
</body>
<script>

    "use strict";

    let btnNode = document.querySelector("button");
    let fileInputNode = document.forms.register.avatar;
    btnNode.addEventListener("click", (event) => {
        console.log(fileInputNode.files);  // 可使用索引操作，获取你想要的文件对象
    });

</script>
```





# 元素节点筛选

## 属性筛选 matches()

matches()方法验证某一个节点是否具有特定属性。

如下示例，我们获取了所有的元素节点，现在要通过Array原型借用filter()方法筛选出具有show属性的元素：

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

    <div show></div>
    <p hidden></p>
    <span hidden></span>
    <mark show></mark>

    <script>

        "use strict";

        let allElementNodeList = document.querySelectorAll("*");
        let showElementArray = Array.prototype.filter.call(allElementNodeList, (element, index, nodelst) => {
            return element.matches("[show]");
        })

        console.log(showElementArray);

        // (2) [div, mark]

    </script>
</body>

</html>
```



## 祖先筛选 closest()

closest()方法会查找最近的符合选择器的祖先元素（包括自身）。

找祖先，看最近的祖先能不能被选择器选中，如果不能继续向上找。

如下示例，我们获取了.float元素，需要查找它的祖先中具有show属性的标签：

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

    <main class="show">
        <div class="hidden">
            <article class="float"></article>
        </div>
    </main>

    <script>

        "use strict";

        let floatNode = document.querySelector(".float");
        console.log(floatNode.closest(".show"));

        // <main class="show">...</main>

    </script>
</body>

</html>
```



# 节点关系获取

## 普通节点关系获取一览

节点对象可通过以下属性可获取与其他节点之间关系。

它们包含文本节点与注释节点：

| 节点属性        | 描述                   | 返回值   |
| --------------- | ---------------------- | -------- |
| childNodes      | 获取子节点对象         | node     |
| parentNode      | 获取父节点对象         | NodeList |
| firstChild      | 获取第一个子节点对象   | node     |
| lastChild       | 获取最后一个子节点对象 | node     |
| previousSibling | 获取上一个兄弟节点对象 | node     |
| nextSibling     | 获取下一个兄弟节点对象 | node     |

由于使用的较少，这里不再进行演示，注意2点：

- 换行也包括在节点关系中，它是一个单独的#text文本节点
- 文本节点不能作为父节点，因为它不会嵌套任何子节点



## 元素节点关系获取一览

节点对象可通过以下属性获取与其他元素节点之间的关系。

它们不包含文本节点与注释节点，所以是非常常用的。

| 节点属性               | 描述                       | 返回值         |
| ---------------------- | -------------------------- | -------------- |
| parentElement          | 获取父元素节点对象         | node           |
| children               | 获取子元素节点对象         | HTMLCollection |
| childElementCount      | 获取子元素节点对象的数量   | number         |
| firstElementChild      | 获取第一个子元素节点对象   | node           |
| lastElementChild       | 获取最后一个子元素节点对象 | node           |
| previousElementSibling | 获取上一个兄弟元素节点对象 | node           |
| nextElementSibling     | 获取下一个兄弟元素节点对象 | node           |



## parentElment

parentElment属性用于获取父元素节点对象，它返回单一被选中的元素节点：

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

    <header></header>
    <main>
        <ul>
            <li id="li-1">1</li>
            <li id="li-2">2</li>
            <li id="li-3">3</li>
            <li id="li-4">4</li>
            <li id="li-5">5</li>
            <li id="li-6">6</li>
            <li id="li-7">7</li>
        </ul>
    </main>
    <footer></footer>

    <script>

        "use strict";

        let ulNode = document.querySelector("ul");
        console.log(ulNode.parentElement);

        // main

    </script>
</body>

</html>
```



## children

children属性用于获取子元素节点对象，它返回HTMLCollection元素集合对象：

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

    <header></header>
    <main>
        <ul>
            <li id="li-1">1</li>
            <li id="li-2">2</li>
            <li id="li-3">3</li>
            <li id="li-4">4</li>
            <li id="li-5">5</li>
            <li id="li-6">6</li>
            <li id="li-7">7</li>
        </ul>
    </main>
    <footer></footer>

    <script>

        "use strict";

        let ulNode = document.querySelector("ul");
        console.log(ulNode.children);

        // HTMLCollection(7) [li#li-1, li#li-2, li#li-3, li#li-4, li#li-5, li#li-6, li#li-7, li-1: li#li-1, li-2: li#li-2, li-3: li#li-3, li-4: li#li-4, li-5: li#li-5, …]

    </script>
</body>

</html>
```



## firstElementChild

firstElementChild属性用于获取第一个子元素节点对象，它返回单一被选中的元素节点：

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

    <header></header>
    <main>
        <ul>
            <li id="li-1">1</li>
            <li id="li-2">2</li>
            <li id="li-3">3</li>
            <li id="li-4">4</li>
            <li id="li-5">5</li>
            <li id="li-6">6</li>
            <li id="li-7">7</li>
        </ul>
    </main>
    <footer></footer>

    <script>

        "use strict";

        let ulNode = document.querySelector("ul");
        console.log(ulNode.firstElementChild);

        // li#li-1

    </script>
</body>

</html>
```



## lastElmentChild

lastElementChild属性用于获取最后一个子元素节点对象，它返回单一被选中的元素节点：

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

    <header></header>
    <main>
        <ul>
            <li id="li-1">1</li>
            <li id="li-2">2</li>
            <li id="li-3">3</li>
            <li id="li-4">4</li>
            <li id="li-5">5</li>
            <li id="li-6">6</li>
            <li id="li-7">7</li>
        </ul>
    </main>
    <footer></footer>

    <script>

        "use strict";

        let ulNode = document.querySelector("ul");
        console.log(ulNode.lastElementChild);

        // li#li-7

    </script>
</body>

</html>
```



## previousElementSibling

previousElementSibling属性用于获取上一个兄弟元素节点对象，它返回单一被选中的元素节点：

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

    <header></header>
    <main>
        <ul>
            <li id="li-1">1</li>
            <li id="li-2">2</li>
            <li id="li-3">3</li>
            <li id="li-4">4</li>
            <li id="li-5">5</li>
            <li id="li-6">6</li>
            <li id="li-7">7</li>
        </ul>
    </main>
    <footer></footer>

    <script>

        "use strict";

        let mainNode = document.querySelector("main");
        console.log(mainNode.previousElementSibling);

        // header

    </script>
</body>

</html>
```



## nextElementSibling

nextElementSibling属性用于获取下一个兄弟元素节点对象，它返回单一被选中的元素节点：

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

    <header></header>
    <main>
        <ul>
            <li id="li-1">1</li>
            <li id="li-2">2</li>
            <li id="li-3">3</li>
            <li id="li-4">4</li>
            <li id="li-5">5</li>
            <li id="li-6">6</li>
            <li id="li-7">7</li>
        </ul>
    </main>
    <footer></footer>

    <script>

        "use strict";

        let mainNode = document.querySelector("main");
        console.log(mainNode.nextElementSibling);

        // footer

    </script>
</body>

</html>
```



# 元素节点集合对象

## 基本介绍

JavaScript中共有2种元素节点集合对象，分别是NodeList和HTMLCollection。

它们的相同点在于：

- 都是类数组
- 都具有length属性
- 都可以用index进行元素项访问
- 都提供了item()方法来根据index操纵元素项
- 都实现了迭代器协议，可以通过for/of进行遍历

它们的不同点在于：

- NodeList是动态映射DOM的（querySelectorAll()方法获得的NodeList除外），而HTMLCollection不是
- HTMLCollection提供了namedItem()方法来根据元素name属性、id属性提取元素项
- NodeList提供了forEach()方法来进行遍历，但是HTMLCollection未提供该方法

两者对比而言，其实NodeList更加的优秀，它支持动态映射DOM这个就令HTMLCollection汗颜了。



## length

NodeList与HTMLCollection都包含length属性，记录了其中包含元素节点的数量：

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

    <ul>
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ul>

    <script>

        "use strict";

        let liNodeList = document.querySelectorAll("li");
        let liCollection = document.getElementsByTagName("li");

        console.log(liNodeList.length);
        console.log(liCollection.length);

        // 3
        // 3

    </script>

</body>

</html>
```



## item()

NodeList与HTMLCollection都提供了item()方法来根据索引获取元素：

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

    <ul>
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ul>

    <script>

        "use strict";

        let liNodeList = document.querySelectorAll("li");
        let liCollection = document.getElementsByTagName("li");

        console.log(liNodeList.item(0));
        console.log(liCollection.item(0));

        // 等同于

        console.log(liNodeList[0]);
        console.log(liCollection[0]);

    </script>

</body>

</html>
```



## namedItem()

HTMLCollection提供了namedItem()方法来根据元素name属性、id属性提取元素项：

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

    <div name="div-name">1</div>
    <div id="div-id">2</div>
    <div>3</div>
    <div>4</div>
    <div>5</div>

    <script>

        "use strict";

        let divCollection = document.getElementsByTagName("div");
        console.log(divCollection.namedItem("div-name"));
        console.log(divCollection.namedItem("div-id"));

        // <div name="div-name">1</div>
        // <div id="div-id">2</div>

    </script>

</body>

</html>
```



## 动态映射

NodeList拥有动态映射的特征，也就是说当DOM上的元素对象发生改变后NodeList中的元素项也会发生改变。

- 注意：querySelectorAll()方法返回的NodeList是静态的

动态特性示例，在获取了NodeList后新增了一个标签插入文档中，可以发现NodeList也会新增标签：

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

    <div name="show"></div>
    <div name="show"></div>
    <div name="show"></div>

    <script>

        "use strict";

        let divNodeList = document.getElementsByName("show");
        console.log(divNodeList.length);  // 3

        let newDiv = document.createElement("div");
        newDiv.setAttribute("name", "show");  // div的name属性是特征属性，input的name是标准属性

        document.body.append(newDiv);

        console.log(divNodeList.length); // 4

    </script>

</body>

</html>
```

querySelectorAll()方法返回的NodeList是静态的：

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

    <div name="show"></div>
    <div name="show"></div>
    <div name="show"></div>

    <script>

        "use strict";

        let divNodeList = document.querySelectorAll("[name=show]");
        console.log(divNodeList.length);  // 3

        let newDiv = document.createElement("div");
        newDiv.setAttribute("name", "show");  // div的name属性是特征属性，input的name是标准属性

        document.body.append(newDiv);

        console.log(divNodeList.length); // 3

    </script>

</body>

</html>
```



## 数组转换

HTMLCollection和NodeList都可以转换为数组，我们以NodeList示例。

1）通过Array.form()方法将NodeList转换为数组：

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

    <div name="show"></div>
    <div name="show"></div>
    <div name="show"></div>

    <script>

        "use strict";

        let divNodeList = document.querySelectorAll("[name=show]");
        let divNodeArray = Array.from(divNodeList);
        console.log(divNodeArray);

        // (3) [div, div, div]

    </script>

</body>

</html>
```

2）通过…语法将NodeList转换为数组：

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

    <div name="show"></div>
    <div name="show"></div>
    <div name="show"></div>

    <script>

        "use strict";

        let divNodeList = document.querySelectorAll("[name=show]");
        let divNodeArray = [...divNodeList];
        console.log(divNodeArray);

        // (3) [div, div, div]

    </script>

</body>

</html>
```

其实在通过Array.form()方法将NodeList转换为数组时，可对mapfn形参传入一个回调函数用以操作数据项。

如下所示，我们只提取元素节点的文本内容：

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

    <div>A</div>
    <div>B</div>
    <div>C</div>

    <script>

        "use strict";

        let ary = Array.from(
            document.querySelectorAll("div"),
            (element, index) => {
                return element.innerText;
            }
        );

        console.log(ary);

        // ["A", "B", "C"]

    </script>

</body>

</html>
```



## 原型借用

HTMLCollection和NodeList都是类数组，所以可以很好的借用Array原型对象的方法。

如下示例，计算书籍总价格并写入到文档中：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        table#book-table {
            width: 300px;
            border: 1px solid #ddd;
            caption-side: top;
            border-collapse: collapse;
        }

        table#book-table th,
        td,
        caption {
            border: 1px solid #ddd;
        }

        table#book-table caption {
            background-color: #aaa;
            font-weight: bolder;
            padding: .3rem;
        }

        table#book-table tfoot td {
            font-weight: bolder;
        }

        table#book-table th,
        td {
            padding: .2rem;
            text-align: center;
        }

        table tbody tr:nth-child(odd) {
            background-color: #eee;
        }

        table tbody tr:hover {
            background-color: #aaa;
        }

        table tbody tr * {
            cursor: pointer;
        }
    </style>
</head>

<body>

    <table id="book-table">
        <caption>book table</caption>
        <thead>
            <tr>
                <th>book name</th>
                <th>book price</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td book>JavaScript</td>
                <td>32</td>
            </tr>
            <tr>
                <td book>HTML5</td>
                <td>24</td>
            </tr>
            <tr>
                <td book>CSS3</td>
                <td>36</td>
            </tr>
        </tbody>
        <tfoot>
            <td>total price</td>
            <td id="total-price"></td>
        </tfoot>
    </table>

    <script>

        "use strict";

        let bookNodeList = document.querySelectorAll("[book]");
        let bookTotalPrice = Array.prototype.reduce.call(bookNodeList, (prev, cur, idx, nodelst) => {
            return prev + Number(cur.nextElementSibling.innerText);
        }, 0)

        document.querySelector("#total-price").innerText = bookTotalPrice;

    </script>

</body>

</html>
```

渲染结果：

![image-20210802210351161](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210802210351161.png)





## 对象遍历

NodeList和Collection都支持的遍历方式有：

- for遍历
- for/of遍历

此外，NodeList还支持forEach()方法进行遍历。

这里不再进行演示，其实它们可以通过Array原型借用获得更好的遍历体验，如借用reduce()、map()、every()、some()等方法实现遍历。





# 节点对象创建

## 创建文本节点 document.createTextNode()

使用document.createTextNode()方法来创建文本节点对象：

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

    <div></div>

    <script>

        "use strict";

        let textNode = document.createTextNode("HELLO WORLD");

        console.log(textNode);  // "HELLO WORLD"
        console.log(textNode.nodeType);  // 3
        console.log(textNode.nodeName);  // #text
        console.log(textNode.nodeValue); // HELLO WORLD

    </script>

</body>

</html>
```



## 创建元素节点 document.createElement()

使用document.createElement()方法来创建元素节点对象：

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

    <div></div>

    <script>

        "use strict";

        let elementNode = document.createElement("div");

        console.log(elementNode);  // <div></div>
        console.log(elementNode.nodeType);  // 1
        console.log(elementNode.nodeName);  // DIV
        console.log(elementNode.nodeValue); // null  元素节点使用该属性总是返回null

    </script>

</body>

</html>
```





## 克隆节点 node.cloneNode()

节点对象都拥有cloneNode()方法，旨在克隆出一个一模一样的节点。

如果它的deep参数为true，会进行递归的深度克隆。

- 注意，如果事件绑定是绑定在元素身上，则会连同事件一起拷贝

示例如下：

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

    <div onclick="callbackfn(this, event)"><button>A</button></div>

    <script>

        "use strict";

        document.importNode

        function callbackfn(element, event) {
            let ascii = element.firstElementChild.innerText.charCodeAt();
            let newDiv = element.cloneNode(true);
            newDiv.firstElementChild.innerText = String.fromCharCode(ascii + 1);
            element.after(newDiv);
        }

    </script>

</body>

</html>
```



## 导入节点 document.importNode()

document.importNode()和node.cloneNode()的功能基本相同，但是使用上有所不同。

- importNode()方法仅能由document调用

- cloneNode()方法可以由任何节点对象调用

document.importNode()方法有2个参数：

- node：传入要被拷贝的节点对象
- deep：是否进行深度拷贝

值得注意的是，在部分IE浏览器上该方法的支持性并不如node.cloneNode()方法。

如下所示，将node.cloneNode()方法改变为document.importNode()，效果不会发生任何变化：

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

    <div onclick="callbackfn(this, event)"><button>A</button></div>

    <script>

        "use strict";

        function callbackfn(element, event) {
            let ascii = element.firstElementChild.innerText.charCodeAt();
            let newDiv = document.importNode(element, true);
            newDiv.firstElementChild.innerText = String.fromCharCode(ascii + 1);
            element.after(newDiv);
        }

    </script>

</body>

</html>
```





# 元素节点内容

## innerHTML

元素节点可通过innerHTML属性获取或设置标签中嵌套的内容。

如果是设置内容，它将触发浏览器解析器重新绘制DOM。

设置内容时，可以通过innerHTML为源标签创建一个嵌套标签，但不能创建&lt;script&gt;标签 + JavaScript代码，这样做会抛出异常。

如下示例：

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

    <div>
        <h2>HELLO WORLD</h2>
    </div>

    <script>

        "use strict";

        let divNode = document.querySelector("div");

        // 获取内容
        console.log(divNode.innerHTML);

        // <h2>HELLO WORLD</h2>

        // 设置内容 有效，只要不是<script>标签即可
        divNode.innerHTML = "<mark>hello world</mark>"

        // 最后DOM上渲染结果为 ： <div><mark>hello world</mark></div>

    </script>

</body>


</html>
```



## outerHTML

innerHTML只包含内容，而outerHTML包含标签本身。

如下图所示：

![image-20210803152048507](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210803152048507.png)

当要替换outerHTML时，它会将标签本身也一起替换：

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

    <div>
        <h2>HELLO WORLD</h2>
    </div>

    <script>

        "use strict";

        let divNode = document.querySelector("div");

        // 获取内容
        console.log(divNode.outerHTML);

        // <div><h2>HELLO WORLD</h2></div>

        // 设置内容 有效，只要不是script标签即可
        divNode.outerHTML = "<mark>hello world</mark>"

        // 最后DOM上渲染结果为 ： <mark>hello world</mark>

    </script>

</body>


</html>
```



## innerText & textContent

元素节点可通过innerText或textContent属性获取或设置标签中嵌套的文本内容，注意与innerHTML的区别。

innerText和textContent属性的功能都一样，但是兼容性上有一些差别：

- textContent属性不被部分IE浏览器所支持
- innerText属性不被部分FireFox浏览器所支持

在获取时，忽略标签本身，只获取其文本内容。

在设置时，不能设置嵌套标签，它将被当做普通文本进行对待。

示例如下：

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

    <div>外层文本<span>内层文本</span></div>

    <script>

        "use strict";

        let divNode = document.querySelector("div");

        // 在获取时，忽略标签本身，只获取其文本内容。
        console.log(divNode.innerHTML);  // 外层文本<span>内层文本</span>
        console.log(divNode.innerText);  // 外层文本内层文本

        // 在设置时，不能设置嵌套标签，它将被当做普通文本进行对待。
        divNode.innerText = "<h2>HELLO WORLD</h2>"

        // 最后页面上会直接显示出 ： <h2>hello world</h2>，并不会做标签渲染

    </script>

</body>


</html>
```



## outerText

区别点同innerHTML和outerHTML相同。

即在替换时，是否连同标签本身一起替换。

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

    <div>外层文本<span>内层文本</span></div>

    <script>

        "use strict";

        let divNode = document.querySelector("div");

        divNode.outerHTML = "<p>不会解析成标签</p>";

    </script>

</body>


</html>
```



# 节点对象管理

## 推荐方法

在当前版本，JavaScript中对节点对象的管理主要使用以下方法，它们都支持插入、删除、替换文本节点和元素节点。

以下方法是任意节点对象都可调用的，其中也包括document对象：

| 方法          | 描述                                               |
| ------------- | -------------------------------------------------- |
| append()      | 在节点内部的末尾添加新的元素节点对象或文本节点对象 |
| prepend()     | 在节点内部的首部插入新的元素节点对象或文本节点对象 |
| before()      | 在节点同级的前面插入新的元素节点对象或文本节点对象 |
| after()       | 在节点同级的后面插入新的元素节点对象或文本节点对象 |
| replaceWith() | 将当前节点对象替换为新的元素节点对象或文本节点对象 |



## append()

在节点内部的末尾添加新的元素节点对象或文本节点对象。

如图所示：

![image-20210803154153432](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210803154153432.png)

代码示例：

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

    <h1>unorder list start</h1>
    <ul>
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ul>
    <h1>unorder list end</h1>

    <script>

        "use strict";

        let ulNode = document.querySelector("ul");
        let newNode = document.createElement("li");
        newNode.innerText = "append()";

        ulNode.append(newNode);

    </script>

</body>


</html>
```

渲染结果：

![image-20210803155121564](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210803155121564.png)



## prepend()

在节点内部的首部插入新的元素节点对象或文本节点对象。

如图所示：

![image-20210803154341605](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210803154341605.png)

代码示例：

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

    <h1>unorder list start</h1>
    <ul>
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ul>
    <h1>unorder list end</h1>

    <script>

        "use strict";

        let ulNode = document.querySelector("ul");
        let newNode = document.createElement("li");
        newNode.innerText = "prepend()";

        ulNode.prepend(newNode);

    </script>

</body>


</html>
```

渲染结果：

![image-20210803155200570](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210803155200570.png)

## before()

在节点同级的前面插入新的元素节点对象或文本节点对象。

如图所示：

![image-20210803154502703](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210803154502703.png)



代码示例：

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

    <h1>unorder list start</h1>
    <ul>
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ul>
    <h1>unorder list end</h1>

    <script>

        "use strict";

        let ulNode = document.querySelector("ul");
        let newNode = document.createElement("h2");
        newNode.innerText = "before()";

        ulNode.before(newNode);

    </script>

</body>


</html>
```

渲染结果：

![image-20210803155312346](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210803155312346.png)



## after()

在节点同级的后面插入新的元素节点对象或文本节点对象。

如图所示：

![image-20210803154515372](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210803154515372.png)

代码示例：

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

    <h1>unorder list start</h1>
    <ul>
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ul>
    <h1>unorder list end</h1>

    <script>

        "use strict";

        let ulNode = document.querySelector("ul");
        let newNode = document.createElement("h2");
        newNode.innerText = "after()";

        ulNode.after(newNode);

    </script>

</body>


</html>
```

渲染结果：

![image-20210803155344728](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210803155344728.png)



## replaceWith()

将当前节点对象替换为新的元素节点对象或文本节点对象。

如图所示：

![image-20210803154814275](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210803154814275.png)

代码示例，将h1替换为mark，保留原本的内容：

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

    <h1>unorder list start</h1>
    <ul>
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ul>
    <h1>unorder list end</h1>

    <script>

        "use strict";

        setTimeout(() => {
            let h1NodeList = document.querySelectorAll("h1");
            h1NodeList.forEach((element, index, lst) => {
                let elementContent = element.innerHTML;
                let newNode = document.createElement("mark");
                newNode.innerHTML = elementContent;
                element.replaceWith(newNode);
            })
        }, 5000);

    </script>

</body>


</html>
```

渲染结果：

1）替换之前：

![image-20210803155652916](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210803155652916.png)

2）替换之后：

![image-20210803155710027](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210803155710027.png)



## 其他方法

上面介绍的5个节点管理方法可针对文本节点、也可针对元素节点，是最常用的节点管理方法，没有什么局限性。

而下面介绍的方法3个节点管理方法或只针对文本节点、或只针对元素节点，都有一定的局限性，因此不太常用。

这3个方法任意节点对象都可调用的，其中也包括document对象：

- insertAdjacentText()：只能将文本节点对象插入到节点的某个位置上，它不会对文本中的标签进行解析
- insertAdjacentHTML()：只能将文本节点对象插入到节点的某个位置上，它可以对文本中的标签进行解析
- insertAdjacentElement()：只能将元素节点对象插入到节点的某个位置上

这3个方法都有2个参数：

- InsertPosition：插入的位置
- text、html、insertedElement：待插入的对象

其中，插入位置可指定的字符串如下所示：

| InsertPosition | 描述         |
| -------------- | ------------ |
| beforebegin    | 元素同级前面 |
| afterend       | 元素同级后面 |
| afterbegin     | 元素内部前面 |
| beforeend      | 元素内部后面 |

代码示例：

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

    <h1>unorder list start</h1>
    <ul>
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ul>
    <h1>unorder list end</h1>

    <script>

        "use strict";

        let node = document.querySelector("ul");

        // 只能将文本节点对象插入到节点的某个位置上，它不会对文本中的标签进行解析
        node.insertAdjacentText("beforeend", "new text");

        // 只能将文本节点对象插入到节点的某个位置上，它可以对文本中的标签进行解析
        node.insertAdjacentHTML("beforeend", "<li>new html li</li>");

        // 只能将元素节点对象插入到节点的某个位置上
        let newNode = document.createElement("li");
        newNode.innerHTML = "new element li"
        node.insertAdjacentElement("beforeend", newNode);

    </script>

</body>


</html>
```

渲染结果：

![image-20210803161417380](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210803161417380.png)



## 古老方法

下面是过去使用的操作节点的方法，现在不建议使用了，在阅读一些老旧代码时可参照此表：

| 方法           | 说明                         |
| -------------- | ---------------------------- |
| appendChild()  | 添加子节点对象               |
| insertBefore() | 在同级的前面插入一个节点对象 |
| removeChild()  | 删除子节点对象               |
| replaceChild() | 替换子节点对象               |





# 标准属性操作

## 什么是标准属性

标准属性即为HTML标签中自带的一些常见属性。

如：id、class、&lt;input&gt;的name、value、readonly、disabled等。

一言以蔽之，只要不是自定义的标签属性都被称之为标准属性。



## 一些特殊的标准属性

一些标准属性的名字和JavaScript关键字有冲突，对此JavaScript对这些标准属性进行了重命名操作：

- 标准属性class被重命名为className
- 标准属性for被重命名为htmlFor



## 如何操纵标准属性

对元素节点对象使用点语法，跟上标准属性名即可。

- 标准属性的获取不一定完全是字符串，也有可能是布尔值
- 操作标准属性时，严格区分大小写
- 操作标准属性时，采用驼峰命名法，如readonly要变更为readOnly

如下所示：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        .blue {
            color: blue;
        }

        .tilt {
            font-style: italic;
        }
    </style>
</head>

<body>

    <h1>HELLO WORLD</h1>
    <input type="text" readonly placeholder="this is input element">

    <script>

        "use strict";

        let h1ElementNode = document.querySelector("h1");
        h1ElementNode.className = "blue tilt";

        let inputElementNode = document.querySelector("input");
        console.log(inputElementNode.readOnly);    // true
        console.log(inputElementNode.placeholder); // this is input element
        console.log(inputElementNode.value);  //
        console.log(inputElementNode.type);   // text

    </script>

</body>


</html>
```



## 常见标准属性与设置

以下例举常见标准属性与值的设置：

| 标签类型       | 标准属性    | 值类型  |
| -------------- | ----------- | ------- |
| 全标签         | id          | string  |
| 全标签         | className   | string  |
| 全标签         | hidden      | boolean |
| &lt;a&gt;      | href        | string  |
| &lt;a&gt;      | target      | string  |
| &lt;img&gt;    | src         | string  |
| &lt;img&gt;    | alt         | string  |
| &lt;form&gt;   | action      | string  |
| &lt;form&gt;   | method      | string  |
| &lt;form&gt;   | enctype     | string  |
| &lt;form&gt;   | novalidate  | boolean |
| &lt;input&gt;  | name        | string  |
| &lt;input&gt;  | value       | string  |
| &lt;input&gt;  | placeholder | string  |
| &lt;input&gt;  | required    | boolean |
| &lt;input&gt;  | readonly    | boolean |
| &lt;input&gt;  | disabled    | boolean |
| &lt;input&gt;  | checked     | boolean |
| &lt;select&gt; | multiple    | boolean |
| &lt;option&gt; | selected    | boolean |



# 特征属性操作

## 什么是特征属性

特征属性即我们为HTML标签自定义的一些属性。

一言以蔽之，只要不是自带的标签属性都被称之为特征属性



## 特征属性的操作方法

特征属性不能使用点语法进行操作，而是要用节点对象提供的方法进行操作。

- 特征属性的操作不区分大小写
- 特征属性的值都为string类型

节点属性操作常用方法如下所示，它们不仅支持操作标准属性，还支持操作特征属性：

| 方法              | 说明                 |
| ----------------- | -------------------- |
| getAttribute()    | 获取某一属性         |
| setAttribute()    | 设置某一属性         |
| removeAttribute() | 删除某一属性         |
| hasAttribute()    | 检测某一属性是否存在 |



## getAttribute()

获取某一属性，支持获取标准、特征属性：

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

    <div class="structure" id="div-1" data="this is div"></div>

    <script>

        "use strict";
        let divNode = document.querySelector("div");

        // 标准属性获取
        console.log(divNode.getAttribute("class"));
        console.log(divNode.getAttribute("id"));

        // 特征属性获取
        console.log(divNode.getAttribute("data"));

        // structure
        // div-1
        // this is div

    </script>

</body>


</html>
```



## setAttribute()

设置某一属性，支持设置标准、特征属性：

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

    <div class="structure" id="div-1" data="this is div"></div>

    <script>

        "use strict";
        let divNode = document.querySelector("div");

        // 设置标准属性
        divNode.setAttribute("class", "c1 c2 c3");
        divNode.setAttribute("id", "div--1");

        // 设置特征属性
        divNode.setAttribute("data", "new description");

        console.log(divNode);

        // <div class="c1 c2 c3" id="div--1" data="new description"></div>

    </script>

</body>


</html>
```



## removeAttribute()

删除某一属性，支持删除标准、特征属性：

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

    <div class="structure" id="div-1" data="this is div"></div>

    <script>

        "use strict";
        let divNode = document.querySelector("div");

        // 删除标准属性
        divNode.removeAttribute("class");

        // 删除特征属性
        divNode.removeAttribute("data");

        console.log(divNode);

        // <div id="div-1"></div>

    </script>

</body>


</html>
```



## hasAttribute()

检测某一属性是否存在，支持检测标准、特征属性：

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

    <div class="structure" id="div-1" data="this is div"></div>

    <script>

        "use strict";
        let divNode = document.querySelector("div");

        // 检测标准属性
        console.log(divNode.hasAttribute("class"));

        // 检测特征属性
        console.log(divNode.hasAttribute("data"));

        // true
        // true

    </script>

</body>


</html>
```



## attributes返回所有属性

节点对象可使用attributes属性来返回所有的特征、标准属性。

attributes将返回一个NameNodeMap对象：

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

    <div class="structure" id="div-1" data="this is div"></div>

    <script>

        "use strict";
        let divNode = document.querySelector("div");

        console.log(divNode.attributes);

        // NamedNodeMap {0: class, 1: id, 2: data, class: class, id: id, data: data, length: 3}

    </script>

</body>


</html>
```

NameNodeMap是一个可迭代对象，你可以使用for/of迭代它或者直接使用index对其进行操作：

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

    <div class="structure" id="div-1" data="this is div"></div>

    <script>

        "use strict";
        let divNode = document.querySelector("div");
        let divNodeAttributes = divNode.attributes;

        console.log(divNodeAttributes[0]);

        // class="structure"

        for (let v of Object.values(divNodeAttributes)) {
            console.log(v.nodeName, v.nodeValue);
        }

        // class structure
        // id div-1
        // data this is div

    </script>

</body>


</html>
```



## dataset 特征属性集

我们自定义的特征属性在某些时候很容易和标准属性名字发生冲突。

针对这种情况，我们只需要在特征属性定义时前面加上data-的前缀即可。

要获取这些特征属性时，可直接通过node.dataset属性集进行获取，属性集和以data-开头的特征属性之间拥有动态映射的关系，你可以在属性集中访问、修改、删除这些以data-开头的特征属性。

- 通过属性集去获取特征属性时，忽略data-的前缀
- 通过属性集去获取特征属性时，要按照小驼峰式命名法
- 通过属性集去操作特征属性时，DOM也会相应发生变化

1）只有data-开头的特征属性才会被加入属性集中，并且在属性集中存储这些特征属性时会去掉data-开头的前缀：

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

    <div class="show" p="left" fc="#000" fz="1.5rem" data-position="right" data-font-color="#888" data-font-size="2rem">
        HELLO WORLD</div>

    <script>

        "use strict";

        let divNode = document.querySelector("div");
        let divDataSet = divNode.dataset;
        console.log(divDataSet);

        // DOMStringMap {position: "left", fontColor: "#888", fontSize: "2rem"}

    </script>

</body>


</html>
```

2）属性集的删改操作都会影响到元素本身，这说明dataset具有动态响应的特性：

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

    <div class="show" p="left" fc="#000" fz="1.5rem" data-position="right" data-font-color="#888" data-font-size="2rem">
        HELLO WORLD</div>

    <script>

        "use strict";

        let divNode = document.querySelector("div");
        let divDataSet = divNode.dataset;

        // 改
        divDataSet.fontSize = "30px";
        divDataSet.position = "center";
        // 删
        delete divDataSet.fontColor;

        console.log(divDataSet);
        // DOMStringMap {position: "center", fontSize: "30px"}
        console.log(divNode);
        // <div class="show" p="left" fc="#000" fz="1.5rem" data-position="center" data-font-size="30px">HELLO WORLD</div>

    </script>

</body>


</html>
```





# 样式设置

## style

元素节点使用style属性可以单独的为某一个元素进行单一的样式设置。

当被设置的样式名有-分割时，应当采用小驼峰命名法：

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

    <div>HELLO WORLD</div>

    <script>

        "use strict";

        let divNode = document.querySelector("div");

        divNode.style.color = "red";
        divNode.style.fontSize = "2rem";
        divNode.style.fontStyle = "italic";

        // <div style="color: red; font-size: 2rem; font-style: italic;">HELLO WORLD</div>

    </script>

</body>


</html>
```



## style.cssText

元素节点使用style.cssText属性可以单独的为某一个元素进行多种样式设置。

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

    <div>HELLO WORLD</div>

    <script>

        "use strict";

        let divNode = document.querySelector("div");

        divNode.style.cssText = "color: red; font-size: 2rem; font-style: italic;";

        // <div style="color: red; font-size: 2rem; font-style: italic;">HELLO WORLD</div>

    </script>

</body>


</html>
```



## setAttribute()

元素节点使用setAttribute()方法也可以单独的为某一个元素进行多种样式设置。

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

    <div>HELLO WORLD</div>

    <script>

        "use strict";

        let divNode = document.querySelector("div");

        divNode.setAttribute("style", "color: red; font-size: 2rem; font-style: italic;");

        // <div style="color: red; font-size: 2rem; font-style: italic;">HELLO WORLD</div>

    </script>

</body>


</html>
```



## classList

元素节点可以通过classList属性下提供的方法来切换已有的class可实现批量化的样式控制。

classList属性下可使用的方法如下表所示：

| 方法                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| classList.contains() | 检测某个class是否存在                                        |
| classList.add()      | 新增class                                                    |
| classList.remove()   | 删除class                                                    |
| classList.toggle()   | 如果class以存在，则删除，如果class不存在，则新增，相当于切换 |

1）检测某个class是否存在：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        .ft {
            font: italic bold 1em/1.5 'Courier New', Courier, monospace;
        }

        .bg {
            background: red;
        }

        .show {
            display: block;
        }

        .hidden {
            display: none;
        }
    </style>
</head>

<body>

    <div class="ft bg">HELLO WORLD</div>
    <button>click me</button>

    <script>

        "use strict";

        let divNode = document.querySelector("div");
        let btnNode = document.querySelector("button");

        btnNode.addEventListener("click", (event) => {
            console.log(divNode.classList.contains("show"));
            console.log(divNode.classList.contains("hidden"));
            console.log(divNode.classList.contains("bg"));
            console.log(divNode.classList.contains("ft"));
        });

        // false
        // false
        // true
        // true

    </script>

</body>


</html>
```

2）新增class：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        .ft {
            font: italic bold 1em/1.5 'Courier New', Courier, monospace;
        }

        .bg {
            background: red;
        }

        .show {
            display: block;
        }

        .hidden {
            display: none;
        }
    </style>
</head>

<body>

    <div class="ft">HELLO WORLD</div>
    <button>click me</button>

    <script>

        "use strict";

        let divNode = document.querySelector("div");
        let btnNode = document.querySelector("button");

        btnNode.addEventListener("click", (event) => {
            divNode.classList.add("bg");
        });

    </script>

</body>


</html>
```

3）删除class：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        .ft {
            font: italic bold 1em/1.5 'Courier New', Courier, monospace;
        }

        .bg {
            background: red;
        }

        .show {
            display: block;
        }

        .hidden {
            display: none;
        }
    </style>
</head>

<body>

    <div class="ft bg">HELLO WORLD</div>
    <button>click me</button>

    <script>

        "use strict";

        let divNode = document.querySelector("div");
        let btnNode = document.querySelector("button");

        btnNode.addEventListener("click", (event) => {
            divNode.classList.remove("bg");
        });

    </script>

</body>


</html>
```

4）如果class以存在，则删除，如果class不存在，则新增，相当于切换：

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        .ft {
            font: italic bold 1em/1.5 'Courier New', Courier, monospace;
        }

        .bg {
            background: red;
        }

        .show {
            display: block;
        }

        .hidden {
            display: none;
        }
    </style>
</head>

<body>

    <div class="ft bg show">HELLO WORLD</div>
    <button>click me</button>

    <script>

        "use strict";

        let divNode = document.querySelector("div");
        let btnNode = document.querySelector("button");

        btnNode.addEventListener("click", (event) => {
            divNode.classList.toggle("show");
            divNode.classList.toggle("hidden");
        });

    </script>

</body>


</html>
```



# 样式获取

## style

如果节点的样式是行内式的话，那么可以通过style属性单独的获取某一样式。

注意！仅针对行内式：

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

    <div style="color: red; font-size: 2rem; font-style: italic;">HELLO WORLD</div>

    <script>

        "use strict";

        let divNode = document.querySelector("div");

        console.log(divNode.style.color);
        console.log(divNode.style.fontSize);
        console.log(divNode.style.fontStyle);

        // red
        // 2rem
        // italic

    </script>

</body>


</html>
```



## getComputedStyle()

通过getComputedStyle()方法，获取节点元素身上所有以生效的样式。

这是计算后的样式属性，所以取得的单位和定义时的可能会有不同。

参数释义：

- elt：要获取样式的元素节点对象
- pseudoElt：获取的伪类样式

示例如下：

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

    <div style="color: red; font-size: 2rem; font-style: italic;">HELLO WORLD</div>

    <script>

        "use strict";

        let divNode = document.querySelector("div");

        console.log(getComputedStyle(divNode));

    </script>

</body>


</html>
```

可通过点语法获取单独的某个样式：

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

    <div style="color: red; font-size: 2rem; font-style: italic;">HELLO WORLD</div>

    <script>

        "use strict";

        let divNode = document.querySelector("div");

        console.log(getComputedStyle(divNode).color);
        console.log(getComputedStyle(divNode).fontSize);
        console.log(getComputedStyle(divNode).fontStyle);

        // rgb(255, 0, 0)
        // 32px
        // italic

    </script>

</body>


</html>
```

