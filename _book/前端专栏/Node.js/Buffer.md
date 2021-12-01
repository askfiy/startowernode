# 基本介绍

我们都知道，如果数据想在网络中进行传播，那么必须传输二进制字节流。但遗憾的是JavaScript中并没有实现类似的直观数据类型。

所幸，Node.js中支持了Buffer格式、Buffer类似于Python中的bytes类型、Go中的byte类型，它能够作用于数据网络传输和文件系统操作等场景。

我们都知道1个Byte等于8个bit，而Buffer则等于256个Byte。

Buffer类的实例实际上是一个0-255之间的数组，它是一个由JavaScript和C++结合的模块，其对象内存不经过V8引擎分配，而是通过C++进行申请、JavaScript 分配。缓冲区的大小在创建时确定，不能调整。

因Buffer对象实在过于常用，故被直接内置到全局变量中，使用时候无需 require 引入。



# 创建Buffer

## Buffer.form(string , encoding)

返回一个包含给定字符串的Bufferr：

```
let bf: Buffer = Buffer.from("hello world", "utf-8");
console.log(bf);

// <Buffer 68 65 6c 6c 6f 20 77 6f 72 6c 64>
```

Buffer目前支持以下几种编码格式

- ascii
- utf8

- utf16le
- base64

- binary
- hex



## Buffer.form(array)

返回一个内容包含所提供的字节副本的Buffer，数组中每一项是一个表示八位字节的数字，所以值必须在0 ~ 255之间，否则会取模：

```
let bf: Buffer = Buffer.from([256, 2444, 3]);
console.log(bf);

// 超过 255 会取模，相当于 number % 256
// <Buffer 00 8c 03>
```



## Buffer.form(object)

取object的valueOf()或Symbol.toPrimitive()初始化Buffer：

```
let bf: Buffer = Buffer.from({
    valueOf() {
        return "hello world";
    }
});    // 自动调用对象的valueOf()方法

console.log(bf);

// <Buffer 68 65 6c 6c 6f 20 77 6f 72 6c 64>
```



## Buffer.form(buffer)

返回一个给定Buufer的副本Buffer：

```
let firstBf: Buffer = Buffer.from("hello world", "utf-8");
let lastBf : Buffer = Buffer.from(firstBf);

console.log(lastBf);

// <Buffer 68 65 6c 6c 6f 20 77 6f 72 6c 64>
```





# 转换为字符串

字符串转Buffer需要使用Buffer.form()方法，而Buffer转字符串只需要调用其下的toString()方法即可：

```
let bf: Buffer = Buffer.from("hello world", "utf-8");
console.log(bf);

let str = bf.toString();
console.log(str);

// <Buffer 68 65 6c 6c 6f 20 77 6f 72 6c 64>
// hello world
```



# Buffer拼接

使用Buffer.concat()方法将多个Buffer进行拼接：

```
let firstBf: Buffer = Buffer.from("hello", "utf-8");
let lastBf: Buffer = Buffer.from("world", "utf-8");

console.log(Buffer.concat([firstBf, lastBf]));
```



# 常用方法

## isBuffer()

判断一个对象是否是Buffer类型：

```
console.log(Buffer.isBuffer("1"));

// false
```



## isEncoding()

判断Buffer对象的默认编码格式：

```
console.log(Buffer.isEncoding("utf-8"));

// true
```



## length

获取Buffer对象的长度：

```
let bf: Buffer = Buffer.from("hello world", "utf-8");

console.log(bf.length);

// 11
```



## indexOf

获取某字符在Buffer中的位置，如果不存在则返回-1：

```
let bf: Buffer = Buffer.from("hello world", "utf-8");

console.log(bf.indexOf("w"));

// 6
```

