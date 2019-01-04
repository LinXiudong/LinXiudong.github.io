---
layout: mypost
title: async、await的使用（二）
categories: [javascript]
---

之前已经有过async的使用文章，这篇是一些补充，内容来自阮一峰的es6教程

# 简介

async是Generator的语法糖，主要改进有4点

（1）内置执行器。

Generator 函数的执行必须靠执行器，而async函数自带执行器。也就是说，async函数的执行，与普通函数一模一样，只要一行。

asyncReadFile();

上面的代码调用了asyncReadFile函数，然后它就会自动执行，输出最后结果。这完全不像 Generator 函数，需要调用next方法，才能真正执行，得到最后结果。

（2）更好的语义。

async和await，比起星号和yield，语义更清楚了。

（3）更广的适用性。

co模块约定，yield命令后面只能是 Thunk 函数或 Promise 对象，而async函数的await命令后面，可以是 Promise 对象和原始类型的值（数值、字符串和布尔值，但这时等同于同步操作）。

（4）返回值是 Promise。

async函数的返回值是 Promise 对象，这比 Generator 函数的返回值是 Iterator 对象方便多了。你可以用then方法指定下一步的操作。


# async 函数有多种使用形式。

```
// 函数声明
async function foo() {}

// 函数表达式
const foo = async function () {};

// 对象的方法
let obj = { async foo() {} };
obj.foo().then(...)

// Class 的方法
class Storage {
  constructor() {
    this.cachePromise = caches.open('avatars');
  }

  async getAvatar(name) {
    const cache = await this.cachePromise;
    return cache.match(`/avatars/${name}.jpg`);
  }
}

const storage = new Storage();
storage.getAvatar('jake').then(…);

// 箭头函数
const foo = async () => {};
```

# await

await命令后面是一个thenable对象（即定义then方法的对象），那么await会将其等同于 Promise 对象。
```
class Sleep {
  constructor(timeout) {
    this.timeout = timeout;
  }
  then(resolve, reject) {
    const startTime = Date.now();
    setTimeout(
      () => resolve(Date.now() - startTime),
      this.timeout
    );
  }
}

(async () => {
  const actualTime = await new Sleep(1000);
  console.log(actualTime);
})();
```
上面代码中，await命令后面是一个Sleep对象的实例。这个实例不是 Promise 对象，但是因为定义了then方法，await会将其视为Promise处理。


await命令后面的 Promise 对象如果变为reject状态，则reject的参数会被catch方法的回调函数接收到。
```
async function f() {
  await Promise.reject('出错了');
}

f()
.then(v => console.log(v))
.catch(e => console.log(e))
// 出错了
```
注意，上面代码中，await语句前面没有return，但是reject方法的参数依然传入了catch方法的回调函数。这里如果在await前面加上return，效果是一样的。

任何一个await语句后面的 Promise 对象变为reject状态，那么整个async函数都会中断执行。

有时，我们希望这时不要中断后面的异步操作。这时可以将第一个await放在try...catch结构里面，这样不管这个异步操作是否成功，第二个await都会执行。

另一种方法是await后面的 Promise 对象再跟一个catch方法，内部处理前面可能出现的错误。
```
async function f() {
  await Promise.reject('出错了')
    .catch(e => console.log(e));
  return await Promise.resolve('hello world');
}
```


# 使用注意点

第一点，前面已经说过，await命令后面的Promise对象，运行结果可能是rejected，所以最好把await命令放在try...catch代码块中。
```
async function myFunction() {
  try {
    await somethingThatReturnsAPromise();
  } catch (err) {
    console.log(err);
  }
}

// 另一种写法

async function myFunction() {
  await somethingThatReturnsAPromise()
  .catch(function (err) {
    console.log(err);
  });
}
```

第二点，多个await命令后面的异步操作，如果不存在继发关系，最好让它们同时触发。
```
let foo = await getFoo();
let bar = await getBar();
```
上面代码中，getFoo和getBar是两个独立的异步操作（即互不依赖），被写成继发关系。这样比较耗时，因为只有getFoo完成以后，才会执行getBar，完全可以让它们同时触发。
```
// 写法一
let [foo, bar] = await Promise.all([getFoo(), getBar()]);

// 写法二
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
```
上面两种写法，getFoo和getBar都是同时触发，这样就会缩短程序的执行时间。

