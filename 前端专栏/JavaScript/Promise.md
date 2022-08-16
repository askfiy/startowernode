# 事件循环

## 基本介绍

JavaScript是一门单线程的编程语言，所以没有真正意义上的并行特性。

为了协调事件处理、页面交互、脚本调用、UI渲染、网络请求等行为对主线程造成的影响，事件循环（event loop）方案应运而生。

事件循环说白了就是一个不断的在等待任务、执行任务的方案。

在JavaScript中，根据执行方式的不同，有2种状态的任务，分别是同步任务和异步任务。

同步任务率先执行，而后执行异步任务，所有的异步任务由2个队列存储，分别是：

- 微任务队列
- 宏任务队列

主线程在执行完同步任务后，会不断的从这2个任务队列中按照先进先出的策略取出异步任务并执行。

并且在此期间也会有新的事件不断的加入至各个任务队列中，以此循环往复、永不阻塞。

如下图所示：

![image-20210811155721716](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210811155721716.png)







## 任务分类

宏任务包括：

- setInterval
- setTimeout
- setImmediate  Node.Js独有
- XHR callbackfn
- event callbackfn
- requestAnimationFrame
- UI rendering



微任务包括：

- Promise.then
- catch finally
- process.nextTick Node.Js独有
- MutationObserver





## 执行顺序

根据任务的状态，任务的执行优先级也会有所不同，具体执行顺序如下所示：

1. 同步任务（sync-task）
2. 微任务（micro-task）
3. 宏任务（macro-task）

而关于微任务和宏任务的执行，还有更详细的划分：

- 微任务队列中一旦有任务，将全部执行完成后再执行宏任务
- 宏任务队列中的任务在执行完成后，会检查微任务队列中是否有新添加的任务，如果有，那么将执行微任务队列中所有新添加的任务，如果没有则继续执行下一个宏任务

如下图所示：

![image-20210811161300458](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210811161300458.png)

代码测试：

```
"use strict";

// 宏任务，每5s添加一个微任务并执行
setInterval(() => {
    async function foo() {
        return "micro-task"
    }

    async function bar() {
        let result = await foo();
        console.log(result);
    }

    bar();

}, 5000);

// 宏任务，每1s执行一次
setInterval(() => { console.log("macro-task"); }, 1000);

// 同步任务
(() => {
    console.log("hello world");
})();
```

测试结果，虽然同步任务的代码在最下面，但是它会最先执行，而每添加一个微任务时，宏任务的执行会被插队：

![image-20210811163008045](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210811163008045.png)



# Promise

## 认识Promise

Promise是ES6中出现的新功能，用于在JavaScript中更加简单的实现异步编程。

我们可以使用new Promise()创建出一个Promise对象，它接收一个执行器函数，该函数需要指定resolve和reject参数用于改变当前Promise对象的执行状态。

由于Promise对象中执行器代码是属于同步任务，所以他会率先的进行执行，一个Promise对象拥有以下几种状态：

- fulfilled：任务完成、使用resolve改变了任务状态
- rejected：任务失败、使用reject改变了任务状态，或任务执行中抛出了异常
- pending：正在等待、未使用resolve或reject改变任务状态

注意，每个Promise对象的状态只允许改变一次！不可以多次更改。

![image-20210811185133685](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210811185133685.png)

示例如下。

1）Promise中执行器任务是同步任务，所以会率先执行：

```
"use strict";

setInterval(() => { console.log("macro task 3"); }, 1000)

let task = new Promise((resolve, reject) => {
    console.log("sync task 1");
});

console.log("sync task 2");

// sync task 1
// sync task 2
// macro task 3
```

2）使用resolve()方法改变Promise对象的状态为fulfilled：

```
"use strict";

let task = new Promise((resolve, reject) => {
    let x = Math.floor(Math.random() * 100) + 1;
    let y = Math.floor(Math.random() * 100) + 1;
    let result = x + y;

    // 返回结果为resolve()中的值
    resolve(result);
});


console.log(task);

// Promise {<fulfilled>: 83}
```

