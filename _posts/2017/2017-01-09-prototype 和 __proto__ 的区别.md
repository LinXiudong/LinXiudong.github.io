---
layout: mypost
title: prototype 和 __proto__ 的区别
categories: [javascript]
---

### \_\_proto\_\_
```javascript
function A(name) {  // 这里是一个构造函数
    thia.name = name
}
var Aobj = {    //这里是一个对象字面量
    name: ''
}
// 我们分别打印出来这两个对象看看
console.dir(A)
console.dir(Aobj)
```
![此处输入图片的描述][1]
这里我们可以很明显的看到

构造函数的\_\_proto\_\_属性指向了function()

对象字面量的\_\_proto\_\_属性指向了Object


为什么指向的是不一样的呢？

因为构造函数本身也是一个函数，所以它的原型指向function()

而对象字面量是一个对象，那么他的原型肯定是指向Object

扩展思考，如果是一个数组对象，那么它的__proto__会指向什么？

```javascript
const arr = [112,22,3333]
console.dir(arr)
```
![此处输入图片的描述][2]

没错，这里的\_\_proto\_\_就指向了Array[0]


总结：一个对象的\_\_proto\_\_属性和自己的内部属性[[Prototype]]指向一个相同的值 (通常称这个值为原型)

*tips：firefox、chrome等浏览器把对象内部属性[[Prototype]]用\_\_proto\_\_的形式暴露了出来.(老版本的IE并不支持\_\_proto\_\_,IE11中已经加上了\_\_proto\_\_属性)*

### prototype
看看上面的截图，你会发现只有构造函数中有这个玩意儿，prototype确实是在function中特有,别的对象类型中都不会有的属性。
![此处输入图片的描述][3]

我们在看这个function对象属性的时候就会发现这么一个prototype的属性，它的值是一个Object。点开这个obj我们就会发现啊，这个obj的constructor属性指向了这个构造函数本身。


在new的时候，构造函数就执行了一次。

new到底发生了什么
```javascript
//对于 var b = new B('testb')
//js 实际上执行的是：
var o = new Object();   // 生成一个新的对象b，这里可以约等于var b = {}
o.__proto__ = B.prototype; // 这里就是函数对象中独有的prototype属性。
// 这个独有的prototype属性 包含了一个constructor 属性方法，指向的就是构造函数，也就是这里的function B(name){}
B.call(o);
// 由于call的使用将这里this是指向o, 所以就可以把什么this.name/getName 强行的绑定到o上。同时，需要注意的一点是， 这里的构造函数执行一遍，只不过是将 this指向的属性和方法，都强行的给新创建的这个o对象绑定了一遍。
var b = o;
// 把这个o返回给了b。从而完成了var b = new B('testb')的过程
```
至于js为什么要把新建对象的原型指向构造函数的prototype属性。

我们可以这样来理解。因为通过new方法来创建的obj。肯定是需要一个标记来找到自己的构造器函数。

所以为了让整个程序结构看上去合理。我们需要把新建对象的原型指向构造函数的prototype属性。


所以到最后，我们总结一下。

在 javascript 中 prototype 和 proto 到底有什么区别。

prototype 是面向构造函数来思考，

proto 是面向实例化后的对象来思考就对了。


[1]: 01.png
[2]: 02.png
[3]: 03.png