第三点，await命令只能用在async函数之中，如果用在普通函数，就会报错。
```
async function dbFuc(db) {
  let docs = [{}, {}, {}];

  // 报错
  docs.forEach(function (doc) {
    await db.post(doc);
  });
}
```
上面代码会报错，因为await用在普通函数之中了。但是，如果将forEach方法的参数改成async函数，也有问题。

第四点，async 函数可以保留运行堆栈。
```
const a = () => {
  b().then(() => c());
};
```
上面代码中，函数a内部运行了一个异步任务b()。当b()运行的时候，函数a()不会中断，而是继续执行。等到b()运行结束，可能a()早就运行结束了，b()所在的上下文环境已经消失了。如果b()或c()报错，错误堆栈将不包括a()。

现在将这个例子改成async函数。
```
const a = async () => {
  await b();
  c();
};
```
上面代码中，b()运行的时候，a()是暂停执行，上下文环境都保存着。一旦b()或c()，错误堆栈将包括a()。


# 异步遍历器

Iterator 接口是一种数据遍历的协议，只要调用遍历器对象的next方法，就会得到一个对象，表示当前遍历指针所在的那个位置的信息。next方法返回的对象的结构是{value, done}，其中value表示当前的数据的值，done是一个布尔值，表示遍历是否结束。

这里隐含着一个规定，next方法必须是同步的，只要调用就必须立刻返回值。也就是说，一旦执行next方法，就必须同步地得到value和done这两个属性。如果遍历指针正好指向同步操作，当然没有问题，但对于异步操作，就不太合适了。目前的解决方法是，Generator 函数里面的异步操作，返回一个 Thunk 函数或者 Promise 对象，即value属性是一个 Thunk 函数或者 Promise 对象，等待以后返回真正的值，而done属性则还是同步产生的。

ES2018 引入了”异步遍历器“（Async Iterator），为异步操作提供原生的遍历器接口，即value和done这两个属性都是异步产生。

## 异步遍历的接口

异步遍历器的最大的语法特点，就是调用遍历器的next方法，返回的是一个 Promise 对象。
```
asyncIterator
  .next()
  .then(
    ({ value, done }) => /* ... */
  );
```
上面代码中，asyncIterator是一个异步遍历器，调用next方法以后，返回一个 Promise 对象。因此，可以使用then方法指定，这个 Promise 对象的状态变为resolve以后的回调函数。回调函数的参数，则是一个具有value和done两个属性的对象，这个跟同步遍历器是一样的。

我们知道，一个对象的同步遍历器的接口，部署在Symbol.iterator属性上面。同样地，对象的异步遍历器接口，部署在Symbol.asyncIterator属性上面。不管是什么样的对象，只要它的Symbol.asyncIterator属性有值，就表示应该对它进行异步遍历。

下面是一个异步遍历器的例子。
```
const asyncIterable = createAsyncIterable(['a', 'b']);
const asyncIterator = asyncIterable[Symbol.asyncIterator]();

asyncIterator
	.next()
	.then(iterResult1 => {
		console.log(iterResult1); // { value: 'a', done: false }
		return asyncIterator.next();
	})
	.then(iterResult2 => {
		console.log(iterResult2); // { value: 'b', done: false }
		return asyncIterator.next();
	})
	.then(iterResult3 => {
		console.log(iterResult3); // { value: undefined, done: true }
	});
```
上面代码中，异步遍历器其实返回了两次值。第一次调用的时候，返回一个 Promise 对象；等到 Promise 对象resolve了，再返回一个表示当前数据成员信息的对象。这就是说，异步遍历器与同步遍历器最终行为是一致的，只是会先返回 Promise 对象，作为中介。

由于异步遍历器的next方法，返回的是一个 Promise 对象。因此，可以把它放在await命令后面。
```
async function f() {
  const asyncIterable = createAsyncIterable(['a', 'b']);
  const asyncIterator = asyncIterable[Symbol.asyncIterator]();
  console.log(await asyncIterator.next());
  // { value: 'a', done: false }
  console.log(await asyncIterator.next());
  // { value: 'b', done: false }
  console.log(await asyncIterator.next());
  // { value: undefined, done: true }
}
```
上面代码中，next方法用await处理以后，就不必使用then方法了。整个流程已经很接近同步处理了。

