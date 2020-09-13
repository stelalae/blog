# async 函数

[ECMAScript 6 入门 - async 函数](http://es6.ruanyifeng.com/#docs/async)

[toc]

### 含义

ES2017 标准引入了 async 函数，使得异步操作变得更加方便。async 函数是 Generator 函数的语法糖，就是将 Generator 函数的星号（\*）替换成async，将yield替换成await，仅此而已。改进点体现在以下四点。
（1）内置执行器。
Generator 函数的执行必须靠执行器，所以才有了co模块，而async函数自带执行器。即async函数的执行，与普通函数一模一样，只要一行。不像 Generator 函数，需要调用next方法，或者用co模块，才能真正执行，得到最后结果。
（2）更好的语义。
async和await，比起星号和yield，语义更清楚。
（3）更广的适用性。
co模块约定，yield命令后面只能是 Thunk 函数或 Promise 对象，而async函数的await命令后面，可以是 Promise 对象和原始类型的值（数值、字符串和布尔值，但这时会自动转成立即 resolved 的 Promise 对象）。
（4）返回值是 Promise。
async函数的返回值是 Promise 对象，这比 Generator 函数的返回值是 Iterator 对象方便多了。你可以用then方法指定下一步的操作。

**进一步说，async函数完全可以看作多个异步操作，包装成的一个 Promise 对象，而await命令就是内部then命令的语法糖。**

### 基本用法

async函数返回一个 Promise 对象，可以使用then方法添加回调函数。当函数执行的时候，一旦遇到await就会先返回，等到异步操作完成，再接着执行函数体内后面的语句。
```JavaScript
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


### 语法

async函数的语法规则总体上比较简单，难点是错误处理机制。

async函数返回一个 Promise 对象，内部return语句返回的值会成为then方法回调函数的参数，内部抛出错误会导致返回的 Promise 对象变为reject状态。抛出的错误对象会被catch方法回调函数接收到。

只有async函数内部的异步操作执行完，才会执行then方法指定的回调函数，或抛出异常后执行catch方法。

正常情况下，await命令后面是一个 Promise 对象，返回该对象的结果。如果不是 Promise 对象，就直接返回对应的值。如果是一个thenable对象（即定义then方法的对象），那么await会将其等同于 Promise 对象。

await命令后面的 Promise 对象如果变为reject状态，则reject的参数会被catch方法的回调函数接收到，那么整个async函数都会中断执行。
```JavaScript
async function f() {
  await Promise.reject('出错了');
  await Promise.resolve('hello world'); // 不会执行
}
```

如果希望即使前一个异步操作失败，也不要中断后面的异步操作，这时可以将第一个await放在try...catch结构里面。另一种方法是await后面的 Promise 对象再跟一个catch方法，处理前面可能出现的错误。
```JavaScript
async function f() {
  await Promise.reject('出错了')
    .catch(e => console.log(e));
  return await Promise.resolve('hello world');
}

f()
.then(v => console.log(v))
// 出错了
// hello world
```

#### 使用注意点

第一点，前面已经说过，await命令后面的Promise对象，运行结果可能是rejected，所以最好把await命令放在try...catch代码块中。

第二点，多个await命令后面的异步操作，如果不存在继发关系，最好让它们同时触发。下两种写法，getFoo和getBar都是同时触发，这样就会缩短程序的执行时间。
```JavaScript
// 写法一
let [foo, bar] = await Promise.all([getFoo(), getBar()]);

// 写法二
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
```

第三点，await命令只能用在async函数之中，如果用在普通函数，就会报错。一般使用for循环代理foreach。
```JavaScript
// 错误
function dbFuc(db) { //这里不需要 async
  let docs = [{}, {}, {}];

  // forEach是并发执行，可能得到错误结果
  docs.forEach(async function (doc) {
    await db.post(doc);
  });
}

// 正确
async function dbFuc(db) {
  let docs = [{}, {}, {}];

  for (let doc of docs) {
    await db.post(doc);
  }
}
```
如果确实希望多个请求并发执行，可以使用Promise.all方法。

第四点，async 函数可以保留运行堆栈。
```JavaScript
const a = () => {
  b().then(() => c());
};
```
上面代码中，函数a内部运行了一个异步任务b()。当b()运行的时候，函数a()不会中断，而是继续执行。等到b()运行结束，可能a()早就运行结束了，b()所在的上下文环境已经消失了。如果b()或c()报错，错误堆栈将不包括a()。

```JavaScript
const a = async () => {
  await b();
  c();
};
```
上面代码中，b()运行的时候，a()是暂停执行，上下文环境都保存着。一旦b()或c()报错，错误堆栈将包括a()。

### async 函数的实现原理

（跳过）


### 与其他异步处理方法的比较

（跳过）推荐使用async写法，有点看本文开头。


### 实例：按顺序完成异步操作

实际开发中，经常遇到一组异步操作，需要按照顺序完成。比如，依次远程读取一组 URL，然后按照读取的顺序输出结果。
```JavaScript
async function logInOrder(urls) {
  // 并发读取远程URL
  const textPromises = urls.map(async url => {
    const response = await fetch(url);
    return response.text();
  });

  // 按次序输出
  for (const textPromise of textPromises) {
    console.log(await textPromise);
  }
}
```
上面代码中，虽然map方法的参数是async函数，但它是并发执行的，因为只有async函数内部是继发执行，外部不受影响。后面的for..of循环内部使用了await，因此实现了按顺序输出。

### 顶层 await

根据语法规格，await命令只能出现在 async 函数内部，否则都会报错。目前，有一个[语法提案](https://github.com/tc39/proposal-top-level-await)，允许在模块的顶层独立使用await命令。这个提案的目的，是借用await解决模块异步加载的问题。

顶层的await命令有点像，交出代码的执行权给其他的模块加载，等异步操作完成后，再拿回执行权，继续向下执行。下面是顶层await的一些使用场景。
```JavaScript
// import() 方法加载
const strings = await import(`/i18n/${navigator.language}`);

// 数据库操作
const connection = await dbConnector();

// 依赖回滚
let jQuery;
try {
  jQuery = await import('https://cdn-a.com/jQuery');
} catch {
  jQuery = await import('https://cdn-b.com/jQuery');
}
```

**注意，如果加载多个包含顶层await命令的模块，加载命令是同步执行的。**
```JavaScript
// x.js
console.log("X1");
await new Promise(r => setTimeout(r, 1000));
console.log("X2");

// y.js
console.log("Y");

// z.js
import "./x.js";
import "./y.js";
console.log("Z");
```
上面代码有三个模块，最后的z.js加载x.js和y.js，打印结果是X1、Y、X2、Z。这说明，z.js并没有等待x.js加载完成，再去加载y.js。