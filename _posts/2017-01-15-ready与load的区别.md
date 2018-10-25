---
layout: mypost
title: ready与load的区别
categories: [jQuery]
---

- load

主要是用来代替原生的window.onload，它只能用在两个场景下：

1. window对象上。比如
```javascript
$(window).load(fn);
```
2. 带有URL的元素(images, scripts, frames, iframes)。比如
```javascript
$('img').load(fn);
```
除此之外，任何元素都没有load事件，比如
```javascript
$(document).load(fn);   //这是错误的写法，根本不会执行
```

load事件需要页面完全加载完成才会触发，所谓完全加载完，不仅仅是dom结构加载完，还需要所有的链接引用都加载完。比如页面中有大量图片，必须等每一个图片都加载完成。


**兼容性**

jQuery官方文档明确说明load事件的跨浏览器兼容性很差，谷歌浏览器仅仅支持
```javascript
$(window).load(fn);
```
而火狐浏览器支持
```javascript
$(window).load(fn);
$('img').load(fn);
//等等
```
***所以，除非必要情况下，否则强烈不推荐使用load事件。***

- ready

可以加在任意元素上，比如
```javascript
$(window).ready(fn);
$(document).ready(fn);
$('div').ready(fn);
//等等
```
ready事件不要求页面完全加载完，只需要加载完dom结构即可触发。


**运行顺序**
```javascript
$(document).ready();    //可以出现多次
$(window).load();   //(划掉)只能出现一次，如果有多个，只运行最后一个。
//但实际demo操作中多个load全部运行了……判断是网上的文档有误
window.onload; //只会运行最后一个。
//window.onload与$(window).load()即js写法和jq写法
```
demo
```javascript
//下面两段代码都会执行
$(document).ready(function(){
    alert("test1");
});
$(document).ready(function(){
    alert("test2");
});
//按论坛内容，只会运行第二段代码。但实际操作时下面两段代码都会执行，判断是论坛内容有误
$(window).load(function(){
    alert("test1");
});
$(window).load(function(){
    alert("test2");
});
//onload 只有第二段代码会执行
window.onload = function(){
    alert("text1");
};
window.onload = function(){
    alert("text2");
};
//下面会按顺序执行，输出顺序为window，document，div
//如果打乱顺序，会按打乱后的顺序执行
$(window).ready(function(){
    alert("window");
});
$(document).ready(function(){
    alert("document");
});
$("div").ready(function(){
    alert("div");
});
```
