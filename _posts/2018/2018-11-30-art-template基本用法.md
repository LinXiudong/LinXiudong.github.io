---
layout: mypost
title: art-template基本用法
categories: [模板语法]
---

[来源及更多使用](https://aui.github.io/art-template/zh-cn/docs/index.html)

由于编译问题，下文统一使用_{_{和_}_}代替没有下划线的情况

# 普通使用
```
_{_{value_}_}
_{_{data.key_}_}
_{_{data['key']_}_}
_{_{a ? b : c_}_}
_{_{a || b_}_}
_{_{a + b_}_}
```
模板一级特殊变量可以使用 $data 加下标的方式访问：
```
_{_{$data['user list']_}_}
```
原文输出
```
_{_{@ value _}_}
```
> 原文输出语句不会对 HTML 内容进行转义处理，可能存在安全风险，请谨慎使用。


# 条件
```
_{_{if v1_}_} ... _{_{else if v2_}_} ... _{_{/if_}_}
```


# 循环
```
_{_{each target_}_}
    _{_{$index_}_} _{_{$value_}_}
_{_{/each_}_}
//target 支持 array 与 object 的迭代，其默认值为 $data。
//$value 与 $index 可以自定义：_{_{each target val key_}_}。
```

# 变量
```
_{_{set temp = data.sub.content_}_}
```

# 模板继承
```
_{_{extend './layout.art'_}_}
_{_{block 'head'_}_} ... _{_{/block_}_}
```
demo
```
<!--layout.art-->
<!doctype html>
<html>
    <head>
        <meta charset="utf-8">
        <title>_{_{block 'title'_}_}My Site_{_{/block_}_}</title>

        _{_{block 'head'_}_}
        <link rel="stylesheet" href="main.css">
        _{_{/block_}_}
    </head>
    <body>
        _{_{block 'content'_}_}_{_{/block_}_}
    </body>
</html>
<!--index.art-->
_{_{extend './layout.art'_}_}

_{_{block 'title'_}_}_{_{title_}_}_{_{/block_}_}

_{_{block 'head'_}_}
    <link rel="stylesheet" href="custom.css">
_{_{/block_}_}

_{_{block 'content'_}_}
    <p>This is just an awesome page.</p>
_{_{/block_}_}
```

# 子模板
```
_{_{include './header.art'_}_}
_{_{include './header.art' data_}_}
//data 数默认值为 $data；标准语法不支持声明 object 与 array，只支持引用变量。
//art-template 内建 HTML 压缩器，请避免书写 HTML 非正常闭合的子模板，否则开启压缩后标签可能会被意外“优化”。
```

# 过滤器
注册过滤器
```
template.defaults.imports.dateFormat = function(date, format){/*[code..]*/};
template.defaults.imports.timestamp = function(value){return value * 1000};
```
过滤器函数第一个参数接受目标值。

标准语法
```
_{_{date | timestamp | dateFormat 'yyyy-MM-dd hh:mm:ss'_}_}
//_{_{value | filter_}_} 过滤器语法类似管道操作符，它的上一个输出作为下一个输入。
```