3）使用reject()方法改变Promise对象的状态为rejected， 它将引发一个异常：

```
"use strict";

let task = new Promise((resolve, reject) => {
    let x = Math.floor(Math.random() * 100) + 1;
    let y = Math.floor(Math.random() * 100) + 1;
    let result = x + y;

    // 返回结果为reject()中的值
    reject("error!")
});


console.log(task);

// Promise {<rejected>: "error!"}
// Uncaught (in promise) error!
```

4）如果未使用resolve()或reject()方法改变Promise对象状态，那么该任务的状态将为pending：

```
"use strict";

let task = new Promise((resolve, reject) => {
    let x = Math.floor(Math.random() * 100) + 1;
    let y = Math.floor(Math.random() * 100) + 1;
    let result = x + y;
});


console.log(task);

// Promise {<pending>}
```



## then()

我们可以在Promise对象后，添加一个用于处理任务状态的回调then()方法。

then()方法只有在Promise对象状态为fulfilled或者rejected时才会进行执行，它具有2个参数，接收2个回调函数：

- onfulfilled：Promise对象状态为fulfilled将执行该函数，具有1个参数value，接收Promise任务中resolve()所传递的值
- onrejected：Promise对象状态为rejected将执行该函数，具有1个参数reason，接收Promise任务中reject()或异常发生时所传递的值

此外，then()方法是属于微任务，所以他会插在宏任务之前进行执行。

![image-20210812154335003](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210812154335003.png)

代码示例如下：

1）Promise对象状态为fulfilled，运行then()方法的第1个回调函数：

```
"use strict";

let task = new Promise((resolve, reject) => {
    resolve("success");
}).then(
    value => {
        console.log(value);
    },
    reason => {
        console.log(reason);
    });


// success
```

2）Promise对象状态为rejected，运行then()方法的第2个回调函数：

```
"use strict";

let task = new Promise((resolve, reject) => {
    throw new Error("error");
}).then(
    value => {
        console.log(value);
    },
    reason => {
        console.log(reason);
    });


// error
```



## then()链式调用

其实每一个then()都将返回一个全新的Promise，默认情况下，该Promise的状态是fulfilled。

此时就会产生一种链式关系，每一个then()都将返回一个新的Promise对象，而每个then()的作用又都是处理上个Promise对象的状态。

这意味着我们可以无限的链式排列then()，如下所示：

![image-20210812154835989](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210812154835989.png)

代码示例：

```
"use strict";

let task = new Promise((resolve, reject) => {
    console.log("first Promise task status is fulfilled");
    resolve("success");
})
    .then(
        value => {
            console.log("then1 fulfilled", value);
            return value
        },
        reason => {
            console.log(reason);
        })
    .then(
        value => {
            console.log("then2 fulfilled", value);
            return value
        },
        reason => {
            console.log(reason);
        })
    .then(
        value => {
            console.log("then3 fulfilled", value);
            return value
        },
        reason => {
            console.log(reason);
        });

// first Promise task status is fulfilled
// then1 fulfilled success
// then2 fulfilled success
// then3 fulfilled success
```





## then()的返回值

要想真正的了解链式调用，就必须搞明白每个then()在不同状态下的返回值对下一个then()的影响。

具体情况如下所示：

1）当前then()无返回值，则当前Promise状态则为fulfilled。

下一个then()的onfulfilled回调函数参数value为undefined：

![image-20210812155012082](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210812155012082.png)

代码示例：

```
"use strict";

let task = new Promise((resolve, reject) => {
    console.log("first Promise task status is fulfilled");
    resolve("success");
})
    .then(
        value => {
            console.log("then1 fulfilled", value);  // success
        },
        reason => {
            console.log(reason);
        })
    .then(
        value => {
            console.log("then2 fulfilled", value);  // undefined
        },
        reason => {
            console.log(reason);
        });

// first Promise task status is fulfilled
// then1 fulfilled success
// then2 fulfilled undefined
```

