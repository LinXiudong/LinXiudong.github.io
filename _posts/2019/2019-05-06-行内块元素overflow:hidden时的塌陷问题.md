---
layout: mypost
title: 行内块元素overflow:hidden时的塌陷问题
categories: [css]
---

# 问题
```html
<style>
.a {
    display:inline-block;
    overflow:hidden;
}
.b { 
    border-bottom:1px solid #000;
}
</style>

<span class="a">我是行内块</span>
<span class="b">我是行元素</span>

```
我们希望.a和.b是水平的，但实际情况是.b会塌陷下去

![01](01.png)

若把.b改为行内块(display: inline-block;)，结果依旧如此


# 解决

 添加vertical-align属性，值是top或者bottom都可以，可添加给任意一个（但不能给塌陷元素添加vertical-align:bottom;）

# 原因

因为实现隐藏功能的时候，隐藏部分的内容的 vertical-align 变成 baseline 对齐了，这样也导致行内块元素高度被撑高了。而后续的行内块元素跟行内元素，是接在了隐藏部分的 vertical-align 的高度上了。只要改回后续行内块元素跟行内元素的 vertical-align 值就可以了。

