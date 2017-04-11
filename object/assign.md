## 基本描述

#### 语法

```javascript
Object.assign(target, ...sources)
```

#### 参数

* target 目标对象
* sources (多个)源对象

#### 返回值

目标对象target

## polyfill

这个方法是低配版的```jQuery.extend```，它只能实现浅拷贝而不能实现深拷贝，所拷贝的限定为自身可枚举属性，所以需要```for in ```和```hasOwnProperty```，这两个确实经常在一起使用。自身可枚举属性这个限定其实还挺常见的，比如```Object.keys```、```Object.values```、```Object.entries```

```javascript
if (typeof Object.assign != 'function') {
  Object.assign = function(target) {
    'use strict';
    if (target == null) {
      throw new TypeError('Cannot convert undefined or null to object');
    }

    target = Object(target);
    for (var index = 1; index < arguments.length; index++) {
      var source = arguments[index];
      if (source != null) {
        for (var key in source) {
          if (Object.prototype.hasOwnProperty.call(source, key)) {
            target[key] = source[key];
          }
        }
      }
    }
    return target;
  };
}
```

其实我还是挺期望深拷贝能够被标准化。