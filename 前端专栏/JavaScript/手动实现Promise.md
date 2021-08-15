# Promise状态实现

在原生Promise中具有三种状态，分别是

- pending：未解决状态

- fulfilled：已解决状态

- rejected：解决失败状态

所以第一步，要先实现这三种状态。

并且在原生Promise中具有value用于记录执行函数resolve()与reject()中的值用于传递给then()方法。

所以我们要定义一个value：

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {
        this.status = AsyncPromise.PENDING;  // 初始为准备状态
        this.value = null;  // 初始值
    }

}
```



# Promise执行函数

原生Promise的构造函数中会接收一个executor参数，该参数当是一个函数。

用于同步的执行当前任务，当任务完成后应该具有resolve()方法以及reject()方法来通知then()方法当前执行任务的执行状态并传递值。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {
        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值

        executor(this.resolve, this.reject);  // 传递形参，运行executor函数
    }

    resolve(value) {

        this.status = AsyncPromise.FULFILLED;
        this.value = value;
    }

    reject(reason) {

        this.status = AsyncPromise.REJECTED;
        this.value = reason;

    }
}
```

上面这样写在执行resolve()以及reject()时会出现问题，原因是this指向为undefiled(严格模式)。

```
<script src="./Promise核心.js"></script>
<script>

    "use strict";

    let task = new AsyncPromise((resolve, reject) => {
        resolve("成功")
    })

    console.log(task);

    // undefined

</script>
```

这是由于我们在执行函数中调用了resolve()与reject()，故this指向为executor的函数上下文。

解决这个问题可以使用bind()来改变this指向。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {

        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值
        executor(this.resolve.bind(this), this.reject.bind(this));  // 传递形参，运行executor函数
    }

    resolve(value) {

        this.status = AsyncPromise.FULFILLED;
        this.value = value;
    }

    reject(reason) {

        this.status = AsyncPromise.REJECTED;
        this.value = reason;

    }
}
```



# Promise状态限制

当前Promise状态只应该改变一次而不能多次改变，显然我们上面的代码不能做到这点限制。

```
<script src="./Promise核心.js"></script>
<script>

    "use strict";

    let task = new AsyncPromise((resolve, reject) => {

        resolve("成功");
        reject("失败");

        // 对于原生Promise来说，状态只能改变一次。但是这里却允许两次改变，故是有问题的
    })

    console.log(task);  // AsyncPromise {status: "rejected", value: "失败"}

</script>
```

所以这里要对代码加上限制。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {

        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值
        executor(this.resolve.bind(this), this.reject.bind(this));  // 传递形参，运行executor函数
    }

    resolve(value) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.FULFILLED;
            this.value = value;
        }
    }

    reject(reason) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.REJECTED;
            this.value = reason;

        }
    }
}
```



# Promise执行异常

在执行函数executor中可能引发异常，这会让当前的Promise的状态改变为rejected。

所以在上面代码基础上需要加入try/catch进行处理。

当then()方法捕捉到执行函数executor中的异常时，可以让第二个参数的函数对其异常进行处理，但是我们目前还没实现then()，所以直接丢给reject()即可，当实现then()时自然会使用到reject()中传递过来的值。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {

        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值

        try {
            executor(this.resolve.bind(this), this.reject.bind(this));  // 传递形参，运行executor函数
        } catch (e) {
            this.status = AsyncPromise.REJECTED; // 异常发生改变状态
            this.reject(e); // 记录异常信息
        }


    }

    resolve(value) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.FULFILLED;
            this.value = value;
        }
    }

    reject(reason) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.REJECTED;
            this.value = reason;

        }
    }
}
```



# then方法基础实现

原生的Promise状态改变后，可以执行其下的then()方法，所以我们需要来封装出一个then()方法。

then()方法接收两个函数，第一个函数onfulfilled用于处理上一个Promise的fulfilled状态，第二个函数onrejected用于处理上一个Promise的rejected状态。

并且then()方法中的这两个函数都应该具有一个形参，用于接收到Promise的resolve()或reject()中传递的值。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {

        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值

        try {
            executor(this.resolve.bind(this), this.reject.bind(this));  // 传递形参，运行executor函数
        } catch (e) {
            this.status = AsyncPromise.REJECTED; // 异常发生改变状态
            this.reject(e); // 记录异常信息
        }


    }

    resolve(value) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.FULFILLED;
            this.value = value;
        }
    }

    reject(reason) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.REJECTED;
            this.value = reason;

        }
    }

    then(onfulfilled, onrejected) {

        if (this.status == AsyncPromise.FULFILLED) {  // 状态改变时执行
            onfulfilled(this.value);
        }

        if (this.status == AsyncPromise.REJECTED) {  // 状态改变时执行
            onrejected(this.value);
        }
    }
}
```



# then方法参数优化

上面已经说过，then()方法具有两个参数，这两个参数分别对应两个函数用来处理上一个Promsie的resolve()与reject()。

但是在原生Promise中，这两个方法可以不进行传递，所以我们需要对上述代码进行优化。

