# 风格和规格

[ECMAScript 6 入门 - 编程风格](http://es6.ruanyifeng.com/#docs/style)
[ECMAScript 6 入门 - 读懂 ECMAScript 规格](http://es6.ruanyifeng.com/#docs/spec)

[toc]

## 编程风格

### 块级作用域

（1）let 取代 var：var命令存在变量提升效用，let命令没有这个问题。
（2）全局常量和线程安全：在let和const之间，建议优先使用const，尤其是在全局环境，不应该设置变量，只应设置常量。const优于let有几个原因。
* const可以提醒阅读程序的人，这个变量不应该改变；
* const比较符合函数式编程思想，运算不改变值，只是新建值，而且这样也有利于将来的分布式运算；
*  JavaScript 编译器会对const进行优化，所以多使用const，有利于提高程序的运行效率。

**也就是说let和const的本质区别，其实是编译器内部的处理不同。**


### 字符串

静态字符串一律使用单引号或反引号，不使用双引号。动态字符串使用反引号。
```JavaScript
// bad
const a = "foobar";
const b = 'foo' + a + 'bar';

// acceptable
const c = `foobar`;

// good
const a = 'foobar';
const b = `foo${a}bar`;
```


### 解构赋值

使用数组成员对变量赋值时，优先使用解构赋值。
```JavaScript
const arr = [1, 2, 3, 4];

// bad
const first = arr[0];
const second = arr[1];

// good
const [first, second] = arr;
```

函数的参数如果是对象的成员，优先使用解构赋值。

如果函数返回多个值，优先使用对象的解构赋值，而不是数组的解构赋值。**这样便于以后添加返回值，以及更改返回值的顺序。**
```JavaScript
// bad
function processInput(input) {
  return [left, right, top, bottom];
}

// good
function processInput(input) {
  return { left, right, top, bottom };
}

const { left, right } = processInput(input);
```


### 对象


对象尽量静态化，一旦定义，就不得随意添加新的属性。如果添加属性不可避免，要使用Object.assign方法。如果对象的属性名是动态的，可以在创造对象的时候，使用属性表达式定义。
```JavaScript
// bad
const a = {};
a.x = 3;

// if reshape unavoidable
const a = {};
Object.assign(a, { x: 3 });

// good
const a = { x: null };
a.x = 3;

// bad
const obj = {
  id: 5,
  name: 'San Francisco',
};
obj[getKey('enabled')] = true;

// good
const obj = {
  id: 5,
  name: 'San Francisco',
  [getKey('enabled')]: true,
};
```
另外，对象的属性和方法，尽量采用简洁表达法，这样易于描述和书写。

### 数组

使用扩展运算符（...）拷贝数组。使用 Array.from 方法，将类似数组的对象转为数组。


### 函数

立即执行函数可以写成箭头函数的形式。那些使用匿名函数当作参数的场合，尽量用箭头函数代替。因为这样更简洁，而且绑定了 this。箭头函数取代Function.prototype.bind，不应再用 self/\_this/that 绑定 this。
```JavaScript
// bad
[1, 2, 3].map(function (x) {
  return x * x;
});

// good
[1, 2, 3].map((x) => {
  return x * x;
});

// best
[1, 2, 3].map(x => x * x);

// bad
const self = this;
const boundMethod = function(...params) {
  return method.apply(self, params);
}

// acceptable
const boundMethod = method.bind(this);

// best
const boundMethod = (...params) => method.apply(this, params);
```
简单的、单行的、不会复用的函数，建议采用箭头函数。如果函数体较为复杂，行数较多，还是应该采用传统的函数写法。

所有配置项都应该集中在一个对象，放在最后一个参数，布尔值不可以直接作为参数。
```JavaScript
// bad
function divide(a, b, option = false ) {
}

// good
function divide(a, b, { option = false } = {}) {
}
```
不要在函数体内使用 arguments 变量，使用 rest 运算符（...）代替。因为 rest 运算符显式表明你想要获取参数，而且 arguments 是一个类似数组的对象，而 rest 运算符可以提供一个真正的数组。
```JavaScript
// bad
function concatenateAll() {
  const args = Array.prototype.slice.call(arguments);
  return args.join('');
}

// good
function concatenateAll(...args) {
  return args.join('');
}
```

使用默认值语法设置函数参数的默认值。

### Map 结构

注意区分 Object 和 Map，只有模拟现实世界的实体对象时，才使用 Object。如果只是需要key: value的数据结构，使用 Map 结构。因为 Map 有内建的遍历机制。


### Class

总是用 Class，取代需要 prototype 的操作。因为 Class 的写法更简洁，更易于理解。

使用extends实现继承，因为这样更简单，不会有破坏instanceof运算的危险。


### 模块

首先，Module 语法是 JavaScript 模块的标准写法，坚持使用这种写法。使用import取代require。

如果模块只有一个输出值，就使用export default，如果模块有多个输出值，就不使用export default，export default与普通的export不要同时使用。不要在模块输入中使用通配符。因为这样可以确保你的模块之中，有一个默认输出（export default）。
```JavaScript
// bad
import * as myObject from './importModule';

// good
import myObject from './importModule';
```

如果模块默认输出一个函数，函数名的首字母应该小写。如果模块默认输出一个对象，对象名的首字母应该大写。




## 读懂 ECMAScript 规格

### 概述

规格文件是计算机语言的官方标准，详细描述语法规则和实现方法。

一般来说，没有必要阅读规格，除非你要写编译器。因为规格写得非常抽象和精炼，又缺乏实例，不容易理解，而且对于解决实际的应用问题，帮助不大。但是，如果你遇到疑难的语法问题，实在找不到答案，这时可以去查看规格文件，了解语言标准是怎么说的。规格是解决问题的“最后一招”。

**这对 JavaScript 语言很有必要。因为它的使用场景复杂，语法规则不统一，例外很多，各种运行环境的行为不一致，导致奇怪的语法问题层出不穷，任何语法书都不可能囊括所有情况。查看规格，不失为一种解决语法问题的最可靠、最权威的终极方法。**

ECMAScript 6 的规格，可以在 ECMA 国际标准组织的官方网站（[www.ecma-international.org/ecma-262/6.0/](http://www.ecma-international.org/ecma-262/6.0/)）免费下载和在线阅读。

这个规格文件相当庞大，一共有 26 章，A4 打印的话，足足有 545 页。它的特点就是规定得非常细致，每一个语法行为、每一个函数的实现都做了详尽的清晰的描述。基本上，编译器作者只要把每一步翻译成代码就可以了。这很大程度上，保证了所有 ES6 实现都有一致的行为。

ECMAScript 6 规格的 26 章之中，第 1 章到第 3 章是对文件本身的介绍，与语言关系不大。第 4 章是对这门语言总体设计的描述，有兴趣的读者可以读一下。第 5 章到第 8 章是语言宏观层面的描述。第 5 章是规格的名词解释和写法的介绍，第 6 章介绍数据类型，第 7 章介绍语言内部用到的抽象操作，第 8 章介绍代码如何运行。第 9 章到第 26 章介绍具体的语法。

对于一般用户来说，除了第 4 章，其他章节都涉及某一方面的细节，不用通读，只要在用到的时候，查阅相关章节即可。

### 术语

ES6 规格使用了一些专门的术语，了解这些术语，可以帮助你读懂规格。本节介绍其中的几个。

抽象操作：就是引擎的一些内部方法，外部不能调用。规格定义了一系列的抽象操作，规定了它们的行为，留给各种引擎自己去实现。
Record 和 field：ES6 规格将键值对（key-value map）的数据结构称为 Record，其中的每一组键值对称为 field。这就是说，一个 Record 由多个 field 组成，而每个 field 都包含一个键名（key）和一个键值（value）。
[[Notation]]：ES6 规格大量使用[[Notation]]这种书写法，比如[[Value]]、[[Writable]]、[[Get]]、[[Set]]等等。它用来指代 field 的键名。
Completion Record：每一个语句都会返回一个 Completion Record，表示运行结果。每个 Completion Record 有一个[[Type]]属性，表示运行结果的类型。[[Type]]属性有五种可能的值：
* normal：表示运行正常。
* return
* throw：表示运行出错。重点关注！
* break
* continue

## 最新提案

### do 表达式

现在有一个[提案](https://github.com/tc39/proposal-do-expressions)，使得块级作用域可以变为表达式，也就是说可以返回值，办法就是在块级作用域之前加上do，使它变为do表达式，然后就会返回内部最后执行的表达式的值。
```JavaScript
let x = do {
  let t = f();
  t * t + 1;
};
```
上面代码中，变量x会得到整个块级作用域的返回值（t * t + 1）。

do表达式的逻辑非常简单：封装的是什么，就会返回什么。提供了单独的作用域，内部操作可以与全局作用域隔绝，让程序更加模块化。
```JavaScript
// 等同于 <表达式>
do { <表达式>; }

// 等同于 <语句>
do { <语句> }
```

### throw 表达式

JavaScript 语法规定throw是一个命令，用来抛出错误，不能用于表达式之中。现在有一个[提案](https://github.com/tc39/proposal-throw-expressions)，允许throw用于表达式。


### 函数的部分执行

多参数的函数有时需要绑定其中的一个或多个参数，然后返回一个新函数。现在有一个[提案](https://github.com/tc39/proposal-partial-application)，使得绑定参数并返回一个新函数更加容易。这叫做函数的部分执行（partial application）。?是单个参数的占位符，...是多个参数的占位符，只能出现其一，如：
```JavaScript
f(x, ?)
f(x, ...)
f(?, x)
f(..., x)
f(?, x, ?)
f(..., x, ...)
```
函数的部分执行，也可以用于对象的方法。
```JavaScript
let obj = {
  f(x, y) { return x + y; },
};

const g = obj.f(?, 3);
g(1) // 4
```


### 管道运算符


把前一个操作的值传给后一个操作，使得简单的操作可以组合成为复杂的操作。许多语言都有管道的实现，现在有一个[提案](https://github.com/tc39/proposal-pipeline-operator)，让 JavaScript 也拥有管道机制。

JavaScript 的管道是一个运算符，写作|>。它的左边是一个表达式，右边是一个函数。管道运算符把左边表达式的值，传入右边的函数进行求值。管道运算符最大的好处，就是可以把嵌套的函数，写成从左到右的链式表达式。
```JavaScript
function doubleSay (str) {
  return str + ", " + str;
}

function capitalize (str) {
  return str[0].toUpperCase() + str.substring(1);
}

function exclaim (str) {
  return str + '!';
}

// 传统写法
exclaim(capitalize(doubleSay('hello'))) // "Hello, hello!"

// 管道写法
'hello'
  |> doubleSay
  |> capitalize
  |> exclaim
// "Hello, hello!"
```

管道运算符只能传递一个值，这意味着它右边的函数必须是一个单参数函数。如果是多参数函数，就必须进行柯里化，改成单参数的版本。管道运算符对于await函数也适用。


### 双冒号运算符

箭头函数可以绑定this对象，大大减少了显式绑定this对象的写法（call、apply、bind）。但是，箭头函数并不适用于所有场合，所以现在有一个[提案](https://github.com/zenparsing/es-function-bind)，提出了“函数绑定”（function bind）运算符，用来取代call、apply、bind调用。

函数绑定运算符是并排的两个冒号（::），双冒号左边是一个对象，右边是一个函数。该运算符会自动将左边的对象，作为上下文环境（即this对象），绑定到右边的函数上面。
```JavaScript
foo::bar;
// 等同于
bar.bind(foo);

foo::bar(...arguments);
// 等同于
bar.apply(foo, arguments);

const hasOwnProperty = Object.prototype.hasOwnProperty;
function hasOwn(obj, key) {
  return obj::hasOwnProperty(key);
}
```

如果双冒号左边为空，右边是一个对象的方法，则等于将该方法绑定在该对象上面。
```JavaScript
var method = obj::obj.foo;
// 等同于
var method = ::obj.foo;

let log = ::console.log;
// 等同于
var log = console.log.bind(console);
```

如果双冒号运算符的运算结果，还是一个对象，就可以采用链式写法。
```JavaScript
import { map, takeWhile, forEach } from "iterlib";

getPlayers()
::map(x => x.character())
::takeWhile(x => x.strength > 100)
::forEach(x => console.log(x));
```