# 基本介绍

JavaScript中提供了Date()对象，它用于对时间的管控。

Date()对象所包含的方法众多，下表将介绍一些常用的方法：

| 方法                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| Date()               | 返回本地化当前完整日期的Date对象                             |
| getFullYear()        | 根据本地时间从Date对象中以四位数字格式返回年份               |
| getMonth()           | 根据本地时间从Date对象中返回一年中的某一月（1 ~ 11）         |
| getDate()            | 根据本地时间从Date对象中返回一月中的某一天（1 ~ 31）         |
| getDay()             | 根据本地时间从Date对象中返回一周中的某一天（1 ~ 6）          |
| getHours()           | 根据本地时间从Date对象中返回当前天数中的的小时 (0 ~ 23)      |
| getMinutes()         | 根据本地时间从Date对象中返回当前小时中的分钟数 (0 ~ 59)      |
| getSeconds()         | 根据本地时间从Date对象中返回当前分钟数中的秒数 (0 ~ 59)      |
| getMilliseconds()    | 根据本地时间从Date对象中返回当前秒数中的的毫秒数 (0 ~ 999)   |
| setFullYear()        | 根据本地时设置Date对象中的年份（四位数字）                   |
| setMonth()           | 根据本地时设置Date对象中月份 (0 ~ 11)                        |
| setDate()            | 根据本地时设置Date对象中月的某一天 (1 ~ 31)                  |
| setHours()           | 根据本地时设置Date对象中的小时 (0 ~ 23)                      |
| setMinutes()         | 根据本地时设置Date对象中的分钟 (0 ~ 59)                      |
| setSeconds()         | 根据本地时设置 Date 对象中的秒钟 (0 ~ 59)                    |
| setMilliseconds()    | 根据本地时设置 Date 对象中的毫秒 (0 ~ 999)                   |
| getUTCFullYear()     | 根据世界时间从Date对象中以四位数字格式返回年份               |
| getUTCMonth()        | 根据世界时间从Date对象中返回一年中的某一月（1 ~ 11）         |
| getUTCDate()         | 根据世界时间从Date对象中返回一月中的某一天（1 ~ 31）         |
| getUTCDay()          | 根据世界时间从Date对象中返回一周中的某一天（0 ~ 6）          |
| getUTCHours()        | 根据世界时间从Date对象中返回当前天数中的的小时 (0 ~ 23)      |
| getUTCMinutes()      | 根据世界时间从Date对象中返回当前小时中的分钟数 (0 ~ 59)      |
| getUTCSeconds()      | 根据世界时间从Date对象中返回当前分钟数中的秒数 (0 ~ 59)      |
| getUTCMilliseconds() | 根据世界时间从Date对象中返回当前秒数中的的毫秒数 (0 ~ 999)   |
| setUTCFullYear()     | 根据世界时设置Date对象中的年份（四位数字）                   |
| setUTCMonth()        | 根据世界时设置Date对象中的月份 (0 ~ 11)                      |
| setUTCDate()         | 根据世界时设置Date对象中月份的一天 (1 ~ 31)                  |
| setUTCHours()        | 根据世界时设置Date对象中的小时 (0 ~ 23)                      |
| setUTCMinutes()      | 根据世界时设置Date对象中的分钟 (0 ~ 59)                      |
| setUTCSeconds()      | 根据世界时设置Date对象中的秒钟 (0 ~ 59)                      |
| setUTCMilliseconds() | 根据世界时设置Date对象中的毫秒 (0 ~ 999)                     |
| toISOString          | 以ISO格式返回一个字符串值的日期                              |
| toLocaleString()     | 根据本地时间格式，把Date对象转换为字符串                     |
| toUCTString()        | 根据世界时间格式，把Date对象转换为字符串                     |
| toJson()             | 将时间对象格式化为JSON字符串                                 |
| toDateString()       | 根据世界时间格式，把Date对象的日期部分转换为字符串           |
| toLocalDateString()  | 根据本地时间格式，把Date对象的日期部分转换为字符串           |
| toTimeString()       | 根据世界时间格式，把Date对象的时间部分转换为字符串           |
| toLocalTimeString()  | 根据本地时间格式，把Date对象的时间部分转换为字符串           |
| getTime()            | 根据本地时返回 1970 年 1 月 1 日至今的毫秒数，注意它返回的是Number类型 |
| Date.parse()         | 返回 1970 年 1 月 1 日午夜到指定日期的毫秒数，注意它返回的是字符串类型 |
| Date.UTC()           | 根据世界时返回 1970 年 1 月 1 日到指定日期的毫秒数，注意它返回的是字符串类型 |
| setTime()            | 根据本地时以毫秒级别设置Date对象                             |
| getTimezoneOffset()  | 返回本地时间与格林威治标准时间 (GMT) 的分钟差                |
| toString()           | 返回Date对象的字符表示值                                     |
| valueOf()            | 返回Date对象的原始表示值                                     |