2）当前then()有返回值，则当前Promise状态则为fulfilled。

下一个then()的onfulfilled回调函数参数value为当前then()的返回值：

![image-20210812155127867](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210812155127867.png)

代码示例：

```
let task = new Promise((resolve, reject) => {
    console.log("first Promise task status is fulfilled");
    resolve("success");
})
    .then(
        value => {
            console.log("then1 fulfilled", value);  // success
            return "then1 value"
        },
        reason => {
            console.log(reason);
        })
    .then(
        value => {
            console.log("then2 fulfilled", value);  // then1 value
        },
        reason => {
            console.log(reason);
        });

// first Promise task status is fulfilled
// then1 fulfilled success
// then2 fulfilled then1 value
```

3）当前then()有返回值，且返回了一个状态为fulfilled的Promise对象。

下一个then()的onfulfilled回调函数参数value为当前then()中被返回Promise里resolve()所传递的值：

![image-20210812155345653](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210812155345653.png)

代码示例：

```
"use strict";

let task = new Promise((resolve, reject) => {
    console.log("first Promise task status is fulfilled");
    resolve("success");
})
    .then(
        value => {
            console.log("then1 fulfilled", value);  // success
            return new Promise((resolve, reject) => {
                resolve("then1 Promise success")
            })
        },
        reason => {
            console.log(reason);
        })
    .then(
        value => {
            console.log("then2 fulfilled", value);  // then1 Promise success
        },
        reason => {
            console.log(reason);
        });

// first Promise task status is fulfilled
// then1 fulfilled success
// then2 fulfilled then1 Promise success
```

4）当前then()有返回值，且返回了一个状态为rejected的Promise对象。

下一个then()的onrejected回调函数参数reason为当前then()中被返回Promise里reject()所传递的值，或者是被返回Promise里抛出异常的值：

![image-20210812155520321](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210812155520321.png)

代码示例：

```
"use strict";

let task = new Promise((resolve, reject) => {
    console.log("first Promise task status is fulfilled");
    resolve("success");
})
    .then(
        value => {
            console.log("then1 fulfilled", value);  // success
            return new Promise((resolve, reject) => {
                reject("then1 Promise error")
            })
        },
        reason => {
            console.log(reason);
        })
    .then(
        value => {
            console.log("then2 fulfilled", value);
        },
        reason => {
            console.log(reason);  // then1 Promise error
        });

// first Promise task status is fulfilled
// then1 fulfilled success
// then1 Promise error
```

5）当前then()有返回值，且返回了一个状态为pending的Promise对象。下一个then()则必须等待当前then()中被返回Promise对象状态发生改变后才能继续执行：

![image-20210812155725841](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210812155725841.png)

代码示例：

```
"use strict";

let task = new Promise((resolve, reject) => {
    console.log("first Promise task status is fulfilled");
    resolve("success");
})
    .then(
        value => {
            console.log("then1 fulfilled", value);  // success
            return new Promise((resolve, reject) => {
                console.log("pending");
            })
        },
        reason => {
            console.log(reason);
        })
    .then(
        value => {
            console.log("then2 fulfilled", value);
        },
        reason => {
            console.log(reason);
        });

// first Promise task status is fulfilled
// then1 fulfilled success
// pending
```

另外，如果在代码执行时抛出了异常，那么返回的Promise对象状态则为rejected，下一个then()的onrejected回调函数参数reason为当前then()中抛出异常的值，这里不再进行演示。



## then()穿透

then()是具有穿透功能的，当一个then()没有指定需要被执行的回调函数时，它将继续冒泡向下传递：

![image-20210812155925418](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210812155925418.png)

代码示例：

