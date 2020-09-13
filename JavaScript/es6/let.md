#  let 和 const 命令

[ECMAScript 6 入门 - let 和 const 命令](http://es6.ruanyifeng.com/#docs/let)

[TOC]

### let

#### 基本用法

let声明的变量只在它所在的代码块有效。for循环体内变量i是let声明的，i只在本轮循环有效，所以每一次循环的i其实都是一个新的变量。你可能会问，如果每一轮循环的变量i都是重新声明的，那它怎么知道上一轮循环的值，从而计算出本轮循环的值？这是因为 JavaScript 引擎内部会记住上一轮循环的值，初始化本轮的变量i时，就在上一轮循环的基础上进行计算。
```JavaScript
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
```
上面输出了 3 次abc。这表明函数内部的变量i与循环变量i不在同一个作用域，有各自单独的作用域。let不允许在相同作用域内，重复声明同一个变量。**所以不要变量重名，给自己造成困扰。**

#### 暂时性死区TDZ

ES6 明确规定，如果区块中存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。总之，在代码块内使用let命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。
```JavaScript
if (true) {
  // TDZ开始
  tmp = 'abc'; // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp; // TDZ结束
  console.log(tmp); // undefined

  tmp = 123;
  console.log(tmp); // 123
}
```
“暂时性死区”不存在变量提升，也意味着typeof不再是一个百分之百安全的操作，但如果一个变量根本没有被声明，使用typeof反而不会报错，会得到undefined。


### 块级作用域

ES5 只有全局作用域和函数作用域，没有块级作用域。
* ES6 中块级作用域允许任意嵌套。
* ES6 的块级作用域必须有大括号，如果没有大括号，JavaScript 引擎就认为不存在块级作用域。

> 如果改变了块级作用域内声明的函数的处理规则，显然会对老代码产生很大影响。为了减轻因此产生的不兼容问题，ES6 在[附录 B](http://www.ecma-international.org/ecma-262/6.0/index.html#sec-block-level-function-declarations-web-legacy-compatibility-semantics)里面规定，浏览器的实现可以不遵守上面的规定，有自己的[行为方式](http://stackoverflow.com/questions/31419897/what-are-the-precise-semantics-of-block-level-functions-in-es6)。
> * 允许在块级作用域内声明函数。
> * 函数声明类似于var，即会提升到全局作用域或函数作用域的头部。
> * 同时，函数声明还会提升到所在的块级作用域的头部。

所以如果确实需要，也应该写成函数表达式或箭头函数，而不是函数声明语句。
```JavaScript
// 块级作用域内部的函数声明语句，建议不要使用
{
  let a = 'secret';
  function f() {
    return a;
  }
}

// 块级作用域内部，优先使用函数表达式
{
  let a = 'secret';
  let f = function () {
    return a;
  };
}
```

### const

const 声明变量时就必须立即初始化，不能留到以后赋值。和let一样，不能提升，同样存在暂时性死区，只能在声明的位置后面使用。

const实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动。对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指向实际数据的指针，const只能保证这个指针是固定的（即总是指向另一个固定的地址），至于它指向的数据结构是不是可变的，就完全不能控制了。因此，将一个对象声明为常量必须非常小心。

如果真的想将对象冻结，应该使用Object.freeze方法。除了将对象本身冻结，对象的属性也应该冻结。
```JavaScript
const foo = Object.freeze({});

// 常规模式时，下面一行不起作用；
// 严格模式时，该行会报错
foo.prop = 123;
```

### globalThis 对象

顶层对象在各种实现里面是不统一的。
* 浏览器里面，顶层对象是window，但 Node 和 Web Worker 没有window。
* 浏览器和 Web Worker 里面，self也指向顶层对象，但是 Node 没有self。
* Node 里面，顶层对象是global，但其他环境都不支持。

现在一般是使用this变量，但是有局限性。
* 全局环境中，this会返回顶层对象。但是，Node 模块和 ES6 模块中，this返回的是当前模块。
* 函数里面的this，如果函数不是作为对象的方法运行，而是单纯作为函数运行，this会指向顶层对象。但是，严格模式下，这时this会返回undefined。
* 不管是严格模式，还是普通模式，new Function('return this')()，总是会返回全局对象。但是，如果浏览器用了 CSP（Content Security Policy，内容安全策略），* 那么eval、new Function这些方法都可能无法使用。

ES2020 在语言标准的层面，引入globalThis作为顶层对象。任何环境下，globalThis都是指向全局环境下的this，推荐垫片库[global-this](https://github.com/ungap/global-this)。