# 对象声明

以下例举最常见的使用方式：

```
"use strict";

// 获取当前日期和时间，返回一个时间对象：Fri Jul 30 2021 20:18:01 GMT+0800 (中国标准时间)
console.log(new Date());

// 也可以直接使用Date()，返回的是一个string类型，不能调用时间对象的方法：Fri Jul 30 2021 20:18:01 GMT+0800 (中国标准时间)
console.log(Date());

// 获取本地时间戳：1627647554707
// 1970年1月1日凌晨8点0分0秒开始距离现在所经历过的时间，单位是毫秒
console.log(new Date() * 1);

// 获取本地时间戳：1627647554707
console.log(Date.now());

// 填入参数，获取时间：Wed May 23 2018 12:00:12 GMT+0800 (中国标准时间)
console.log(new Date(2018, 4, 23, 12, 0, 12, 13));
```





# 时间统计

最通用的方法，以时间戳的毫秒为单位，结束时间减去开始时间：

```
"use strict";

const sTime = Date.now();

for (let i = 0; i < 2000000; i++) { }

const eTime = Date.now();

const totalRunTime = eTime - sTime;

console.log(totalRunTime);

// 5
```

JavaScript独有的console系列方法，适用于控制台打印，单位也是毫秒：

```
"use strict";

console.time("Statistics Time")

for (let i = 0; i < 2000000; i++) { }

console.timeEnd("Statistics Time")

// tatistics Time: 4.534ms
```





# 时间转换

以下是将时间对象转换为时间戳格式：

```
"use strict";

const time = new Date();

// 可以与Number类型做运算得到时间戳
console.log(time * 1); 
console.log(Number(time));

// 可以调用时间对象内部提供的方法
console.log(time.valueOf());
console.log(time.getTime());

// 1627648373167
```

如果后端提供的时间戳格式，我们前端需要为它进行手动转换，如下所示，使用requestTime()函数模拟请求：

```
"use strict";

function requestTime() {
    return new Date(1990, 2, 22, 13, 22, 19).getTime();
}

const timestamp = requestTime();
console.log(new Date(timestamp));

// Thu Mar 22 1990 13:22:19 GMT+0800 (中国标准时间)
```





# 时间计算

以下是时间计算的相关演示：

```
"use strict";

// 求5天后的3小时22分的时间
const time = new Date(1990, 2, 22, 13, 22, 19);

// 5天后的时间
time.setDate(time.getDate() + 5);
// 3小时
time.setHours(time.getHours() + 3);
// 22分
time.setMinutes(time.getMinutes() + 22);

// 最终结果
console.log(time);
// Tue Mar 27 1990 16:44:19 GMT+0800 (中国标准时间)
```





# 时间格式化

有时候我们会觉得JavaScript提供给我们的格式不太好用，这个时候可以写一个函数自定义格式：

```
"use strict";

function dateFormat(date, format = "YYYY-MM-DD HH:mm:ss") {

    const config = {
        YYYY: date.getFullYear(),  // 获取年份
        MM: date.getMonth() + 1,   // 获取月份，月份+1是因为Js中的月份是0-11
        DD: date.getDate(),        // 获取天数
        HH: date.getHours(),       // 获取小时
        mm: date.getMinutes(),     // 获取分
        ss: date.getSeconds()      // 获取秒
    };
    for (const key in config) {
        format = format.replace(key, config[key]);
    }
    return format;
}

console.log(dateFormat(new Date(), "YYYY年MM月DD日"));

// 2020年7月28日
```





# moment.js

Moment.js是一个轻量级的JavaScript时间库，它方便了日常开发中对时间的操作，提高了开发效率。

更多使用方法请访问 [中文官网](http://momentjs.cn/ )或[英文官网](https://momentjs.com/)。

在线引入：

```
<script src="https://cdn.bootcss.com/moment.js/2.24.0/moment.min.js"></script>
```

1）获取当前时间：

```
<script src="https://cdn.bootcss.com/moment.js/2.24.0/moment.min.js"></script>
<script>

    "use strict";
    
    console.log(moment().format("YYYY-MM-DD HH:mm:ss"));
    
    // 2021-07-30 20:45:58
    
</script>
```

2）时间计算：

```
<script>

    "use strict";

    console.log(moment().add(10, "days").format("YYYY-MM-DD hh:mm:ss"));

    // 2021-08-09 08:47:45

</script>
```

3）格式化时间：

```
<script src="https://cdn.bootcss.com/moment.js/2.24.0/moment.min.js"></script>
<script>

    "use strict";
    
    console.log(moment().format("YYYY-MM-DD HH:mm:ss"));
    
    // 2021-07-30 20:45:58
    
</script>
```

