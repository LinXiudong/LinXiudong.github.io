---
layout: mypost
title: async&await的使用及执行顺序
categories: [javascript]
---

# async
```
async function testAsync() {
    return "hello async";
}

const result = testAsync();
console.log(result

// Promise { 'hello async' }
```
async返回的是一个promise对象，所以也可以在后面接then()

# await
```
function getSomething() {
    return "something";
}

async function testAsync() {
    return Promise.resolve("hello async");
}

async function test() {
    const v1 = await getSomething();
    const v2 = await testAsync();
    console.log(v1, v2);
}

test();

// something hello async
```
await等的是一个返回值，如果返回值不是Promise对象， await 表达式的运算结果就是它等到的东西；如果返回值是Promise对象，await就会堵塞后面的代码，等着 Promise 对象 resolve，然后得到 resolve 的值，作为 await 表达式的运算结果。


# 做个简单的比较
一个 setTimeout 模拟耗时的异步操作

用Promise
```
function takeLongTime() {
    return new Promise(resolve => {
        setTimeout(() => resolve("long_time_value"), 1000);
    });
}

takeLongTime().then(v => {
    console.log("got", v);
});
```

用 async/await
```
function takeLongTime() {
    return new Promise(resolve => {
        setTimeout(() => resolve("long_time_value"), 1000);
    });
}

async function test() {
    const v = await takeLongTime();
    console.log(v);
}

test();
```
这里 takeLongTime() 没有申明为 async。因为 takeLongTime() 本身就是返回的 Promise 对象，加不加 async 结果都一样。

而这个两种方式优势并不明显，async/await 的优势要在处理由多个 Promise 组成的 then 链时才能体现出来。

假设一个业务，分多个步骤完成，每个步骤都是异步的，而且依赖于上一个步骤的结果。我们仍然用 setTimeout 来模拟异步操作：

```
function takeLongTime(n) {
    return new Promise(resolve => {
        setTimeout(() => resolve(n + 200), n);
    });
}

function step1(n) {
    console.log(`step1 with ${n}`);
    return takeLongTime(n);
}

function step2(n) {
    console.log(`step2 with ${n}`);
    return takeLongTime(n);
}

function step3(n) {
    console.log(`step3 with ${n}`);
    return takeLongTime(n);
}
```

现在用 Promise 方式来实现这三个步骤的处理
```
function doIt() {
    console.time("doIt");
    const time1 = 300;
    step1(time1)
        .then(time2 => step2(time2))
        .then(time3 => step3(time3))
        .then(result => {
            console.log(`result is ${result}`);
            console.timeEnd("doIt");
        });
}

doIt();

// c:\var\test>node --harmony_async_await .
// step1 with 300
// step2 with 500
// step3 with 700
// result is 900
// doIt: 1507.251ms
```

如果用 async/await 来实现
```
async function doIt() {
    console.time("doIt");
    const time1 = 300;
    const time2 = await step1(time1);
    const time3 = await step2(time2);
    const result = await step3(time3);
    console.log(`result is ${result}`);
    console.timeEnd("doIt");
}

doIt();
```
这个代码看起来是不是清晰得多，几乎跟同步代码一样


现在把业务要求改一下，仍然是三个步骤，但每一个步骤都需要之前每个步骤的结果。
```
function step1(n) {
    console.log(`step1 with ${n}`);
    return takeLongTime(n);
}

function step2(m, n) {
    console.log(`step2 with ${m} and ${n}`);
    return takeLongTime(m + n);
}

function step3(k, m, n) {
    console.log(`step3 with ${k}, ${m} and ${n}`);
    return takeLongTime(k + m + n);
}
```

这回先用 async/await 来写：
```
async function doIt() {
    console.time("doIt");
    const time1 = 300;
    const time2 = await step1(time1);
    const time3 = await step2(time1, time2);
    const result = await step3(time1, time2, time3);
    console.log(`result is ${result}`);
    console.timeEnd("doIt");
}

doIt();

// c:\var\test>node --harmony_async_await .
// step1 with 300
// step2 with 800 = 300 + 500
// step3 with 1800 = 300 + 500 + 1000
// result is 2000
// doIt: 2907.387ms
```
似乎和之前的示例没啥区别啊！别急，认真想想如果把它写成 Promise 方式实现会是什么样子？
```
function doIt() {
    console.time("doIt");
    const time1 = 300;
    step1(time1)
        .then(time2 => {
            return step2(time1, time2)
                .then(time3 => [time1, time2, time3]);
        })
        .then(times => {
            const [time1, time2, time3] = times;
            return step3(time1, time2, time3);
        })
        .then(result => {
            console.log(`result is ${result}`);
            console.timeEnd("doIt");
        });
}

doIt();
```
有没有感觉有点复杂的样子？那一堆参数处理，就是 Promise 方案的死穴—— 参数传递太麻烦了。


# async/await和promise的执行顺序

下面是一道面试题
```javascript
async function async1() {
    console.log("async1 start");
    await async2();
    console.log("async1 end");
}

async function async2() {
    console.log("async2");
}

console.log("script start");

setTimeout(function() {
    console.log("setTimeout");
}, 0);

async1();

new Promise(function(resolve) {
    console.log("promise1");
    resolve();
}).then(function() {
    console.log("promise2");
});

console.log("script end");
```

答案
```javascript
// script start
// async1 start
// async2
// promise1
// script end
// promise2
// async1 end
// setTimeout
```

## 一步步看清宏任务、微任务的执行过程

![p01][01]

「宏任务」、「微任务」都是队列。

一段代码执行时，会先执行宏任务中的同步代码

- 如果执行中遇到setTimeout之类宏任务，那么就把这个setTimeout内部的函数推入「宏任务的队列」中，下一轮宏任务执行时调用。
- 如果执行中遇到promise.then()之类的微任务，就会推入到「当前宏任务的微任务队列」中，在本轮宏任务的同步代码执行都完成后，依次执行所有的微任务1、2、3

上面的面试题代码

先console.log("script start")加入宏任务1队列并运行

setTimeout(...) 加入宏任务2队列

然后运行async1()，运行同步的console.log("async1 start")和async2中的console.log("async2")

这时async1中的await会等待async2返回，虽然是同步的，但还是会堵塞后面的代码，堵塞后执行async之外的代码

接着运行new Promise()中的console.log("promise1")，并把then()加入宏任务一的微任务队列

后面输出console.log("script end")

回到async内部，执行await Promise.resolve(undefined) （即await async2()）

并把then((undefined) => {})推入微任务队列

这时宏任务1的同步队列都运行结束，开始按顺序运行里面的微任务

then(function() {console.log("promise2");})

then((undefined) => {})

结束后运行await后面的函数console.log("async1 end")

最后运行宏任务2的console.log("setTimeout")

![p02][02]

[原文 8张图让你一步步看清async/await和promise的执行顺序](https://zhuanlan.zhihu.com/p/52000508)



[01]: 01.jpg
[02]: 02.jpg