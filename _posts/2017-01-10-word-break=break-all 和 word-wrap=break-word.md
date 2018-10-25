---
layout: mypost
title: word-break:break-all 和 word-wrap:break-word
categories: [css]
---

word-break属性

有以下值：

- normal  使用默认的换行规则。
- break-all  允许任意非CJK(Chinese/Japanese/Korean)文本间的单词断行。
- keep-all  不允许CJK(Chinese/Japanese/Korean)文本中的单词换行，只能在半角空格或连字符处换行。非CJK文本的行为实际上和normal一致。

word-wrap属性

有以下值：

- normal  正常的换行规则。
- break-word  一行单词中实在没有其他靠谱的换行点的时候换行。

word-wrap属性由于和word-break长得太像，难免会让人记不住搞混淆，于是在CSS3规范里，把这个属性的名称给改了，叫做：**overflow-wrap**

*但兼容性较差，只有Chrome/Safari等WebKit/Blink浏览器支持。为了兼容使用，还是乖乖使用word-wrap吧。*

![此处输入图片的描述][1]
可以发现，word-break:break-all正如其名，所有的都换行。毫不留情，一点空隙都不放过。而word-wrap:break-word则带有怜悯之心，如果这一行文字有可以换行的点，如空格，或CJK(Chinese/Japanese/Korean)(中文/日文/韩文)之类的，则就不打英文单词或字符的主意了，让这些换行点换行。

*另外还有两个很像：word-spacing是单词之间间距，white-space是字符是否换行显示。*


[1]: 01.png
