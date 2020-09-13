# 变量的解构赋值

[ECMAScript 6 入门 - 变量的解构赋值](http://es6.ruanyifeng.com/#docs/destructuring)

[TOC]

### 数组的解构赋值

为变量赋值，这种写法属于“模式匹配”，只要等号两边的模式相同，左边的变量就会被赋予对应的值。如果解构不成功，变量的值就等于undefined。
```JavaScript
let [a, b, c] = [1, 2, 3];

let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3

let [ , , third] = ["foo", "bar", "baz"];
third // "baz"

let [x, , y] = [1, 2, 3];
x // 1
y // 3

let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []

let [foo] = []; // foo - undefined
let [bar, foo] = [1]; // foo - undefined
```

不完全解构，即等号左边的模式，只匹配一部分的等号右边的数组。
```JavaScript
let [x, y] = [1, 2, 3];
x // 1
y // 2

let [a, [b], d] = [1, [2, 3], 4];
a // 1
b // 2
d // 4
```
由于数组本质是特殊的对象，因此可以对数组进行对象属性的解构。
```JavaScript
let arr = [1, 2, 3];
let {0 : first, [arr.length - 1] : last} = arr;
first // 1
last // 3
```
数组arr的0键对应的值是1，[arr.length - 1]就是2键，对应的值是3。方括号这种写法，属于后面介绍的对象解构。

对于 Set 结构，也可以使用数组的解构赋值。事实上，只要某种数据结构具有 Iterator 接口，都可以采用数组形式的解构赋值。
```JavaScript
let [x, y, z] = new Set(['a', 'b', 'c']);
x // "a"
```

#### 默认值

ES6 内部使用严格相等运算符（===），判断一个位置是否有值。所以，只有当等于undefined，默认值才会生效。
如果默认值是一个表达式，那么这个表达式是**惰性求值**的，即只有在用到的时候，才会求值。
```JavaScript
function f() {
  console.log('aaa');   // 不会执行
}

let [x = f()] = [1];
```

默认值可以引用解构赋值的其他变量，但该变量必须已经声明。
```JavaScript
let [x = y, y = 1] = [];     // ReferenceError: y is not defined
```

#### 字符串的解构赋值

字符串也可以解构赋值。这是因为此时，字符串被转换成了一个类似数组的对象。类似数组的对象都有一个length属性，因此还可以对这个属性解构赋值。
```JavaScript
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"

let {length : len} = 'hello';
len // 5
```

### 对象的解构赋值

嵌套解构的对象。下面第二个p是模式，不是变量，因此不会被赋值。如果p也要作为变量赋值，需再写一个。
```JavaScript
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};

let { p, p: [x, { y }] } = obj;
x // "Hello"
y // "World"
p // ["Hello", {y: "World"}]

const node = {
  loc: {
    start: {
      line: 1,
      column: 5
    }
  }
};

let { loc, loc: { start }, loc: { start: a, start: { line }} } = node;
line // 1
loc  // Object {start: Object}
start, a // Object {line: 1, column: 5}

// 嵌套赋值
let obj = {};
let arr = [];

({ foo: obj.prop, bar: arr[0] } = { foo: 123, bar: true });

obj // {prop:123}
arr // [true]

// 对象的解构赋值可以取到继承的属性。
const obj1 = {};
const obj2 = { foo: 'bar' };
Object.setPrototypeOf(obj1, obj2);

const { foo } = obj1;
foo // "bar"

var {x = 3} = {x: undefined};
x // 3

// 一个已经声明的变量用于解构赋值，加()是避免=左侧被解析成作用域。建议只要有可能，就不要在模式中放置圆括号。
let x;
({x} = {x: 1});

// 下面是有效的
({} = [true, false]);
({} = 'abc');
({} = []);
```

**解构赋值的规则是，只要等号右边的值不是对象或数组，就先将其转为对象。由于undefined和null无法转为对象，所以对它们进行解构赋值，都会报错。**
```JavaScript
let {toString: s} = 123;
s === Number.prototype.toString // true

let {toString: s} = true;
s === Boolean.prototype.toString // true

let { prop: x } = undefined; // TypeError
let { prop: y } = null; // TypeError
```

### 用途

* 交换变量的值
* 从函数返回多个值
* 函数参数的定义
* 提取 JSON 数据
* 函数参数的默认值
* 遍历 Map 结构
* 输入模块的指定方法