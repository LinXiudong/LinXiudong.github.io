---
layout: mypost
title: webpack中node全局环境变量process.env的设置
categories: [webpack]
---

[文章来源](https://blog.csdn.net/greekmrzzj/article/details/83270543)


在看一些前框框架实现的源码的时候，经常会看到类似如下的代码：

```js
if (process.env.NODE_ENV === 'production') {
  module.exports = require('./prod.js')
} else {
  module.exports = require('./dev.js')
}
```

node中有全局变量process表示当前node进程，process（进程）其实就是存在node中的一个全局变量，process.env包含着关于系统环境的信息。但是process.env中并不存在NODE_ENV这个东西。其实NODE_ENV只是一个用户自定义的变量。

而具体 process.env.xxx 中的 xxx 是开发者自己定义的。比如：

```js
process.env.NODE_ENV
// 或者
process.env.VUE_CLI_DEBUG = true
process.env.PORT
```

# 设置环境变量

下面设置好后就可以使用process.env.NODE_ENV取到

## window 设置环境变量

> set NODE_ENV=dev

## Unix 设置环境变量

> export NODE_ENV=dev

## 直接在 js 代码中设置环境变量

> process.env.VUE_CLI_DEBUG = true

## package.json 中设置环境变量

```js
"scripts": {
  "start-win": "set NODE_ENV=dev && node app.js",
  "start-unix": "export NODE_ENV=dev && node app.js",
 }
```

# 解决 window 和 unix 命令不一致的问题

> 安装 npm i cross-env --save-dev

```js
"scripts": {
  "start-win": "cross-en NODE_ENV=dev && node app.js",
 }
```

# unix 永久设置环境变量

## 所有用户都生效

> vim /etc/profile

## 当前用户生效

> vim ~/.bash_profile

最后修改完成后需要运行如下语句令系统重新加载

## 修改/etc/profile文件后

> source /etc/profile

## 修改~/.bash_profile文件后

> source ~/.bash_profile