当then()方法中的某一个参数不为函数时，让它自动创建一个空函数。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {

        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值

        try {
            executor(this.resolve.bind(this), this.reject.bind(this));  // 传递形参，运行executor函数
        } catch (e) {
            this.status = AsyncPromise.REJECTED; // 异常发生改变状态
            this.reject(e); // 记录异常信息
        }


    }

    resolve(value) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.FULFILLED;
            this.value = value;
        }
    }

    reject(reason) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.REJECTED;
            this.value = reason;

        }
    }

    then(onfulfilled, onrejected) {
        if (typeof onfulfilled != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onfulfilled = () => { };
        }

        if (typeof onrejected != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onrejected = () => { };
        }

        if (this.status == AsyncPromise.FULFILLED) {  // 状态改变时执行
            onfulfilled(this.value);
        }

        if (this.status == AsyncPromise.REJECTED) {  // 状态改变时执行
            onrejected(this.value);
        }
    }
}
```



# then方法异常捕获

当then()方法中处理fulfilled状态的函数onfulfilled或者处理rejected状态的函数onrejected在运行时出现异常应该进行捕获并且传递给下一个then()的处理rejected状态的函数onrejected。

这里我们先让所有的异常都交由当前then()处理rejected状态的函数onrejected，后面再进行优化。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {

        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值

        try {
            executor(this.resolve.bind(this), this.reject.bind(this));  // 传递形参，运行executor函数
        } catch (e) {
            this.status = AsyncPromise.REJECTED; // 异常发生改变状态
            this.reject(e); // 记录异常信息
        }


    }

    resolve(value) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.FULFILLED;
            this.value = value;
        }
    }

    reject(reason) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.REJECTED;
            this.value = reason;
        }
    }

    then(onfulfilled, onrejected) {
        if (typeof onfulfilled != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onfulfilled = () => { };
        }

        if (typeof onrejected != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onrejected = () => { };
        }

        if (this.status == AsyncPromise.FULFILLED) {   // 状态改变时执行

            try {  // then处理成功的函数onfulfilled中出现异常，交由当前then处理失败的函数onrejected函数进行处理。这个后面会做优化
                onfulfilled(this.value);
            } catch (e) {
                onrejected(e);
            }

        }

        if (this.status == AsyncPromise.REJECTED) {   // 状态改变时执行
            try {  // then处理失败的函数onrejected中出现异常,交由当前then处理失败的函数onrejected函数进行处理。这个后面会做优化
                onrejected(this.value);
            } catch (e) {
                onrejected(e);
            }
        }
    }
}
```



# then方法异步执行

在原生的Promise中，executor函数是同步执行的，而then()方法是异步执行故排在同步执行之后。

但是我们的Promise却没有做到这一点，下面的实验将说明这个问题

```
<script src="./Promise核心.js"></script>
<script>

    "use strict";

    let task = new AsyncPromise((resolve, reject) => {
        reject("失败");
    }).then(
        null,
        error => {
            console.log(error);  // 先打印  失败
        }
    )

    console.log("hello,Promise");  // 后打印  hello,Promise

</script>
```

最简单的解决方案就是为then()中处理成功或处理失败的函数运行外套上一个setTimeout，让其处理排在线程同步任务执行之后再进行执行。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {

        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值

        try {
            executor(this.resolve.bind(this), this.reject.bind(this));  // 传递形参，运行executor函数
        } catch (e) {
            this.status = AsyncPromise.REJECTED; // 异常发生改变状态
            this.reject(e); // 记录异常信息
        }


    }

    resolve(value) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.FULFILLED;
            this.value = value;
        }
    }

    reject(reason) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.REJECTED;
            this.value = reason;

        }
    }

    then(onfulfilled, onrejected) {
        if (typeof onfulfilled != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onfulfilled = () => { };
        }

        if (typeof onrejected != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onrejected = () => { };
        }

        if (this.status == AsyncPromise.FULFILLED) {   // 状态改变时执行

            setTimeout(() => {  // 晚于线程同步任务执行
                try {  // then处理成功的函数onfulfilled中出现异常，交由当前then处理失败的函数onrejected函数进行处理。这个后面会做优化
                    onfulfilled(this.value);
                } catch (e) {
                    onrejected(e);
                }
            })
        }

        if (this.status == AsyncPromise.REJECTED) {   // 状态改变时执行

            setTimeout(() => {  // 晚于线程同步任务执行
                try {  // then处理失败的函数onrejected中出现异常,交由当前then处理失败的函数onrejected函数进行处理。这个后面会做优化
                    onrejected(this.value);
                } catch (e) {
                    onrejected(e);
                }
            })
        }
    }
}
```



# 执行函数异步阻塞

此时我们的代码仍然具有一个问题，即在执行函数executor中使用setTimeout时，下面的then()会进行阻塞。

这是因为当前Promise状态是pending而then()方法中并没有对pending状态进行处理的策略所导致的。

```
<script src="./Promise核心.js"></script>
<script>


    "use strict";

    new AsyncPromise((resolve, reject) => {
        setTimeout(() => {
            resolve("成功");  // 同步任务执行完三秒后才会改变当前Promise状态
        }, 3000);

    }).then((success) => {  // 但是这里先执行了then，Promise状态为pending，故发生阻塞
        console.log(success);  // 阻塞了，不打印
    })

</script>
```

既然当前Promise状态是pending，3秒后状态才发生改变，那么我们就可以通过不断的循环来看看它何时改变状态。

所以第一步是定义一个执行异步的数组。然后再将then()中处理正确的函数onfulfilled与处理错误的函数onrejected压进去。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {

        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值

        this.callbacks = []; // 如果是一个异步操作，则放入该数组中

        try {
            executor(this.resolve.bind(this), this.reject.bind(this));  // 传递形参，运行executor函数
        } catch (e) {
            this.status = AsyncPromise.REJECTED; // 异常发生改变状态
            this.reject(e); // 记录异常信息
        }


    }

    resolve(value) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.FULFILLED;
            this.value = value;
        })
    }
}

reject(reason) {

    if (this.status == AsyncPromise.PENDING) {  // 限制
        this.status = AsyncPromise.REJECTED;
        this.value = reason;
    }
}

then(onfulfilled, onrejected) {

    if (typeof onfulfilled != "function") {  // 如果传入的不是一个函数，默认创建空函数
        onfulfilled = () => { };
    }

    if (typeof onrejected != "function") {  // 如果传入的不是一个函数，默认创建空函数
        onrejected = () => { };
    }

    if (this.status == AsyncPromise.FULFILLED) {   // 状态改变时执行

        setTimeout(() => {  // 晚于线程同步任务执行
            try {   // then处理成功的函数onfulfilled中出现异常，交由当前then处理失败的函数onrejected函数进行处理。这个后面会做优化
                onfulfilled(this.value);
            } catch (e) {
                onrejected(e);
            }
        })
    }

    if (this.status == AsyncPromise.REJECTED) {   // 状态改变时执行

        setTimeout(() => {  // 晚于线程同步任务执行
            try {  // then处理失败的函数onrejected中出现异常,交由当前then处理失败的函数onrejected函数进行处理。这个后面会做优化
                onrejected(this.value);
            } catch (e) {
                onrejected(e);
            }
        })
    }

    if (this.status == AsyncPromise.PENDING) {  // 如果当前Promise是等待处理状态，则将处理成功的函数与处理失败的函数压入异步数组。
        this.callbacks.push({
            onfulfilled,
            onrejected,
        });
    }
}
}
```

