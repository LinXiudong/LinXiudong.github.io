---
layout: mypost
title: typeof 和 instanceof的用法与区别
categories: [javascript]
---


### typeof

typeof会返回一个变量的基本类型（number,boolean,string,object,undefined,function）。
```javascript
alert(typeof(1));//number
alert(typeof("abc"));//string
```

如果我们想要判断一个变量是否存在，可以使用typeof：(*不能使用if(a)，若a未声明，则报错*)
```javascript
if(typeof a != 'undefined'){
    //变量存在
}
```


### instanceof

instanceof返回的是一个布尔值，如：
```javascript
var a = {};
alert(a instanceof Object);  //true
var b = [];
alert(b instanceof Array);  //true
```

需要注意的是，instanceof只能用来判断对象和函数，不能用来判断字符串和数字等
```javascript
var b = '123';
alert(b instanceof String);  //false
alert(typeof b);  //string
var c = new String("123");
alert(c instanceof String);  //true
alert(typeof c);  //object
```

另外，用instanceof可以判断变量是否为数组。（*typeof不适用于来判断数组，因为不管是数组还是对象，都会返回object*）


判断变量是否为数组有以下方法

- 利用constructor属性

这个属性在我们使用js系统或者自己创建的对象的时候，会默认的加上，例如：
```javascript
var arr = [1,2,3];  //创建一个数组对象
arr.prototype.constructor = Array;  //这一句是系统默认加上的
//所以我们就可以这样来判断：
var arr = [1,2,3,1];
alert(arr.constructor === Array);   // true
```

- 使用instanceof

instanceof是检测对象的原型链是否指向构造函数的prototype对象的，所以我们也可以用它来判断：
```javascript
var arr = [1,2,3];
alert(arr instanceof Array);   // true
```

下面是一个判断是否为数组的终极解决方案：
```javascript
var arr = [1,2,3];
function isArrayFn(obj){  //封装一个函数
    if (typeof Array.isArray === "function") {
        return Array.isArray(obj);  //浏览器支持则使用isArray()方法
    }else{  //否则使用toString方法
        return Object.prototype.toString.call(obj) === "[object Array]";
    }
}
alert(isArrayFn(arr));// true
```
