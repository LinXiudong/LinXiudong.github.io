---
layout: mypost
title: 类数组对象(Array-like Object)和转换
categories: [javascript]
---

# 类数组对象

类数组是有 length 属性，其他属性为非负整数的对象，不同的是类数组没有数组的方法，例如 push 等。

比如
```js
let person = {
    0: "小明",
    1: "18",
    length: 2
};
```


# 常见的类数组对象

- arguments对象
- NodeList（比如 document.getElementsByClassName('a') 得到的数据集
- typedArray


# 类数组对象和数组的区别：

类数组对象不能调用数组原型上的方法。比如：xx.push()、xx.slice()、xx.indexOf()。


# 类数组对象转为数组

- Array.prototype.slice.call(person, 0)/[].slice.call(person, 0)
- Array.from(person)

## Array.prototype.slice.call()

slice(start, end) 方法可从已有的数组中返回选定的元素。

start：必需。规定从何处开始选取。负数则为从数组尾部开始算起的位置。

end：可选。规定从何处结束选取。如果没有，则到数组结束。负数则为从数组尾部开始算起的位置。

为什么不直接用xx.slice()?

因为类数组对象xx没有数组的slice方法，需要用call。

Array.prototype.slice.call(arguments)能将具有length属性的对象转成数组，除了IE下的节点集合（因为ie下的dom对象是以com对象的形式实现的，js对象与com对象不能进行转换）

## Array.from()

ES6语法，可将类数组对象或可遍历对象转换成真正的数组。


