---
layout: mypost
title: arguments对象
categories: [javascript]
---

在函数代码中，使用特殊对象 arguments，开发者无需明确指出参数名，就能访问它们。
```javascript
function say() {
    console.log(arguments[0]);
}
say(12, 11);	//12

function howManyArgs() {	//与括号内形参无关
    alert(arguments.length);
}
howManyArgs("string", 45);	//2
howManyArgs();	//0
```

arguments对象是比较特别的一个对象，实际上是当前函数的一个内置属性。arguments非常类似Array，但实际上又不是一个Array实例。可以通过如下代码得以证实：
```javascript
Array.prototype.testArg = "test";
function funcArg() {
    alert(funcArg.arguments.testArg);
    alert(funcArg.arguments[0]);
}
alert(new Array().testArg);	// result: "test"
funcArg(10);				// result: "undefined"  "10"
```
```javascript
//没参数时，a，b与argument同步，但c不与arguments[2]同步
function f(a, b, c){
    alert(c);                  // result: "undefined"
    c = 2012;
    alert(arguments[2]);       // result: "undefined"
}
f(1, 2);

// arguments.callee可调用函数本身，递归时推荐使用
function count(a){
	if(a == 1){
		return 1;
   	}
	return a + arguments.callee(--a);
}
var mm = count(10);
alert(mm);	//55
```