注意，异步遍历器的next方法是可以连续调用的，不必等到上一步产生的 Promise 对象resolve以后再调用。这种情况下，next方法会累积起来，自动按照每一步的顺序运行下去。下面是一个例子，把所有的next方法放在Promise.all方法里面。
```
const asyncIterable = createAsyncIterable(['a', 'b']);
const asyncIterator = asyncIterable[Symbol.asyncIterator]();
const [{value: v1}, {value: v2}] = await Promise.all([
  asyncIterator.next(), asyncIterator.next()
]);

console.log(v1, v2); // a b
```
另一种用法是一次性调用所有的next方法，然后await最后一步操作。
```
async function runner() {
  const writer = openFile('someFile.txt');
  writer.next('hello');
  writer.next('world');
  await writer.return();
}

runner();
```

# for await...of

前面介绍过，for...of循环用于遍历同步的 Iterator 接口。新引入的for await...of循环，则是用于遍历异步的 Iterator 接口。
```
async function f() {
  for await (const x of createAsyncIterable(['a', 'b'])) {
    console.log(x);
  }
}
// a
// b
```
上面代码中，createAsyncIterable()返回一个拥有异步遍历器接口的对象，for...of循环自动调用这个对象的异步遍历器的next方法，会得到一个 Promise 对象。await用来处理这个 Promise 对象，一旦resolve，就把得到的值（x）传入for...of的循环体。

for await...of循环的一个用途，是部署了 asyncIterable 操作的异步接口，可以直接放入这个循环。
```
let body = '';

async function f() {
  for await(const data of req) body += data;
  const parsed = JSON.parse(body);
  console.log('got', parsed);
}
```
上面代码中，req是一个 asyncIterable 对象，用来异步读取数据。可以看到，使用for await...of循环以后，代码会非常简洁。

如果next方法返回的 Promise 对象被reject，for await...of就会报错，要用try...catch捕捉。


# 异步 Generator 函数

就像 Generator 函数返回一个同步遍历器对象一样，异步 Generator 函数的作用，是返回一个异步遍历器对象。

在语法上，异步 Generator 函数就是async函数与 Generator 函数的结合。
```
async function* gen() {
  yield 'hello';
}
const genObj = gen();
genObj.next().then(x => console.log(x));
// { value: 'hello', done: false }
```
上面代码中，gen是一个异步 Generator 函数，执行后返回一个异步 Iterator 对象。对该对象调用next方法，返回一个 Promise 对象。

异步遍历器的设计目的之一，就是 Generator 函数处理同步操作和异步操作时，能够使用同一套接口。
```
// 同步 Generator 函数
function* map(iterable, func) {
  const iter = iterable[Symbol.iterator]();
  while (true) {
    const {value, done} = iter.next();
    if (done) break;
    yield func(value);
  }
}

// 异步 Generator 函数
async function* map(iterable, func) {
  const iter = iterable[Symbol.asyncIterator]();
  while (true) {
    const {value, done} = await iter.next();
    if (done) break;
    yield func(value);
  }
}
```
上面代码中，map是一个 Generator 函数，第一个参数是可遍历对象iterable，第二个参数是一个回调函数func。map的作用是将iterable每一步返回的值，使用func进行处理。上面有两个版本的map，前一个处理同步遍历器，后一个处理异步遍历器，可以看到两个版本的写法基本上是一致的。

下面是另一个异步 Generator 函数的例子。
```
async function* readLines(path) {
  let file = await fileOpen(path);

  try {
    while (!file.EOF) {
      yield await file.readLine();
    }
  } finally {
    await file.close();
  }
}
```
上面代码中，异步操作前面使用await关键字标明，即await后面的操作，应该返回 Promise 对象。凡是使用yield关键字的地方，就是next方法停下来的地方，它后面的表达式的值（即await file.readLine()的值），会作为next()返回对象的value属性，这一点是与同步 Generator 函数一致的。

异步 Generator 函数内部，能够同时使用await和yield命令。可以这样理解，await命令用于将外部操作产生的值输入函数内部，yield命令用于将函数内部的值输出。

上面代码定义的异步 Generator 函数的用法如下。
```
(async function () {
  for await (const line of readLines(filePath)) {
    console.log(line);
  }
})()
```

异步 Generator 函数可以与for await...of循环结合起来使用。
```
async function* prefixLines(asyncIterable) {
  for await (const line of asyncIterable) {
    yield '> ' + line;
  }
}
```
异步 Generator 函数的返回值是一个异步 Iterator，即每次调用它的next方法，会返回一个 Promise 对象，也就是说，跟在yield命令后面的，应该是一个 Promise 对象。如果像上面那个例子那样，yield命令后面是一个字符串，会被自动包装成一个 Promise 对象。
```
function fetchRandom() {
  const url = 'https://www.random.org/decimal-fractions/'
    + '?num=1&dec=10&col=1&format=plain&rnd=new';
  return fetch(url);
}

async function* asyncGenerator() {
  console.log('Start');
  const result = await fetchRandom(); // (A)
  yield 'Result: ' + await result.text(); // (B)
  console.log('Done');
}

const ag = asyncGenerator();
ag.next().then(({value, done}) => {
  console.log(value);
})
```
上面代码中，ag是asyncGenerator函数返回的异步遍历器对象。调用ag.next()以后，上面代码的执行顺序如下。