当数组压入完成后，执行函数executor会去调用resolve()或者reject()改变当前Promise状态。

所以我们还需要在resolve()与reject()方法中对异步的数组处理函数进行调用。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {

        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值

        this.callbacks = []; // 如果是一个异步操作，则放入该数组中

        try {
            executor(this.resolve.bind(this), this.reject.bind(this));  // 传递形参，运行executor函数
        } catch (e) {
            this.status = AsyncPromise.REJECTED; // 异常发生改变状态
            this.reject(e); // 记录异常信息
        }


    }

    resolve(value) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.FULFILLED;
            this.value = value;

            this.callbacks.map(callback => {  // // 调用处理异步executor里resolve的方法。
                callback.onfulfilled(value);
            })
        }
    }

    reject(reason) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.REJECTED;
            this.value = reason;

            this.callbacks.map(callback => {  // 调用处理异步executor里reject的方法。
                callback.onrejected(reason);
            })

        }
    }

    then(onfulfilled, onrejected) {

        if (typeof onfulfilled != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onfulfilled = () => { };
        }

        if (typeof onrejected != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onrejected = () => { };
        }

        if (this.status == AsyncPromise.FULFILLED) {   // 状态改变时执行

            setTimeout(() => {  // 晚于线程同步任务执行
                try {  // then处理成功的函数onfulfilled中出现异常，交由当前then处理失败的函数onrejected函数进行处理。这个后面会做优化
                    onfulfilled(this.value);
                } catch (e) {
                    onrejected(e);
                }
            })
        }

        if (this.status == AsyncPromise.REJECTED) {   // 状态改变时执行

            setTimeout(() => {  // 晚于线程同步任务执行
                try {  // then处理失败的函数onrejected中出现异常,交由当前then处理失败的函数onrejected函数进行处理。这个后面会做优化
                    onrejected(this.value);
                } catch (e) {
                    onrejected(e);
                }
            })
        }

        if (this.status == AsyncPromise.PENDING) {  // 如果当前Promise是等待处理状态，则将处理成功的函数与处理失败的函数压入异步数组。
            this.callbacks.push({
                onfulfilled,
                onrejected,
            });
        }
    }
}
```



# 异步执行函数的then异常捕获

上面我们对同步执行函数executor调用then()方法中可能出现的异常进行了处理。

就是下面这一段代码。

```
if (this.status == AsyncPromise.FULFILLED) {   // 状态改变时执行

    setTimeout(() => {  // 晚于线程同步任务执行
        try {  // then处理成功的函数onfulfilled中出现异常，交由当前then处理失败的函数onrejected函数进行处理。这个后面会做优化
            onfulfilled(this.value);
        } catch (e) {
            onrejected(e);
        }
    })
}

if (this.status == AsyncPromise.REJECTED) {   // 状态改变时执行

    setTimeout(() => {  // 晚于线程同步任务执行
        try {  // then处理失败的函数onrejected中出现异常,交由当前then处理失败的函数onrejected函数进行处理。这个后面会做优化
            onrejected(this.value);
        } catch (e) {
            onrejected(e);
        }
    })
}
```

但是我们还没有对异步执行函数executor调用then()方法中可能出现的异常进行处理。

```
if (this.status == AsyncPromise.PENDING) {  // 如果当前Promise是等待处理状态，则将处理成功的函数与处理失败的函数压入异步数组。
    this.callbacks.push({
        onfulfilled,
        onrejected,
    });
}
}
```

这会导致下面这样的使用场景出现问题。

```
<script src="./Promise核心.js"></script>
<script>

    "use strict";

    new AsyncPromise((resolve, reject) => {
        setTimeout(() => {
            resolve("成功");
        }, 3000);

    }).then((success) => {
        throw new Error("自定义异常抛出");  // 直接在处理成功状态的函数onfulfilled中抛出了异常，显然是不符合原生Promise的
    });

</script>
```

那么我们就来加上异常捕获即可，这里还是先传递给当前then()处理rejected状态的函数，后面会做修改。

因为原版Promise会传递给下一个then()中处理rejected状态的函数，而不是当前then()。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {

        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值

        this.callbacks = []; // 如果是一个异步操作，则放入该数组中

        try {
            executor(this.resolve.bind(this), this.reject.bind(this));  // 传递形参，运行executor函数
        } catch (e) {
            this.status = AsyncPromise.REJECTED; // 异常发生改变状态
            this.reject(e); // 记录异常信息
        }


    }

    resolve(value) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.FULFILLED;
            this.value = value;

            this.callbacks.map(callback => {  // // 调用处理异步executor里resolve的方法。
                callback.onfulfilled(value);
            })
        }
    }

    reject(reason) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.REJECTED;
            this.value = reason;

            this.callbacks.map(callback => {  // 调用处理异步executor里reject的方法。
                callback.onrejected(reason);
            })

        }
    }

    then(onfulfilled, onrejected) {

        if (typeof onfulfilled != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onfulfilled = () => { };
        }

        if (typeof onrejected != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onrejected = () => { };
        }

        if (this.status == AsyncPromise.FULFILLED) {   // 状态改变时执行

            setTimeout(() => {  // 晚于线程同步任务执行
                try {  // then处理成功的函数onfulfilled中出现异常，交由当前then处理失败的函数onrejected函数进行处理。这个后面会做优化
                    onfulfilled(this.value);
                } catch (e) {
                    onrejected(e);
                }
            })
        }

        if (this.status == AsyncPromise.REJECTED) {   // 状态改变时执行

            setTimeout(() => {  // 晚于线程同步任务执行
                try {  // then处理失败的函数onrejected中出现异常,交由当前then处理失败的函数onrejected函数进行处理。这个后面会做优化
                    onrejected(this.value);
                } catch (e) {
                    onrejected(e);
                }
            })
        }

        if (this.status == AsyncPromise.PENDING) {  // 如果当前Promise是等待处理状态，则将处理成功的函数与处理失败的函数压入异步数组。

            this.callbacks.push({
                onfulfilled: value => {
                    try {  //  异步executor改变状态对其then中的onfulfilled进行异常捕获
                        onfulfilled(value);
                    } catch (e) {
                        onrejected(e);
                    }
                },
                onrejected: value => {
                    try {  //  异步executor改变状态对其then中的onrejected进行异常捕获
                        onrejected(value);
                    } catch (e) {
                        onrejected(e);
                    }
                }
            });
        }
    }
}
```



