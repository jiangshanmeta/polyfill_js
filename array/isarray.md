## 基本描述

#### 语法

```javascript
Array.isArray(obj)
```

#### 参数

* obj 想要检测是否为数组的值

#### 返回值

布尔值，表明obj是否为数组

## polyfill

类型判断算是一个基本问题，也算是一个小坑了，毕竟```typeof```这个关键字真心检测不出来array。


```javascript
if (!Array.isArray) {
  Array.isArray = function(arg) {
    return Object.prototype.toString.call(arg) === '[object Array]';
  };
}
```

其实jQuery3的```jQuery.isArray```就是引用的这个方法，毕竟IE9都支持了。