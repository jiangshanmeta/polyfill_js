在ES6中我最喜欢的的两个特性，一个是模块化，另一个是promise。

promise其实和观察者模式非常类似，他们都有自身的订阅者。所不同的是promise决议后事件是异步触发的(即订阅者的回调是异步调用的)，而且在决议后再订阅该事件依然可以触发订阅者的回调(观察者模式事件触发后再订阅事件只能等下一次事件触发)。

我所要采用的promise的polyfill是[github上的一个项目](https://github.com/stefanpenner/es6-promise)提供的，ajax库axios也推荐这个promise的polyfill。


## 异步

异步是promise很重要的一点，这个第三方库的异步相关代码在 **asap.js** 中。

类似于Vue中nextTick方法的实现，这里首先是根据环境尝试选择不同的异步方案，兜底方案就是使用setTimeout方法。

具体实现上维护了一个队列，我们需要异步调用的方法都往这个队列里面放，然后异步处理完整个队列：

```javascript
const queue = new Array(1000);
export var asap = function asap(callback, arg) {
    // 将任务放入到队列中
    queue[len] = callback;
    queue[len + 1] = arg;
    len += 2;

    // 当异步队列为空时向队列添加任务，此时才需要启动异步处理队列
    // 当不为空时添加任务，已经开启了异步处理队列，无需重复处理
    if (len === 2) {
        if (customSchedulerFn) {
            customSchedulerFn(flush);
        } else {
            scheduleFlush();
        }
    }
}

// 处理异步队列的任务
function flush() {
    for (let i = 0; i < len; i+=2) {
        let callback = queue[i];
        let arg = queue[i+1];
        callback(arg);
        queue[i] = undefined;
        queue[i+1] = undefined;
    }

    len = 0;
}
```

## 构造函数

promise的构造函数在 **promise.js** 中，它比较简单，主要是声明了几个私有属性```_result```(反映promise的结果)、```_state```(反映promise的状态)、```_subscribers```(这个promise未处理的订阅者)。


```javascript
export default function Promise(resolver) {
    this[PROMISE_ID] = nextId();
    this._result = this._state = undefined;
    this._subscribers = [];

    if (noop !== resolver) {
        typeof resolver !== 'function' && needsResolver();
        this instanceof Promise ? initializePromise(this, resolver) : needsNew();
    }
}
```

比较难理解的是initializePromise方法：

```javascript
function initializePromise(promise, resolver) {
    try {
        resolver(function resolvePromise(value){
            resolve(promise, value);
        }, function rejectPromise(reason) {
            reject(promise, reason);
        });
    } catch(e) {
        reject(promise, e);
    }
}
```

在promise世界有个词叫 **revealing constructor** ,构造器是由外界传进来的(resolver方法)，resolver会立即执行，并且会传入两个函数作为参数。

## 订阅 决议 

之前提到过，promise和观察者模式有相似之处，他们都有自己的订阅者。上面已经提到，订阅者相关信息被存放到了```_subscribers```这一私有属性中。订阅是在then方法中调用的,我们看一下订阅的实现：


```javascript
function subscribe(parent, child, onFulfillment, onRejection) {
    let { _subscribers } = parent;
    let { length } = _subscribers;

    parent._onerror = null;

    _subscribers[length] = child;
    _subscribers[length + FULFILLED] = onFulfillment;
    _subscribers[length + REJECTED]  = onRejection;

    if (length === 0 && parent._state) {
        asap(publish, parent);
    }
}
```

promise的then方法会返回一个新promise，对应这里的child，onFulfillment、onRejection分别对应then方法的两个参数。

注册完我们就可以等决议了：

```javascript
function resolve(promise, value) {
    if (promise === value) {
        reject(promise, selfFulfillment());
    } else if (objectOrFunction(value)) {
        handleMaybeThenable(promise, value, getThen(value));
    } else {
        fulfill(promise, value);
    }
}
function reject(promise, reason) {
    if (promise._state !== PENDING) { return; }
    promise._state = REJECTED;
    promise._result = reason;

    asap(publishRejection, promise);
}
function fulfill(promise, value) {
    if (promise._state !== PENDING) { return; }

    promise._result = value;
    promise._state = FULFILLED;

    if (promise._subscribers.length !== 0) {
        asap(publish, promise);
    }
}
```

这里请允许我先发明两个概念，一个是临时决议值，一个是最终决议值。临时决议值是我们在promise构造器中调用resolve方法传入的值，或者then方法resolve回调中返回的值，临时决议值可以是最终决议值，也可以不是，这主要取决于临时决议值是不是一个promise(传入thenable没遇到过，不考虑)。如果临时决议值不是promise，就会通过fulfill转换成最终决议值，同时promise的状态也更新为resolved。类似于观察值模式，已决议的promise会通知订阅者已决议这个事件：

```javascript
function publish(promise) {
    let subscribers = promise._subscribers;
    let settled = promise._state;

    if (subscribers.length === 0) { return; }

    let child, callback, detail = promise._result;

    // 遍历订阅者，执行回调
    for (let i = 0; i < subscribers.length; i += 3) {
        // child是then方法创造出来的新promise
        child = subscribers[i];
        callback = subscribers[i + settled];

        if (child) {
            invokeCallback(settled, child, callback, detail);
        } else {
            callback(detail);
        }
    }

    // _subscribers维护的是为执行的订阅者，所以这里要清空
    promise._subscribers.length = 0;
}
```

你可能会好奇invokeCallback起到了什么作用。这个方法是把决议按照promise链向后传递的关键：

```javascript
function invokeCallback(settled, promise, callback, detail) {
    let hasCallback = isFunction(callback),
        value, error, succeeded, failed;

    if (hasCallback) {
        value = tryCatch(callback, detail);

        if (value === TRY_CATCH_ERROR) {
            failed = true;
            error = value.error;
            value.error = null;
        } else {
            succeeded = true;
        }

        if (promise === value) {
            reject(promise, cannotReturnOwn());
            return;
        }

    } else {
        value = detail;
        succeeded = true;
    }

    // 决议向后传递的关键
    if (promise._state !== PENDING) {
    // noop
    } else if (hasCallback && succeeded) {
        resolve(promise, value);
    } else if (failed) {
        reject(promise, error);
    } else if (settled === FULFILLED) {
        fulfill(promise, value);
    } else if (settled === REJECTED) {
        reject(promise, value);
    }
}
```

这里的一个问题是有没有对应的回调函数，毕竟决议状态有两种。如果有对应的回调函数，回调函数的返回值就是临时决议值，需要通过resolve方法进一步处理，如果没有，这个then方法产生的promise的决议值和决议状态由其订阅的promise(父promise)决定，即可以简单使用fulfill方法(因为父promise已经决议)。

到现在还有两个问题，一个是如果调用then方法时promise已决议怎么办，另一个是临时决议值如果是promise怎么办。

第一个方法的解决方案其实很简单，就是判断一下promise的状态，如果未决议走订阅的路子，如果已决议走异步执行回调的路子。

```javascript
export default function then(onFulfillment, onRejection) {
    const parent = this;

    const child = new this.constructor(noop);

    if (child[PROMISE_ID] === undefined) {
        makePromise(child);
    }

    const { _state } = parent;

    // 如果已决议异步执行回调
    if (_state) {
        const callback = arguments[_state - 1];
        asap(() => invokeCallback(_state, child, callback, parent._result));
    } else {
        subscribe(parent, child, onFulfillment, onRejection);
    }

    return child;
}
```

第二个问题就比较复杂了：

```javascript
// 这个方法关联两个promise
function handleOwnThenable(promise, thenable) {
    if (thenable._state === FULFILLED) {
        fulfill(promise, thenable._result);
    } else if (thenable._state === REJECTED) {
        reject(promise, thenable._result);
    } else {
        subscribe(
            thenable, 
            undefined, 
            value  => resolve(promise, value),
            reason => reject(promise, reason)
        )
    }
}
```

上面代码中thenable是类型为promise的临时决议值，在这种情况下原始promise的最终决议值由这个临时决议promise决定。请注意这种模式下订阅者是关联到父promise而不是返回值promise，但是父promise的状态由子promise决定。


## Promise.resolve  Promise.reject

这两个静态方法其实只是语法糖罢了，Promise.resolve(obj)相当于：

```javascript
new Promise((resolve,reject)=>{
    resolve(obj)
})
```

所以Promise的实现就非常简单：

```javascript
Promise.all = function(obj){
    return new Promise(function(resolve,reject){
        resolve(obj);
    })
}
```

类似的，我们可以得到Promise.reject的polyfill：

```javascript
Promise.reject = function(obj){
    return new Promise(function(resolve,reject){
        reject(obj);
    });
}
```

上面提到的实现和这个提到的第三方库的实现有所出入，但最终结果都是返回已决议的promise。

Promise.reject我用得不多，Promise.resolve用处相对较多，后者的功能除了看做是返回一个已决议promise，还可以看做是把非promise包装成promise，这一特性在静态方法Promise.race和Promise.all中得到了应用。


## Promise.race Promise.all

Promise.race是传入一个数组，返回的是一个新promise，新promise的状态(resolve还是reject)以及决议值由传入的数组对应的promise中第一个已决议的promise决定。


```javascript
export default function race(entries) {
    let Constructor = this;

    if (!isArray(entries)) {
        return new Constructor((_, reject) => reject(new TypeError('You must pass an array to race.')));
    } else {
        return new Constructor((resolve, reject) => {
            let length = entries.length;
            for (let i = 0; i < length; i++) {
                Constructor.resolve(entries[i]).then(resolve, reject);
            }
        });
    }
}
```

这段代码的核心是```Constructor.resolve(entries[i]).then(resolve, reject);```，首先是把传入数组中所有元素包装成promise(事实上如果传入的是promise也会被包装一层)，然后等待第一个决议的promise，由这个promise决定返回的promise的状态和值。


这个第三方库给出的Promise.all的polyfill我感觉过于复杂，下面是我给出的方案：

```javascript
Promise.all = function(arr){
    return new Promise((resolve,reject)=>{
        // 此处应有对arr数据类型的校验

        let rst = [];
        let counter = 0;
        let len = arr.length;
        for(let i=0;i<len;i++){
            Promise.resolve(arr[i]).then((value)=>{
                rst[i] = value;
                counter++;
                if(counter===len){
                    resolve(rst);
                }
            }).catch((value)=>{
                reject(value)
            })
        }
    })
}
```

虽说是polyfill但是依然用到了一些ES6的特性，尤其是let关键字。