- ag.next()立刻返回一个 Promise 对象。
- asyncGenerator函数开始执行，打印出Start。
- await命令返回一个 Promise 对象，asyncGenerator函数停在这里。
- A 处变成 fulfilled 状态，产生的值放入result变量，asyncGenerator函数继续往下执行。
- 函数在 B 处的yield暂停执行，一旦yield命令取到值，ag.next()返回的那个 Promise 对象变成 fulfilled 状态。
- ag.next()后面的then方法指定的回调函数开始执行。该回调函数的参数是一个对象{value, done}，其中value的值是yield命令后面的那个表达式的值，done的值是false。

A 和 B 两行的作用类似于下面的代码。
```
return new Promise((resolve, reject) => {
  fetchRandom()
  .then(result => result.text())
  .then(result => {
     resolve({
       value: 'Result: ' + result,
       done: false,
     });
  });
});
```

如果异步 Generator 函数抛出错误，会导致 Promise 对象的状态变为reject，然后抛出的错误被catch方法捕获。
```
async function* asyncGenerator() {
  throw new Error('Problem!');
}

asyncGenerator()
.next()
.catch(err => console.log(err)); // Error: Problem!
```

注意，普通的 async 函数返回的是一个 Promise 对象，而异步 Generator 函数返回的是一个异步 Iterator 对象。可以这样理解，async 函数和异步 Generator 函数，是封装异步操作的两种方法，都用来达到同一种目的。区别在于，前者自带执行器，后者通过for await...of执行，或者自己编写执行器。下面就是一个异步 Generator 函数的执行器。

```
async function takeAsync(asyncIterable, count = Infinity) {
  const result = [];
  const iterator = asyncIterable[Symbol.asyncIterator]();
  while (result.length < count) {
    const {value, done} = await iterator.next();
    if (done) break;
    result.push(value);
  }
  return result;
}
```
上面代码中，异步 Generator 函数产生的异步遍历器，会通过while循环自动执行，每当await iterator.next()完成，就会进入下一轮循环。一旦done属性变为true，就会跳出循环，异步遍历器执行结束。

下面是这个自动执行器的一个使用实例。
```
async function f() {
  async function* gen() {
    yield 'a';
    yield 'b';
    yield 'c';
  }

  return await takeAsync(gen());
}

f().then(function (result) {
  console.log(result); // ['a', 'b', 'c']
})
```

异步 Generator 函数出现以后，JavaScript 就有了四种函数形式：普通函数、async 函数、Generator 函数和异步 Generator 函数。请注意区分每种函数的不同之处。基本上，如果是一系列按照顺序执行的异步操作（比如读取文件，然后写入新内容，再存入硬盘），可以使用 async 函数；如果是一系列产生相同数据结构的异步操作（比如一行一行读取文件），可以使用异步 Generator 函数。

异步 Generator 函数也可以通过next方法的参数，接收外部传入的数据。
```
const writer = openFile('someFile.txt');
writer.next('hello'); // 立即执行
writer.next('world'); // 立即执行
await writer.return(); // 等待写入结束
```
上面代码中，openFile是一个异步 Generator 函数。next方法的参数，向该函数内部的操作传入数据。每次next方法都是同步执行的，最后的await命令用于等待整个写入操作结束。

最后，同步的数据结构，也可以使用异步 Generator 函数。
```
async function* createAsyncIterable(syncIterable) {
  for (const elem of syncIterable) {
    yield elem;
  }
}
```
上面代码中，由于没有异步操作，所以也就没有使用await关键字。


# yield* 语句

yield*语句也可以跟一个异步遍历器。
```
async function* gen1() {
  yield 'a';
  yield 'b';
  return 2;
}

async function* gen2() {
  // result 最终会等于 2
  const result = yield* gen1();
}
```
上面代码中，gen2函数里面的result变量，最后的值是2。

与同步 Generator 函数一样，for await...of循环会展开yield*。
```
(async function () {
  for await (const x of gen2()) {
    console.log(x);
  }
})();
// a
// b
```