---
layout: mypost
title: chrome下font-size小于12px时失效问题
categories: [css]
---

Chrome做了如下限制：

- font-size 有一个最小值 12px（不同操作系统、不同语言可能限制不一样），低于 12px 的，一律按 12px 显示。理由是 Chrome 认为低于 12px 的中文对人类是不友好的。
- 但是允许你把 font-size 设置为 0.
- 这个 12px 的限制用户是可以自行调整的，进入 chrome://settings/fonts 设置，滚动到最下方你就可以调整 12px 为其他值。

**解决方案**

~~-webkit-text-size-adjust:none;~~

在新版本谷歌里已经失效

css3的transform:scale()
```css
p{
    font-size: 12px;
    -webkit-transform: scale(0.83333);
}
```
上文即12px缩放0.8333倍，大约等于10px的效果。但也会缩小其他一些属性(比如背景)，需要多测试或用其他方法规避。

所以不建议设置小于12px的值。