# then方法链式操作

对于原生的Promise来讲，每一个then()最后返回的都是一个新的Promise。所以才能达到支持不断的then()进行链式操作，所以我们也可以这样做。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {

        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值

        this.callbacks = []; // 如果是一个异步操作，则放入该数组中

        try {
            executor(this.resolve.bind(this), this.reject.bind(this));  // 传递形参，运行executor函数
        } catch (e) {
            this.status = AsyncPromise.REJECTED; // 异常发生改变状态
            this.reject(e); // 记录异常信息
        }


    }

    resolve(value) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.FULFILLED;
            this.value = value;

            this.callbacks.map(callback => {  // // 调用处理异步executor里resolve的方法。
                callback.onfulfilled(value);
            })
        }
    }

    reject(reason) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.REJECTED;
            this.value = reason;

            this.callbacks.map(callback => {  // 调用处理异步executor里reject的方法。
                callback.onrejected(reason);
            })

        }
    }

    then(onfulfilled, onrejected) {

        if (typeof onfulfilled != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onfulfilled = () => { };
        }

        if (typeof onrejected != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onrejected = () => { };
        }

        return new AsyncPromise((resolve, reject) => { // 返回一个新的Promise

            if (this.status == AsyncPromise.FULFILLED) {   // 状态改变时执行

                setTimeout(() => {  // 晚于线程同步任务执行
                    try {  // then处理成功的函数onfulfilled中出现异常，交由当前then处理失败的函数onrejected函数进行处理。这个后面会做优化
                        onfulfilled(this.value);
                    } catch (e) {
                        onrejected(e);
                    }
                })
            }

            if (this.status == AsyncPromise.REJECTED) {   // 状态改变时执行

                setTimeout(() => {  // 晚于线程同步任务执行
                    try {  // then处理失败的函数onrejected中出现异常,交由当前then处理失败的函数onrejected函数进行处理。这个后面会做优化
                        onrejected(this.value);
                    } catch (e) {
                        onrejected(e);
                    }
                })
            }

            if (this.status == AsyncPromise.PENDING) {  // 如果当前Promise是等待处理状态，则将处理成功的函数与处理失败的函数压入异步数组。

                this.callbacks.push({
                    onfulfilled: value => {
                        try {  //  异步executor改变状态对其then中的onfulfilled进行异常捕获
                            onfulfilled(value);
                        } catch (e) {
                            onrejected(e);
                        }
                    },
                    onrejected: value => {
                        try {  //  异步executor改变状态对其then中的onrejected进行异常捕获
                            onrejected(value);
                        } catch (e) {
                            onrejected(e);
                        }
                    }
                });
            }
        });


    }
}
```

现在我们的Promise已经支持then()的链式操作了，但是上面代码还是遗留了几个问题。

1. then()还没有返回值，返回普通值该怎么处理，返回一个新的Promise该怎么处理
2. 没有异常传递，原生Promise中的then()当抛出异常时应该进行捕获并传递给下一个then()
3. 不支持then()穿透
4. 不支持类型限制

接下来继续对代码做出优化调整。



# then中返回普通值

在原生的Promise中每一个then()所产生的Promise默认状态都是fulfilled，如果当前then()返回是一个值的话那么下一个then()将接受到该值。

这个也非常简单，代码接收一下每一个onfulfilled()与onrejected()的返回值就好，并使用resolve()改变状态为fulfilled以及将值进行传递给下一个then()。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {

        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值

        this.callbacks = []; // 如果是一个异步操作，则放入该数组中

        try {
            executor(this.resolve.bind(this), this.reject.bind(this));  // 传递形参，运行executor函数
        } catch (e) {
            this.status = AsyncPromise.REJECTED; // 异常发生改变状态
            this.reject(e); // 记录异常信息
        }


    }

    resolve(value) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.FULFILLED;
            this.value = value;

            this.callbacks.map(callback => {  // // 调用处理异步executor里resolve的方法。
                callback.onfulfilled(value);
            })
        }
    }

    reject(reason) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.REJECTED;
            this.value = reason;

            this.callbacks.map(callback => {  // 调用处理异步executor里reject的方法。
                callback.onrejected(reason);
            })

        }
    }

    then(onfulfilled, onrejected) {

        if (typeof onfulfilled != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onfulfilled = () => { };
        }

        if (typeof onrejected != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onrejected = () => { };
        }

        return new AsyncPromise((resolve, reject) => { // 返回一个新的Promise

            if (this.status == AsyncPromise.FULFILLED) {   // 状态改变时执行

                setTimeout(() => {  // 晚于线程同步任务执行
                    try {  // then处理成功的函数onfulfilled中出现异常，交由当前then处理失败的函数onrejected函数进行处理。这个后面会做优化
                        let result = onfulfilled(this.value);
                        resolve(result);
                    } catch (e) {
                        onrejected(e);
                    }
                })
            }

            if (this.status == AsyncPromise.REJECTED) {   // 状态改变时执行

                setTimeout(() => {  // 晚于线程同步任务执行
                    try {  // then处理失败的函数onrejected中出现异常,交由当前then处理失败的函数onrejected函数进行处理。这个后面会做优化
                        let result = onrejected(this.value);
                        resolve(result);
                    } catch (e) {
                        onrejected(e);
                    }
                })
            }

            if (this.status == AsyncPromise.PENDING) {  // 如果当前Promise是等待处理状态，则将处理成功的函数与处理失败的函数压入异步数组。

                this.callbacks.push({
                    onfulfilled: value => {
                        try {  //  异步executor改变状态对其then中的onfulfilled进行异常捕获
                            let result = onfulfilled(value);
                            resolve(result);
                        } catch (e) {
                            onrejected(e);
                        }
                    },
                    onrejected: value => {
                        try {  //  异步executor改变状态对其then中的onrejected进行异常捕获
                            let result = onrejected(value);
                            resolve(result);
                        } catch (e) {
                            onrejected(e);
                        }
                    }
                });
            }
        });


    }
}
```

