---
layout: mypost
title: font-weight在不同浏览器下的显示差异
categories: [css]
---

```css
    p {
	  font-size: 50px;
	  /* font-family: Arial; */
	}
	p.fw1 {font-weight: 100;}
	p.fw2 {font-weight: 200;}
	p.fw3 {font-weight: 300;}
	p.fw4 {font-weight: 400;}	/* 400 等同于 normal */
	p.fw5 {font-weight: 500;}
	p.fw6 {font-weight: 600;}
	p.fw7 {font-weight: 700;}	/* 700 等同于 bold */
	p.fw8 {font-weight: 800;}
	p.fw9 {font-weight: 900;}
```
```html
    <p class="fw1">This is a paragraph</p>
	<p class="fw2">This is a paragraph</p>
	<p class="fw3">This is a paragraph</p>
	<p class="fw4">This is a paragraph</p>
	<p class="fw5">This is a paragraph</p>
	<p class="fw6">This is a paragraph</p>
	<p class="fw7">This is a paragraph</p>
	<p class="fw8">This is a paragraph</p>
	<p class="fw9">This is a paragraph</p>
```
下图为各浏览器的效果:

edge、safari(windows)
![edge、safari(windows)][1]

火狐、chrome
![火狐、chrome][2]

原因是各浏览器的默认字体不同，导致显示效果不同。如果上文的css放开font-family: Arial，采用统一字体，效果如下:

edge、chrome、火狐
![edge、chrome、火狐][3]

*但在safari for windows浏览器下，600/700和800/900的字体还是一样的。原以为是由于某些原因字体没有设置上，但对比safari下设置了font-family和没设置的，显示效果是不一样的，说明应该是设置上了。但为什么和其他浏览器存在差异暂且不明。*

**font-weight和字体的关系**

每种字体都有字重，且一般字体都没有100-900九个字重，只会有其中几个。其中400(normal)和700(bold)两个是必备的。

下面是某字体的字重表：

![此处输入图片的描述][4]
其中黑色表示有该字重，灰色表示没有。即该字体只有300、600两种字重。

设置font-weight后，若字体有该字重，设置该字重；若没有，以以下规则匹配：

- 如果所需的字重小于400，则首先降序检查小于所需字重的各个字重，如仍然没有，则升序检查大于所需字重的各字重，直到找到匹配的字重。
- 如果所需的字重大于500，则首先升序检查大于所需字重的各字重，之后降序检查小于所需字重的各字重，直到找到匹配的字重。
- 如果所需的字重是400，那么会优先匹配500对应的字重，如仍没有，那么执行第一条所需字重小于400的规则。
- 如果所需的字重是500，则优先匹配400对应的字重，如仍没有，那么执行第二条所需字重大于500的规则。


[1]: 01.png
[2]: 02.png
[3]: 03.png
[4]: 04.png
