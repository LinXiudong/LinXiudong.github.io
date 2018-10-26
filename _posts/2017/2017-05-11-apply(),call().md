---
layout: mypost
title: apply(),call()
categories: [javascript]
---


两者效果相同，不同的是传入参数的方法
```javascript
myFunc.call(obj,arg1,arg2,...); //call直接传入参数arg1,arg2,...
myFunc.apply(obj,[arg1,arg2,...]); //apply以数组形式传入参数[arg1,arg2,...]
```

obj——该对象将代替myFunc类里的this对象

arg1,arg2,...——作为参数传给myFunc


```javascript
//例1
var x = 0;
function test(){
    alert(this.x);
}
var o = {
    x: 1,
    m: test
};
o.m.apply(); //obj（第一个参数）为空时默认指向全局对象Global，返回0
//最后一句改为o.m.apply(o);则o.m的this指向对象o，返回1

//例2
/*定义一个人类*/
function Person(name, age){
    this.name = name;
}
/*定义一个学生类*/
function Student(name, age){
    Person.apply(this, arguments);
    this.age = age;
}
//创建一个学生类
var student = new Student("Bob", 13);
alert("name:" + student.name + " " + "age:" + student.age);
//在Student类中没有定义name属性，但可以通过apply取得该属性
```

**apply()的妙用**

apply可以将一个数组默认的转换为一个参数列表([param1,param2,param3]转换为param1,param2,param3)，这个如果让我们用程序来实现将数组的每一个项,来装换为参数的列表,可能都得费一会功夫,借助apply的这点特性,所以就有了以下高效率的方法:

a) Math.max可以实现得到数组中最大的一项（min同理）

因为Math.max参数里面不支持Math.max([param1,param2])，也就是数组，但是它支持Math.max(param1,param2,param3…),所以可以根据刚才apply的那个特点来解决：
```javascript
var max = Math.max.apply(null,array)
```
这样轻易的可以得到一个数组中最大的一项(apply会将一个数组转换为一个个参数并传递给方法)。

这块在调用的时候第一个参数给了一个null,这个是因为没有对象去调用这个方法,我只需要用这个方法帮我运算,得到返回的结果就行,所以直接传递了一个null过去。

b) Array.prototype.push可以实现两个数组合并

同样push方法没有提供push一个数组,但是它提供了push(param1,param,…paramN) 所以同样也可以通过apply来转换一下这个数组,即:
```javascript
vararr1 = new Array("1","2","3");
vararr2 = new Array("4","5","6");
Array.prototype.push.apply(arr1,arr2);
```
也可以这样理解，arr1调用了push方法,参数是通过apply将数组转换为参数列表的集合。


**什么情况下可以使用其妙用？**

一般在目标函数只需要n个参数列表,而不接收一个数组的形式，可以通过apply的方式巧妙地解决这个问题！