这样我们的then()就支持返回普通值了。

```
<script src="./Promise核心.js"></script>
<script>

    "use strict";

    new AsyncPromise((resolve, reject) => {
        setTimeout(() => {
            resolve("成功");
        }, 3000);
    }).then((success) => {
        return "hello";
    }).then((success) => {
        console.log(success);  // hello
    });

</script>
```



# then中的异常传递

在上面的代码中，then()方法里的处理成功函数onfulfilled以及处理失败函数onrejected在代码执行时抛出的异常都会统一进行捕获并且传递给当前then()方法处理失败的函数onrejected。

这个与原生的Promise有出入，对于原生Promise来讲应该是传递给下一个then()进行处理而不是当前then()。

改动也非常简单，将原来发生异常传递的函数onrejected()改为reject()即可，这就是传递给下一个then()。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {

        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值

        this.callbacks = []; // 如果是一个异步操作，则放入该数组中

        try {
            executor(this.resolve.bind(this), this.reject.bind(this));  // 传递形参，运行executor函数
        } catch (e) {
            this.status = AsyncPromise.REJECTED; // 异常发生改变状态
            this.reject(e); // 记录异常信息
        }


    }

    resolve(value) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.FULFILLED;
            this.value = value;

            this.callbacks.map(callback => {  // // 调用处理异步executor里resolve的方法。
                callback.onfulfilled(value);
            })
        }
    }

    reject(reason) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.REJECTED;
            this.value = reason;

            this.callbacks.map(callback => {  // 调用处理异步executor里reject的方法。
                callback.onrejected(reason);
            })

        }
    }

    then(onfulfilled, onrejected) {

        if (typeof onfulfilled != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onfulfilled = () => { };
        }

        if (typeof onrejected != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onrejected = () => { };
        }

        return new AsyncPromise((resolve, reject) => { // 返回一个新的Promise

            if (this.status == AsyncPromise.FULFILLED) {   // 状态改变时执行

                setTimeout(() => {  // 晚于线程同步任务执行
                    try {
                        let result = onfulfilled(this.value);
                        resolve(result);
                    } catch (e) {
                        reject(e); // 传递给下一个then
                    }
                })
            }

            if (this.status == AsyncPromise.REJECTED) {   // 状态改变时执行

                setTimeout(() => {  // 晚于线程同步任务执行
                    try {
                        let result = onrejected(this.value);
                        resolve(result);
                    } catch (e) {
                        reject(e); // 传递给下一个then
                    }
                })
            }

            if (this.status == AsyncPromise.PENDING) {  // 如果当前Promise是等待处理状态，则将处理成功的函数与处理失败的函数压入异步数组。

                this.callbacks.push({
                    onfulfilled: value => {
                        try {
                            let result = onfulfilled(value);
                            resolve(result);
                        } catch (e) {
                            reject(e); // 传递给下一个then
                        }
                    },
                    onrejected: value => {
                        try {
                            let result = onrejected(value);
                            resolve(result);
                        } catch (e) {
                            reject(e); // 传递给下一个then
                        }
                    }
                });
            }
        });


    }
}

```

代码测试：

```
<script src="./Promise核心.js"></script>
<script>

    "use strict";

    new AsyncPromise((resolve, reject) => {
        setTimeout(() => {
            resolve("成功");
        }, 3000);
    }).then((success) => {
        throw new Error("新错误");
    }).then(null, error => {
        console.log(error);  // 上一个then的错误成功由该then接收
    });

</script>
```



# then穿透功能实现

在原生的Promise中是支持then()的穿透传值的。

```
<script>

    "use strict";

    new Promise((resolve, reject) => {

        resolve("成功");

    })
        .then() // 穿透
        .then(
            success => {
                console.log(success); // 成功
            },
            error => {
                console.log(error);
            })

</script>
```

但是我们的Promise却不支持。

```
<script src="./Promise核心.js"></script>
<script>

    "use strict";

    new AsyncPromise((resolve, reject) => {

        resolve("成功");

    })
        .then() // 不支持穿透
        .then(
            success => {
                console.log(success);

            },
            error => {
                console.log(error);
            })

