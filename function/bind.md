## 基本描述

#### 语法

```javascript
fun.bind(thisArg[, arg1[, arg2[, ...]]])
```

#### 参数

* thisArg 新函数运行时this的指向，注意使用```new```操作符的时候绑定this无效

#### 返回值

bind方法返回一个新的函数，当新函数被调用的时候，bind的第一个参数将作为它运行时的this，之后的参数将会置于实参之前传递给被绑定的方法。

## polyfill

在javascript中**this**算是个难点吧，但是一旦接受了这种设定也就日常了。在```bind```方法出现前，要显式改变this指向只能通过call/apply这两个方法了(下面的polyfill就是应用了这个原理)，但是call/apply方法除了绑定了this指向，还会立即调用函数，所以就有了下面的```fBound```函数。


```javascript
if (!Function.prototype.bind) {
  Function.prototype.bind = function (oThis) {
    if (typeof this !== "function") {
      throw new TypeError("Function.prototype.bind - what is trying to be bound is not callable");
    }
    var aArgs = Array.prototype.slice.call(arguments, 1), 
        fToBind = this, 
        fNOP = function () {},
        fBound = function () {
          return fToBind.apply(this instanceof fNOP
                                 ? this
                                 : oThis || this,
                               aArgs.concat(Array.prototype.slice.call(arguments)));
        };

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();

    return fBound;
  };
}
```

上面是从[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)上搬过来的，虽然行数不多但是包含的知识挺丰富的。

最简单的知识点就是**闭包**，通过bind方法我们得到的新函数```fBound```可以访问到oThis等一系列变量(当然本质上是词法作用域)。

然后这段代码还包含了模拟```Object.create```方法。

```javascript
fNOP = function () {},
fNOP.prototype = this.prototype;
fBound.prototype = new fNOP();
```

相当于

```javascript
fBound.prototype = Object.create(this.prototype);
```

熟悉javascript的你很容易会看出来这里的this指向的是调用bind方法的那个function。这里使用Object.create目的是实现继承。

接下来重点就是```fBound```中```this instanceof fNOP? this: oThis || this```这一句。什么时候这里的this会是fNOP的实例呢？答案是使用new操作符调用的时候，此时按照文档的说明bind希望绑定的this无效，为此才会有这个判断。这里其实也可以判断是否是fBound的实例。

最后需要解释的是：

```javascript
var aArgs = Array.prototype.slice.call(arguments, 1), 

```

```javascript
aArgs.concat(Array.prototype.slice.call(arguments))
```

这两句体现了bind方法预置参数的功能，一般是用来制造偏函数，这个功能其实我用得不多。


希望这些能帮助理解bind方法。