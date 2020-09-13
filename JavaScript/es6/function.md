# 函数的扩展

[ECMAScript 6 入门 - 函数的扩展](http://es6.ruanyifeng.com/#docs/function)

[toc]

### 函数参数的默认值

#### 基本用法

请问下面两种写法的差别：写法一函数参数的默认值是空对象，但是设置了对象解构赋值的默认值；写法二函数参数的默认值是一个有具体属性的对象，但是没有设置对象解构赋值的默认值。
```JavaScript
// 写法一
function m1({x = 0, y = 0} = {}) {
  return [x, y];
}

// 写法二
function m2({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}

// 函数没有参数的情况
m1() // [0, 0]
m2() // [0, 0]

// x 和 y 都有值的情况
m1({x: 3, y: 8}) // [3, 8]
m2({x: 3, y: 8}) // [3, 8]

// x 有值，y 无值的情况
m1({x: 3}) // [3, 0]
m2({x: 3}) // [3, undefined]

// x 和 y 都无值的情况
m1({}) // [0, 0];
m2({}) // [undefined, undefined]

m1({z: 3}) // [0, 0]
m2({z: 3}) // [undefined, undefined]
```

#### 函数的 length 属性

length属性的含义是，该函数预期传入的参数个数。指定了默认值以后，length属性将返回没有指定默认值的参数个数。所以rest 参数也不会计入length属性，设置了默认值的参数不是尾参数，那么length属性也不再计入后面的参数了。
```JavaScript
(function (a) {}).length // 1
(function (a = 5) {}).length // 0
(function (a, b, c = 5) {}).length // 2

(function(...args) {}).length // 0

(function (a = 0, b, c) {}).length // 0
(function (a, b = 1, c) {}).length // 1
```

#### **作用域**

**一旦设置了参数的默认值，函数进行声明初始化时，参数会形成一个单独的作用域（context）。等到初始化结束，这个作用域就会消失。这种语法行为，在不设置参数默认值时，是不会出现的。** 如果参数的默认值是一个函数，该函数的作用域也遵守这个规则。

！！！[重点！跳转到原内容查看](http://es6.ruanyifeng.com/#docs/function#%E4%BD%9C%E7%94%A8%E5%9F%9F)！！！


### rest 参数

arguments对象不是数组，而是一个类似数组的对象。rest 是一个真正的数组，数组特有的方法都可以使用。函数的length属性，不包括 rest 参数。

### 严格模式

函数内部从 ES5 开始可以显式设定为严格模式。ES2016 规定只要函数参数使用了默认值、解构赋值、或者扩展运算符，那么函数内部就不能显式设定为严格模式，否则会报错。

这样规定的原因是，函数内部的严格模式，同时适用于函数体和函数参数。但是，函数执行的时候，先执行函数参数，然后再执行函数体。这样就有一个不合理的地方，只有从函数体之中，才能知道参数是否应该以严格模式执行，但是参数却应该先于函数体执行。

两种方法可以规避这种限制。第一种是设定全局性的严格模式，这是合法的。第二种是把函数包在一个无参数的立即执行函数里面。

### name 属性

返回该函数的函数名。将一个匿名函数赋值给一个变量，ES5会返回空字符串，ES6会返回实际的函数名。将一个具名函数赋值给一个变量，ES5 和 ES6 都返回这个具名函数原本的名字。
Function构造函数返回的函数实例，name属性的值为anonymous。bind返回的函数，name属性值会加上bound前缀。取值函数（getter）和存值函数（setter）返回值是方法名前加上get和set，如`"get foo"  "set foo"`。

### 箭头函数

箭头函数有几个使用注意点。
1. 函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。
2. 不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误。
3. 不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。
4. 不可以使用yield命令，因此箭头函数不能用作 Generator 函数。

上面四点中，第一点尤其值得注意。this对象的指向是可变的，但是在箭头函数中，它是固定的。
长期以来，JavaScript 语言的this对象一直是一个令人头痛的问题，在对象方法中使用this，必须非常小心。箭头函数”绑定”this，很大程度上解决了这个困扰。

**不适用场合**

由于箭头函数使得this从“动态”变成“静态”，下面两个场合不应该使用箭头函数：
第一个场合是定义对象的方法，且该方法内部包括this。调用cat.jumps()时，如果是普通函数，该方法内部的this指向cat；如果写成上面那样的箭头函数，使得this指向全局对象，因此不会得到预期结果。**对象不构成单独的作用域** ，导致jumps箭头函数定义时的作用域就是全局作用域。
```JavaScript
const cat = {
  lives: 9,
  jumps: () => {
    this.lives--;
  }
}
```
第二个场合是需要动态this的时候，也不应使用箭头函数。

### 尾调用优化

指某个函数的最后一步是调用另一个函数，与其他调用不同，就在于它的特殊的调用位置。

我们知道，函数调用会在内存形成一个“调用记录”，又称“调用帧”（call frame），保存调用位置和内部变量等信息。如果在函数A的内部调用函数B，那么在A的调用帧上方，还会形成一个B的调用帧。等到B运行结束，将结果返回到A，B的调用帧才会消失。如果函数B内部还调用函数C，那就还有一个C的调用帧，以此类推。所有的调用帧，就形成一个“调用栈”（call stack）。

尾调用由于是函数的最后一步操作，所以不需要保留外层函数的调用帧，因为调用位置、内部变量等信息都不会再用到了，只要直接用内层函数的调用帧，取代外层函数的调用帧就可以了。这就叫做“尾调用优化”（Tail call optimization），即只保留内层函数的调用帧。如果所有函数都是尾调用，那么完全可以做到每次执行时，调用帧只有一项，这将大大节省内存。这就是“尾调用优化”的意义。

以下三种情况，都不属于尾调用。
```JavaScript
// 情况一
function f(x){
  let y = g(x);
  return y;
}

// 情况二
function f(x){
  return g(x) + 1;
}

// 情况三
function f(x){
  g(x);
}
function f(x){
  g(x);
  return undefined;
}

// 只要是最后一步操作是尾调，就是尾调用。
function f(x) {
  if (x > 0) {
    return m(x)
  }
  return n(x);
}
```

注意，只有不再用到外层函数的内部变量，内层函数的调用帧才会取代外层函数的调用帧，否则就无法进行“尾调用优化”。内层函数inner用到了外层函数addOne的内部变量one。
```JavaScript
function addOne(a){
  var one = 1;
  function inner(b){
    return b + one;
  }
  return inner(a);
}
```

目前只有 Safari 浏览器支持尾调用优化，Chrome 和 Firefox 都不支持。**ES6 的尾调用优化只在严格模式下开启，正常模式是无效的。**

尾调用优化对递归函数有重大优化作用。

### 其他

#### 函数参数的尾逗号

ES2017 [允许](https://github.com/jeffmo/es-trailing-function-commas)函数的最后一个参数有尾逗号（trailing comma），使得函数参数与数组和对象的尾逗号规则保持一致了。
```JavaScript
function clownsEverywhere(
  param1,
  param2,
) { /* ... */ }

clownsEverywhere(
  'foo',
  'bar',
);
```

#### Function.prototype.toString()

[ES2019](https://github.com/tc39/Function-prototype-toString-revision) 要求toString()方法返回一模一样的原始代码，以前会省略注释和空格。

#### catch 命令的参数省略

很多时候，catch代码块可能用不到这个参数，[ES2019](https://github.com/tc39/proposal-optional-catch-binding) 做出了改变，允许catch语句省略参数。
```JavaScript
try {
  // ...
} catch {
  // ...
}
```