</script>
```

原因在于如果没有对then()进行传递参数，那么内部其实是会创建两个空函数。

我们只需要在空函数内部返回this.value即可。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {

        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值

        this.callbacks = []; // 如果是一个异步操作，则放入该数组中

        try {
            executor(this.resolve.bind(this), this.reject.bind(this));  // 传递形参，运行executor函数
        } catch (e) {
            this.status = AsyncPromise.REJECTED; // 异常发生改变状态
            this.reject(e); // 记录异常信息
        }


    }

    resolve(value) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.FULFILLED;
            this.value = value;

            this.callbacks.map(callback => {  // // 调用处理异步executor里resolve的方法。
                callback.onfulfilled(value);
            })
        }
    }

    reject(reason) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.REJECTED;
            this.value = reason;

            this.callbacks.map(callback => {  // 调用处理异步executor里reject的方法。
                callback.onrejected(reason);
            })

        }
    }

    then(onfulfilled, onrejected) {

        if (typeof onfulfilled != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onfulfilled = () => this.value;  // 支持穿透
        }

        if (typeof onrejected != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onrejected = () => this.value;  // 支持穿透
        }

        return new AsyncPromise((resolve, reject) => { // 返回一个新的Promise

            if (this.status == AsyncPromise.FULFILLED) {   // 状态改变时执行

                setTimeout(() => {  // 晚于线程同步任务执行
                    try {
                        let result = onfulfilled(this.value);
                        resolve(result);
                    } catch (e) {
                        reject(e); // 传递给下一个then
                    }
                })
            }

            if (this.status == AsyncPromise.REJECTED) {   // 状态改变时执行

                setTimeout(() => {  // 晚于线程同步任务执行
                    try {
                        let result = onrejected(this.value);
                        resolve(result);
                    } catch (e) {
                        reject(e); // 传递给下一个then
                    }
                })
            }

            if (this.status == AsyncPromise.PENDING) {  // 如果当前Promise是等待处理状态，则将处理成功的函数与处理失败的函数压入异步数组。

                this.callbacks.push({
                    onfulfilled: value => {
                        try {
                            let result = onfulfilled(value);
                            resolve(result);
                        } catch (e) {
                            reject(e); // 传递给下一个then
                        }
                    },
                    onrejected: value => {
                        try {
                            let result = onrejected(value);
                            resolve(result);
                        } catch (e) {
                            reject(e); // 传递给下一个then
                        }
                    }
                });
            }
        });


    }
}
```



# then返回Promise

原生的Promise支持返回一个新的Promise，但是我们的Promise现在还不支持。

其实也很简单，判断一下then()中两个函数返回的是不是一个新的Promise，如果是的话则使用其then()方法将其中resolve()或reject()的值进行传递。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {

        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值

        this.callbacks = []; // 如果是一个异步操作，则放入该数组中

        try {
            executor(this.resolve.bind(this), this.reject.bind(this));  // 传递形参，运行executor函数
        } catch (e) {
            this.status = AsyncPromise.REJECTED; // 异常发生改变状态
            this.reject(e); // 记录异常信息
        }


    }

    resolve(value) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.FULFILLED;
            this.value = value;

            this.callbacks.map(callback => {  // // 调用处理异步executor里resolve的方法。
                callback.onfulfilled(value);
            })
        }
    }

    reject(reason) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.REJECTED;
            this.value = reason;

            this.callbacks.map(callback => {  // 调用处理异步executor里reject的方法。
                callback.onrejected(reason);
            })

        }
    }

    then(onfulfilled, onrejected) {

        if (typeof onfulfilled != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onfulfilled = () => this.value;  // 支持穿透
        }

        if (typeof onrejected != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onrejected = () => this.value;  // 支持穿透
        }

        return new AsyncPromise((resolve, reject) => { // 返回一个新的Promise

            if (this.status == AsyncPromise.FULFILLED) {   // 状态改变时执行

                setTimeout(() => {  // 晚于线程同步任务执行
                    try {
                        let result = onfulfilled(this.value);
                        if (result instanceof AsyncPromise) {  // 判断是否返回Promise对象
                            result.then(resolve, reject);
                        } else {
                            resolve(result); // 改变状态并将值交由下一个then接收
                        }
                    } catch (e) {
                        reject(e); // 传递给下一个then
                    }
                })
            }

            if (this.status == AsyncPromise.REJECTED) {   // 状态改变时执行

                setTimeout(() => {  // 晚于线程同步任务执行
                    try {
                        let result = onrejected(this.value);
                        if (result instanceof AsyncPromise) {  // 判断是否返回Promise对象
                            result.then(resolve, reject);
                        } else {
                            resolve(result); // 改变状态并将值交由下一个then接收
                        }
                    } catch (e) {
                        reject(e); // 传递给下一个then
                    }
                })
            }

            if (this.status == AsyncPromise.PENDING) {  // 如果当前Promise是等待处理状态，则将处理成功的函数与处理失败的函数压入异步数组。

                this.callbacks.push({
                    onfulfilled: value => {
                        try {
                            let result = onfulfilled(value);
                            if (result instanceof AsyncPromise) {  // 判断是否返回Promise对象
                                result.then(resolve, reject);
                            } else {
                                resolve(result); // 改变状态并将值交由下一个then接收
                            }
                        } catch (e) {
                            reject(e); // 传递给下一个then
                        }
                    },
                    onrejected: value => {
                        try {
                            let result = onrejected(value);
                            if (result instanceof AsyncPromise) {  // 判断是否返回Promise对象
                                result.then(resolve, reject);
                            } else {
                                resolve(result); // 改变状态并将值交由下一个then接收
                            }
                        } catch (e) {
                            reject(e); // 传递给下一个then
                        }
                    }
                });
            }
        });


    }
}
```



# then的代码优化

可以观察到上面的then()方法中有很多重复代码，所以我们需要对重复代码做一下优化。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {

        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值

        this.callbacks = []; // 如果是一个异步操作，则放入该数组中

        try {
            executor(this.resolve.bind(this), this.reject.bind(this));  // 传递形参，运行executor函数
        } catch (e) {
            this.status = AsyncPromise.REJECTED; // 异常发生改变状态
            this.reject(e); // 记录异常信息
        }

    }

    resolve(value) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.FULFILLED;
            this.value = value;

            this.callbacks.map(callback => {  // // 调用处理异步executor里resolve的方法。
                callback.onfulfilled(value);
            })
        }
    }

    reject(reason) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.REJECTED;
            this.value = reason;

            this.callbacks.map(callback => {  // 调用处理异步executor里reject的方法。
                callback.onrejected(reason);
            })

        }
    }

    then(onfulfilled, onrejected) {

        if (typeof onfulfilled != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onfulfilled = () => this.value;  // 支持穿透
        }

        if (typeof onrejected != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onrejected = () => this.value;  // 支持穿透
        }

        return new AsyncPromise((resolve, reject) => { // 返回一个新的Promise

            if (this.status == AsyncPromise.FULFILLED) {   // 状态改变时执行

                setTimeout(() => {  // 晚于线程同步任务执行
                    this.parse(onfulfilled(this.value), resolve, reject);
                })
            }

            if (this.status == AsyncPromise.REJECTED) {   // 状态改变时执行

                setTimeout(() => {  // 晚于线程同步任务执行
                    this.parse(onrejected(this.value), resolve, reject);
                })
            }

            if (this.status == AsyncPromise.PENDING) {  // 如果当前Promise是等待处理状态，则将处理成功的函数与处理失败的函数压入异步数组。
                this.callbacks.push({
                    onfulfilled: value => {
                        this.parse(onfulfilled(value), resolve, reject);
                    },
                    onrejected: value => {
                        this.parse(onrejected(value), resolve, reject);
                    }
                });
            }
        });

    }

    parse(result, resolve, reject) {
        try {
            if (result instanceof AsyncPromise) {  // 判断是否返回Promise对象
                result.then(resolve, reject);
            } else {
                resolve(result); // 改变状态并将值交由下一个then接收
            }

        } catch (e) {
            reject(e);  // 向下传递异常
        }
    }
}
```



