---
layout: mypost
title: html+css设置head固定body可滚动的表格
categories: [html]
---

```html
<!--
	把头部和内容分为两个table放在两个div中，因为tbody直接设置overflow无效，
	同时如果放在一个div中，滚动时头部也会滚动，用户体验不好
 -->
<div class="thead tDiv">
	<table>
		<thead>
			<tr>
				<td>日期</td>
				<td>编号</td>
				<td>姓名</td>
			</tr>
		</thead>
	</table>
</div>
<div class="tbody tDiv">
	<table>
		<tbody>
			<tr>
				<td>2018-01-01</td>
				<td>1101</td>
				<td>aaa</td>
			</tr>
			<tr>
				<td>2018-01-02</td>
				<td>1102</td>
				<td>bbb</td>
			</tr>
			<tr>
				<td>2018-01-03</td>
				<td>1103</td>
				<td>ccc</td>
			</tr>
		</tbody>
	</table>
</div>
```
```css
.tDiv{
	width: 800px;
	/* 如果不设置下面属性，设置padding会导致溢出，里面的table宽度不变 */
	box-sizing: border-box;
}
table{
	width: 100%;
	border-collapse:collapse;	/*单元格之间无间隙*/
}
/*分为两个div两个table后head和body无法对齐，这里设置宽度对齐*/
table tr td:first-child{
	width: 30%;
}
table tr td + td{
	width: 30%;
}
table tr td + td + td{
	width: 40%;
}
table tr td{
	border: 1px solid black;

}
.tbody tbody tr:first-child > td{
	border-top: none;	/*防止head和body的边框重合导致更长*/
}

.tbody{
	height: 100px;	/*限制高度*/
	overflow: auto;
	/*  上面设置后body会出现滚动条，导致body宽度变小，
		此时虽然设置了百分比宽度，也会和head对不齐，
		所以要把head的宽度也变小 */
}
.thead{
	padding-right: 17px;
}
```
