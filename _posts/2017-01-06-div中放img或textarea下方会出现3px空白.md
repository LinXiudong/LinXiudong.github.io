---
layout: mypost
title: div中放img或textarea下方会出现3px空白
categories: [css]
---

在一个div里放一个img，但下方会出现一小块空白。

css
```css
div{
    display: inline-block;
    border: 1px solid black;
}
img{
    width: 300px;
    height: 300px;
}
```
html
```html
<div>
    <img src="1.jpg">
</div>
```

![此处输入图片的描述][1]

*并非padding、margin等边距问题，当内外边距都为0时，还是会有3px的空白。*

- 解决一：

因为img是内联元素，div是块元素，浏览器解析时会在img下预留高度，而且火狐是5px，谷歌是3px。

直接把img变成块元素就可以了，即给其一个 ***display:block;*** 属性。

- 解释二：

通过google了解到，img是一种类似text的元素，在结束的时候，会在末尾加上一个空白符，所以会多3px。

因此如果div里面放textarea同样会出现这个问题。

![此处输入图片的描述][2]

- 解决方法二：***vertical-align:middle;***


上面两种解决方法都对textarea和img有效。



[1]: 01.png
[2]: 02.png