# then返回类型限制

我们都知道then()会创建一个Promise并返回，但是原生的Promise不支持then()将自己创建的Promise进行返回

```
<script>

    "use strict";

    let task = new Promise((resolve, reject) => {

        resolve("成功");

    })

    let then01 = task.then(  // 由于then中的处理成功与处理失败的函数是属于异步执行。所以会先将创建好的Promise对象返回再运行其中的处理成功函数与处理失败函数。
        success => {
            return p2;
        }
    )

    // Uncaught (in promise) TypeError: Chaining cycle detected for promise #<Promise>

</script>
```

但是我们的Promise还不支持这一点，所以需要改一改代码。

解决的思路也很简单，在运行失败或处理函数时判断一下本次返回的值是否等同于创建的Promise对象。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {

        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值

        this.callbacks = []; // 如果是一个异步操作，则放入该数组中

        try {
            executor(this.resolve.bind(this), this.reject.bind(this));  // 传递形参，运行executor函数
        } catch (e) {
            this.status = AsyncPromise.REJECTED; // 异常发生改变状态
            this.reject(e); // 记录异常信息
        }

    }

    resolve(value) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.FULFILLED;
            this.value = value;

            this.callbacks.map(callback => {  // // 调用处理异步executor里resolve的方法。
                callback.onfulfilled(value);
            })
        }
    }

    reject(reason) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.REJECTED;
            this.value = reason;

            this.callbacks.map(callback => {  // 调用处理异步executor里reject的方法。
                callback.onrejected(reason);
            })

        }
    }

    then(onfulfilled, onrejected) {

        if (typeof onfulfilled != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onfulfilled = () => this.value;  // 支持穿透
        }

        if (typeof onrejected != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onrejected = () => this.value;  // 支持穿透
        }

        let promise = new AsyncPromise((resolve, reject) => { // 返回一个新的Promise

            if (this.status == AsyncPromise.FULFILLED) {   // 状态改变时执行

                setTimeout(() => {  // 晚于线程同步任务执行
                    this.parse(promise, onfulfilled(this.value), resolve, reject);
                })
            }

            if (this.status == AsyncPromise.REJECTED) {   // 状态改变时执行

                setTimeout(() => {  // 晚于线程同步任务执行
                    this.parse(promise, onrejected(this.value), resolve, reject);
                })
            }

            if (this.status == AsyncPromise.PENDING) {  // 如果当前Promise是等待处理状态，则将处理成功的函数与处理失败的函数压入异步数组。

                this.callbacks.push({
                    onfulfilled: value => {
                        this.parse(promise, onfulfilled(value), resolve, reject);
                    },
                    onrejected: value => {
                        this.parse(promise, onrejected(value), resolve, reject);
                    }
                });
            }
        });

        return promise; // 同步，先返回。onfulfilled与onrejected由于套了setTimeout，是异步执行。

    }

    parse(promise, result, resolve, reject) {

        if (promise == result) {
            throw new TypeError("Chaining cycle detected");
        }

        try {
            if (result instanceof AsyncPromise) {  // 判断是否返回Promise对象
                result.then(resolve, reject);
            } else {
                resolve(result); // 改变状态并将值交由下一个then接收
            }

        } catch (e) {
            reject(e);  // 向下传递异常
        }
    }
}
```



# resolve与reject实现

使用 Promise.resolve() 方法可以快速的返回一个状态是fulfilled的Promise对象。

使用 Promise.reject() 方法可以快速的返回一个状态是rejected的Promise对象。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {

        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值

        this.callbacks = []; // 如果是一个异步操作，则放入该数组中

        try {
            executor(this.resolve.bind(this), this.reject.bind(this));  // 传递形参，运行executor函数
        } catch (e) {
            this.status = AsyncPromise.REJECTED; // 异常发生改变状态
            this.reject(e); // 记录异常信息
        }

    }

    resolve(value) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.FULFILLED;
            this.value = value;

            this.callbacks.map(callback => {  // // 调用处理异步executor里resolve的方法。
                callback.onfulfilled(value);
            })
        }
    }

    reject(reason) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.REJECTED;
            this.value = reason;

            this.callbacks.map(callback => {  // 调用处理异步executor里reject的方法。
                callback.onrejected(reason);
            })

        }
    }

    then(onfulfilled, onrejected) {

        if (typeof onfulfilled != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onfulfilled = () => this.value;  // 支持穿透
        }

        if (typeof onrejected != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onrejected = () => this.value;  // 支持穿透
        }

        let promise = new AsyncPromise((resolve, reject) => { // 返回一个新的Promise

            if (this.status == AsyncPromise.FULFILLED) {   // 状态改变时执行

                setTimeout(() => {  // 晚于线程同步任务执行
                    this.parse(promise, onfulfilled(this.value), resolve, reject);
                })
            }

            if (this.status == AsyncPromise.REJECTED) {   // 状态改变时执行

                setTimeout(() => {  // 晚于线程同步任务执行
                    this.parse(promise, onrejected(this.value), resolve, reject);
                })
            }

            if (this.status == AsyncPromise.PENDING) {  // 如果当前Promise是等待处理状态，则将处理成功的函数与处理失败的函数压入异步数组。

                this.callbacks.push({
                    onfulfilled: value => {
                        this.parse(promise, onfulfilled(value), resolve, reject);
                    },
                    onrejected: value => {
                        this.parse(promise, onrejected(value), resolve, reject);
                    }
                });
            }
        });

        return promise; // 同步，先返回。onfulfilled与onrejected由于套了setTimeout，是异步执行。

    }

    parse(promise, result, resolve, reject) {

        if (promise == result) {
            throw new TypeError("Chaining cycle detected");
        }

        try {
            if (result instanceof AsyncPromise) {  // 判断是否返回Promise对象
                result.then(resolve, reject);
            } else {
                resolve(result); // 改变状态并将值交由下一个then接收
            }

        } catch (e) {
            reject(e);  // 向下传递异常
        }
    }

    static resolve(value) {
        return new AsyncPromise((resolve, reject) => {
            if (value instanceof AsyncPromise) {
                value.then(resolve, reject);
            } else {
                resolve(value);
            }
        });
    }

    static reject(value) {
        return new AsyncPromise((resolve, reject) => {
            reject(value);
        });
    }
}
```