```
"use strict";

let task = new Promise((resolve, reject) => {
    console.log("first Promise task status is fulfilled");
    resolve("success");
})
    .then()
    .then(
        value => {
            console.log("then2 fulfilled", value);
        },
        reason => {
            console.log(reason);
        });

// first Promise task status is fulfilled
// then2 fulfilled success
```





## catch()

每个then()都可以指定onrejected回调函数用于处理上一个Promise状态为rejected的情况。如果每个then()都进行这样的设置会显得很麻烦，所以我们只需要使用catch()即可。

catch()可以捕获之前所有Promise的错误执行，故建议将catch()放在最后。

catch()需要指定一个回调函数onrejected，具有1个参数reason，接收Promise任务中reject()或异常发生时所传递的值。

错误是冒泡传递的，如果没有任何一个then()定义onrejected的回调函数，那么错误将一直冒泡到catch()处进行处理：

![image-20210812160722513](https://images-1302522496.cos.ap-nanjing.myqcloud.com/img/image-20210812160722513.png)

代码示例：

```
"use strict";

let task = new Promise((resolve, reject) => {
    console.log("first Promise task status is fulfilled");
    resolve("success");
})
    .then(
        value => {
            console.log("then1 fulfilled", value);
            return value
        }
    )
    .then(
        value => {
            console.log("then2 rejected", value);
            throw new Error("error");
        })
    .then(
        value => {
            console.log("then3 ...", value);
        }
    )
    .catch(
        reason => {
            console.log(reason);
        });

// first Promise task status is fulfilled
// then1 fulfilled success
// then2 rejected success
// Error: error
```



## finally()

finally()是无论任务处理成功或者失败都会执行，因此建议将它放在链式调用的最后面。

它需要指定一个回调函数onfinally，该回调函数没有任何参数：

```
"use strict";

let task = new Promise((resolve, reject) => {
    console.log("first Promise task status is fulfilled");
    resolve("success");
})
    .then(
        value => {
            console.log("then1 fulfilled", value);
            return value
        }
    )
    .catch(
        reason => {
            console.log(reason);
        }
    )
    .finally(
        () => {
            console.log("run");
        }
    );

// first Promise task status is fulfilled
// then1 fulfilled success
// run
```



# 扩展方法

## resolve()

resolve()方法用于快速返回一个状态为fulfilled的Promise对象，生产环境中使用较少：

```
"use strict";

let task = Promise.resolve("success");

console.log(task);

// Promise {<fulfilled>: "success"}
```



## reject()

reject()方法用于快速返回一个状态为rejected的Promise对象，生产环境中使用较少：

```
"use strict";

let task = Promise.reject("error");

console.log(task);

// Promise {<rejected>: "error"}
// Uncaught (in promise) error
```





## all()

all()方法用于一次同时执行多个异步任务，并且必须确保这些任务是成功的。

- all()方法接收的参数必须是可迭代类型，如Array、map、set
- 任何一个Promise状态为rejected，都将调用catch()方法
- 当所有任务成功执行后，将返回一个包含所有任务的执行结果数组

all()方法应用场景还是非常广泛的，如我们需要使用Ajax请求后端的书籍与价格信息时，不论是书籍获取失败还是价格获取失败，都将认为此次任务的失败。

示例如下：

```
"use strict";

const getBookNameTask = new Promise((resolve, reject) => {
    // 模拟请求后端的书籍名称，需要花费3s
    setTimeout(() => {
        resolve(JSON.stringify(
            ["HTML", "CSS", "JavaScript"]
        ))
    }, 3000);
});

const getBookPriceTask = new Promise((resolve, reject) => {
    // 模拟请求后端的书籍价格，需要花费5s
    setTimeout(() => {
        resolve(JSON.stringify(
            [98, 120, 40]
        ))
    }, 5000);
})

// 执行任务
Promise.all(
    [getBookNameTask, getBookPriceTask]
)
    .then(value => {
        // 书籍和价格全部获取后才执行这里
        // value = ["[\"HTML\",\"CSS\",\"JavaScript\"]", "[98,120,40]"]
        const bookNameArray = JSON.parse(value[0]);
        const bookPriceArray = JSON.parse(value[1]);
        const bookAndNameMap = new Map();
        for (let i = 0; i < bookNameArray.length; i++) {
            bookAndNameMap.set(bookNameArray[i], bookPriceArray[i]);
        }
        console.log(bookAndNameMap);
    })
    .catch(reason => {
        // 任何一个没获取到都执行这里
        console.log(reason);
    });
```





## allSettled()

allSettled()方法和all()方法相似，都是用于同时执行多个异步任务，但是它并不关心所有任务是否都执行成功。

allSettled()的状态只会是fulfilled：

```
"use strict";

const getBookNameTask = new Promise((resolve, reject) => {
    setTimeout(() => {
        reject("error! Can't query all books name")
    }, 3000);
});

const getBookPriceTask = new Promise((resolve, reject) => {
    setTimeout(() => {
        reject("error! Can't query all books price")
    }, 5000);
})

// 执行任务
Promise.allSettled(
    [getBookNameTask, getBookPriceTask]
)
    .then(value => {
        // 不管怎样都会执行这里
        console.log("run me");
    })
```



## race()

race()也可同时执行多个任务，它仅会返回最快完成任务的执行结果。

- 以最快返回的任务结果为准
- 如果最快返回的任务状态为rejected，那么race()的状态也将视为rejected，此时将执行catch()方法

race()方法用的也比较多，如我们需要加载一些图片，这些图片在多个服务端上都有存储，但为了提高用户体验我们需要根据用户所在的地理位置选择最近的服务器，此时race()就派上了用场：

```
"use strict";

const getCacheImages = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve("get cache images success!!");
    }, 1000);
})

const getWebImages = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve("get web images success!!");
    }, 3000);
})

// 创建任务
Promise.race(
    [getCacheImages, getWebImages]
)
    .then(value => {
        console.log(value);
    })
    .catch(reason => {
        console.log(reason);
    })

// get cache images success!!
```





# async&await

## async

async其实是new Promise()的语法糖简写形式。

在某一个函数前面加上async，运行该函数时将会返回一个Promise对象。

- 没有return：返回的Promise对象状态为fulfilled，下一个then()的onfulfilled回调函数参数value为undefined
- 直接return：返回的Promise对象状态为fulfilled，下一个then()的onfulfilled回调函数参数value为当前async函数的返回值
- return了一个状态为fulfilled的Promise对象：下一个then()的onfulfilled回调函数参数value为当前async函数中被返回Promise里resolve()所传递的值
- return了一个状态为rejected的Promise对象：下一个then()的onrejected回调函数参数reason为当前async函数中被返回Promise里reject()所传递的值，或者是被返回Promise里抛出异常的值
- 运行时抛出异常：返回的Promise对象状态为rejected，下一个then()的onrejected回调函数参数reason为当前async函数中抛出异常的值

示例演示：

```
"use strict";

async function task() {
    return "success"
}

task().then(value => {
    console.log(value);
});

// success
```



## await

await其实是then()的另一种写法，它只能在async函数中使用。

- await后面一般都会跟上一个Promise对象，如果不是Promise对象，将直接返回该值。
- await使用必须在async函数中
- await作为then()的语法糖形式，使用它编写代码将使代码变的更加优雅

如下所示，我们有3个任务，这3个任务必须是先通过用户ID获取人员姓名、再通过用户ID获取信息ID、最后再通过用户ID获取人员信息。

如果你用纯Promise+then()的方式进行代码编写，它将是这样的：

```
"use strict";

const idAndName = new Map([
    [1, "Jack"],
    [2, "Tom"],
    [3, "Mary"],
]);

const personnelInformation = new Map([
    [1, { gender: "female", age: 18, addr: "TianJin", desc: "my name is Mary" }],
    [2, { gender: "male", age: 21, addr: "ShangHai", desc: "my name is Tom" }],
    [3, { gender: "male", age: 18, addr: "BeiJing", desc: "my name is Jack" }],
]);

const nameAndMessage = new Map([
    [1, 3],
    [2, 2],
    [3, 1],
])



function getUserMessage(id) {
    let userName, messageId, message, str;

    new Promise((resolve, reject) => {
        // 获取姓名
        if (idAndName.has(id)) {
            userName = idAndName.get(id);
            resolve();
        }
        reject(`no information id : ${id}`);
    })
        .then(() => {
            // 获取关系
            messageId = nameAndMessage.get(id);
        })
        .then(() => {
            // 获取信息
            message = personnelInformation.get(messageId);
        })
        .then(() => {
            // 进行渲染
            str = `name : ${userName}</br>`;
            for (let [k, v] of Object.entries(message)) {
                str += `${k} : ${v}</br>`;
            }
            document.write(str)
        })
        .catch(reason => {
            document.write(`<p style="color:red">${reason}</p>`);
        })
}

getUserMessage(3);
```

如果你使用async+awit的方式编写，那么它的逻辑就会清楚很多：

```
"use strict";

const idAndName = new Map([
    [1, "Jack"],
    [2, "Tom"],
    [3, "Mary"],
]);

const personnelInformation = new Map([
    [1, { gender: "female", age: 18, addr: "TianJin", desc: "my name is Mary" }],
    [2, { gender: "male", age: 21, addr: "ShangHai", desc: "my name is Tom" }],
    [3, { gender: "male", age: 18, addr: "BeiJing", desc: "my name is Jack" }],
]);

const nameAndMessage = new Map([
    [1, 3],
    [2, 2],
    [3, 1],
])

// 获取姓名
async function getName(id) {
    if (idAndName.has(id)) {
        return idAndName.get(id);
    }
    throw new Error(`no information id : ${id}`);
}

// 获取关系
async function getRelation(id) {
    return nameAndMessage.get(id);
}

// 获取信息
async function getMessage(messageId) {
    return personnelInformation.get(messageId);
}

// 入口函数，进行渲染
async function getUserMessage(id) {
    try {
        let userName = await getName(id);  // 必须等待该函数执行完成才会继续向下执行
        let messageId = await getRelation(id);
        let message = await getMessage(messageId);
        let str = `name : ${userName}</br>`;

        for (let [k, v] of Object.entries(message)) {
            str += `${k} : ${v}</br>`;
        }

        document.write(str)
    } catch (e) {
        document.write(`<p style="color:red">${e}</p>`);
    }

}


getUserMessage(3);
```



## 异常处理

async+await的异常处理推荐使用try+catch语句将所有执行代码进行包裹，它将处理所有可能出现的异常，相当于在链式调用的最后面加上catch()方法：

```
"use strict";

async function task01() {
    console.log("run task 01");
}

async function task02() {
    throw new Error("task02 error");
    console.log("run task 02");
}

async function task03() {
    console.log("run task 03");
}

async function main() {
    try {
        await task01();
        await task02();
        await task03();
    } catch (e) {
        console.log(e);
    }
}

main();
```

也可以在主函数外部使用catch()方法来处理异常，但是我并不推荐这么做。

```
"use strict";

async function task01() {
    console.log("run task 01");
}

async function task02() {
    throw new Error("task02 error");
    console.log("run task 02");
}

async function task03() {
    console.log("run task 03");
}

async function main() {
    await task01();
    await task02();
    await task03();
}

main().catch(reason => {
    console.log(reason);
});
```

除此之外，你也可以使用try+catch语句块对单独的async函数语句块进行处理，预防可能出现的异常。