# all与race实现

使用Promise.all() 方法可以同时执行多个并行异步操作，比如页面加载时同进获取课程列表与推荐课程。任何一个 Promise 执行失败就会调用 catch方法，成功后返回 Promise 结果的有序数组。（Ps：我们这个Promise没有实现catch方法）

使用Promise.race() 处理容错异步，和race单词一样哪个Promise快用哪个，哪个先返回用哪个。

```
class AsyncPromise {

    static PENDING = "pending";
    static FULFILLED = "fulfilled";
    static REJECTED = "rejected";

    constructor(executor) {

        this.status = AsyncPromise.PENDING;  // 初始状态为准备状态
        this.value = null;  // 初始值

        this.callbacks = []; // 如果是一个异步操作，则放入该数组中

        try {
            executor(this.resolve.bind(this), this.reject.bind(this));  // 传递形参，运行executor函数
        } catch (e) {
            this.status = AsyncPromise.REJECTED; // 异常发生改变状态
            this.reject(e); // 记录异常信息
        }

    }

    resolve(value) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.FULFILLED;
            this.value = value;

            this.callbacks.map(callback => {  // // 调用处理异步executor里resolve的方法。
                callback.onfulfilled(value);
            })
        }
    }

    reject(reason) {

        if (this.status == AsyncPromise.PENDING) {  // 限制
            this.status = AsyncPromise.REJECTED;
            this.value = reason;

            this.callbacks.map(callback => {  // 调用处理异步executor里reject的方法。
                callback.onrejected(reason);
            })

        }
    }

    then(onfulfilled, onrejected) {

        if (typeof onfulfilled != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onfulfilled = () => this.value;  // 支持穿透
        }

        if (typeof onrejected != "function") {  // 如果传入的不是一个函数，默认创建空函数
            onrejected = () => this.value;  // 支持穿透
        }

        let promise = new AsyncPromise((resolve, reject) => { // 返回一个新的Promise

            if (this.status == AsyncPromise.FULFILLED) {   // 状态改变时执行

                setTimeout(() => {  // 晚于线程同步任务执行
                    this.parse(promise, onfulfilled(this.value), resolve, reject);
                })
            }

            if (this.status == AsyncPromise.REJECTED) {   // 状态改变时执行

                setTimeout(() => {  // 晚于线程同步任务执行
                    this.parse(promise, onrejected(this.value), resolve, reject);
                })
            }

            if (this.status == AsyncPromise.PENDING) {  // 如果当前Promise是等待处理状态，则将处理成功的函数与处理失败的函数压入异步数组。

                this.callbacks.push({
                    onfulfilled: value => {
                        this.parse(promise, onfulfilled(value), resolve, reject);
                    },
                    onrejected: value => {
                        this.parse(promise, onrejected(value), resolve, reject);
                    }
                });
            }
        });

        return promise; // 同步，先返回。onfulfilled与onrejected由于套了setTimeout，是异步执行。

    }

    parse(promise, result, resolve, reject) {

        if (promise == result) {
            throw new TypeError("Chaining cycle detected");
        }

        try {
            if (result instanceof AsyncPromise) {  // 判断是否返回Promise对象
                result.then(resolve, reject);
            } else {
                resolve(result); // 改变状态并将值交由下一个then接收
            }

        } catch (e) {
            reject(e);  // 向下传递异常
        }
    }

    static resolve(value) {
        return new AsyncPromise((resolve, reject) => {
            if (value instanceof AsyncPromise) {
                value.then(resolve, reject);
            } else {
                resolve(value);
            }
        });
    }

    static reject(value) {
        return new AsyncPromise((resolve, reject) => {
            reject(value);
        });
    }

    static all(value) {

        return new AsyncPromise((resolve, reject) => {
            const values = [];  // 记录当前有多少promise状态是成功
            promise.forEach((promise) => {
                promise.then(value => {
                    values.push(value);
                    if (values.length == promise.length) {
                        resolve(values); // 如果都成功，当前all返回的promise则状态为fulfilled。
                    }
                }, reason => {
                    reject(reason);  // 如果有一个promise错误，则当前all返回的promise则为拒绝
                })
            });
        });

    }

    static race(value) {
        return new AsyncPromise((resolve, reject) => {
            value.forEach(promise => {
                promise.then(value => { // 如果循环中的promise状态为fulfilled，则当前的race创建的promise状态也为resolve
                    resolve(value);
                }, reason => {
                    reject(value);  // 同上
                })
            })
        })
    }
}
```