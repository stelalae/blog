#  数值的扩展

[ECMAScript 6 入门 - 数值的扩展](http://es6.ruanyifeng.com/#docs/number)
[MDN Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)
[MDN BigInt](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/BigInt)

[TOC]

ES6 中二进制前缀：0b（或0B），八进制前缀：0o（或0O）。使用Number方法将字符串数值转为十进制
```JavaScript
Number('0b111')  // 7
Number('0o10')  // 8
```

isFinite()、isNaN()：传统的全局方法isFinite()和isNaN()的是先调用Number()将非数值的值转为数值，再进行判断，而这两个新方法只对数值有效，Number.isFinite()对于非数值一律返回false, Number.isNaN()只有对于NaN才返回true，非NaN一律返回false。
```JavaScript
Number.isFinite(0.8); // true
Number.isFinite(NaN); // false
Number.isFinite(Infinity); // false
Number.isFinite(-Infinity); // false
Number.isFinite('foo'); // false
Number.isFinite('15'); // false
Number.isFinite(true); // false
```

parseInt()、parseFloat()：ES6 将全局方法parseInt()和parseFloat()，移植到Number对象上面，行为完全保持不变。目的是逐步减少全局性方法，使得语言逐步模块化。

isInteger()：对数据精度的要求较高时，判断一个数值是否为整数，不是数值返回false。JavaScript内部，整数和浮点数采用的是同样的储存方法，所以 25 和 25.0 被视为同一个值。

Number.EPSILON：极小的常量，它表示 1 与大于 1 的最小浮点数之间的差，等于 2 的 -52 次方。可以用来设置“能够接受的误差范围”，即如果两个浮点数的差小于这个值，我们就认为这两个浮点数相等，或单独设定误差范围。
```JavaScript
0.1 + 0.2
// 0.30000000000000004

0.1 + 0.2 - 0.3
// 5.551115123125783e-17

0.1 + 0.2 - 0.3 < Number.EPSILON
// true

// 误差范围设为 2 的-50 次方
function withinErrorMargin (left, right) {
  return Math.abs(left - right) < Number.EPSILON * Math.pow(2, 2);
}
0.1 + 0.2 === 0.3 // false
withinErrorMargin(0.1 + 0.2, 0.3) // true
1.1 + 1.3 === 2.4 // false
withinErrorMargin(1.1 + 1.3, 2.4) // true
```

isSafeInteger：MAX_SAFE_INTEGER、MIN_SAFE_INTEGER分表表示整数范围在-2^53到2^53之间。实际使用这个函数时，需要注意。验证运算结果是否落在安全整数的范围内，不要只验证运算结果，而要同时验证参与运算的每个值。

### Math 对象的扩展

Math.trunc()：用于去除一个数的小数部分，返回整数部分。对于非数值，Math.trunc内部使用Number方法将其先转为数值。对于空值和无法截取整数的值，返回NaN。
```JavaScript
Math.trunc(4.9) // 4
Math.trunc(-4.1) // -4

Math.trunc('123.456') // 123
Math.trunc(true) //1
Math.trunc(false) // 0
Math.trunc(null) // 0

Math.trunc(NaN);      // NaN
Math.trunc('foo');    // NaN
Math.trunc();         // NaN
Math.trunc(undefined) // NaN
```

Math.sign：判断一个数到底是正数、负数、还是零。对于非数值，会先将其转换为数值，对于那些无法转为数值的值，会返回NaN。参数为正数，返回+1；参数为负数，返回-1；参数为 0，返回0；参数为-0，返回-0；其他值，返回NaN。

Math.cbrt：计算一个数的立方根。对于非数值，会先将其转换为数值。

Math.clz32：将参数转为 32 位无符号整数的形式，然后返回这个 32 位值里面有多少个前导 0。对于非数值，会先将其转换为数值。对于小数，Math.clz32方法只考虑整数部分。左移运算符（<<）与Math.clz32方法直接相关。

Math.imul：返回两个数以 32 位带符号整数形式相乘的结果，返回的也是一个 32 位的带符号整数。

Math.fround：返回一个数的32位单精度浮点数形式。

Math.hypot：返回所有参数的平方和的平方根。

#### 对数

Math.expm1、Math.log1p、Math.log10、Math.log2

#### 双曲函数

* Math.sinh(x) 返回x的双曲正弦（hyperbolic sine）
* Math.cosh(x) 返回x的双曲余弦（hyperbolic cosine）
* Math.tanh(x) 返回x的双曲正切（hyperbolic tangent）
* Math.asinh(x) 返回x的反双曲正弦（inverse hyperbolic sine）
* Math.acosh(x) 返回x的反双曲余弦（inverse hyperbolic cosine）
* Math.atanh(x) 返回x的反双曲正切（inverse hyperbolic tangent）

#### 指数运算符

多个指数运算符连用时，是从最右边开始计算的。注意，V8 引擎的指数运算符与Math.pow的实现不相同，对于特别大的运算结果，两者会有细微的差异。
```JavaScript
2 ** 2 // 4
2 ** 3 // 8
2 ** 3 ** 2 // 512，相当于 2 ** (3 ** 2)

a **= 2; // 等同于 a = a * a;
b **= 3;// 等同于 b = b * b * b;

Math.pow(99, 99)
// 3.697296376497263e+197

99 ** 99
// 3.697296376497268e+197
```

#### BigInt

**JavaScript 所有数字都保存成 64 位浮点数** ，这给数值的表示带来了两大限制：
* 一是数值的精度只能到 53 个二进制位（相当于 16 个十进制位），大于这个范围的整数，JavaScript 是无法精确表示的，这使得 JavaScript 不适合进行科学和金融方面的精确计算。
* 二是大于或等于2的1024次方的数值，JavaScript 无法表示，会返回Infinity。

[ES2020](https://github.com/tc39/proposal-bigint) 引入了一种新的数据类型 BigInt（大整数），**只用来表示整数**，没有位数的限制，任何位数的整数都可以精确表示。BigInt 同样可以使用各种进制表示，都要加上后缀n。
```JavaScript
1234 // 普通整数
1234n // BigInt

// BigInt 的运算
1n + 2n // 3n

0b1101n // 二进制
0o777n // 八进制
0xFFn // 十六进制

42n === 42 // false，但是宽松相等的

typeof 123n // 'bigint'

// 可以使用负号（-），但是不能使用正号（+），因为会与 asm.js 冲突。
-42n // 正确
+42n // 报错
```

##### BigInt 对象

JavaScript 原生提供BigInt对象，可以用作构造函数生成 BigInt 类型的数值。转换规则基本与Number()一致，将其他类型的值转为 BigInt。
```JavaScript
BigInt(123) // 123n
BigInt('123') // 123n
BigInt(false) // 0n
BigInt(true) // 1n
```

BigInt()构造函数必须有参数，而且参数必须可以正常转为数值，下面的用法都会报错。
```JavaScript
new BigInt() // TypeError
BigInt(undefined) //TypeError
BigInt(null) // TypeError
BigInt('123n') // SyntaxError   // 123n无法解析成 Number 类型，所以会报错。
BigInt('abc') // SyntaxError
BigInt(1.5) // RangeError
BigInt('1.5') // SyntaxError
```

* BigInt.prototype.toString()：继承了 Object 对象
* BigInt.prototype.valueOf()：继承了 Object 对象
* BigInt.prototype.toLocaleString()：继承了 Number 对象
* BigInt.asUintN(width, BigInt)：静态方法，给定的 BigInt 转为 0 到 2width - 1 之间对应的值。
* BigInt.asIntN(width, BigInt)：静态方法，给定的 BigInt 转为 -2width - 1 到 2width - 1 - 1 之间对应的值。
* BigInt.parseInt(string[, radix])：静态方法，近似于Number.parseInt()，将一个字符串转换成指定进制的 BigInt。

对于二进制数组，BigInt 新增了两个类型BigUint64Array和BigInt64Array，这两种数据类型返回的都是64位 BigInt。DataView对象的实例方法DataView.prototype.getBigInt64()和DataView.prototype.getBigUint64()，返回的也是 BigInt。

##### 转换规则

```JavaScript
Boolean(0n) // false
Boolean(1n) // true
Number(1n)  // 1
String(1n)  // "1"
!0n // true
!1n // false
```

##### 数学运算

BigInt 类型的+、-、\*和\*\*这四个二元运算符，与 Number 类型的行为一致。除法运算/会舍去小数部分，返回一个整数。几乎所有的数值运算符都可以用在 BigInt，但是有两个例外。
* 不带符号的右移位运算符>>>：>>>运算符是不带符号的，但是 BigInt 总是带有符号的，导致该运算无意义，完全等同于右移运算符>>。
* 一元的求正运算符+：在 asm.js 里面总是返回 Number 类型，为了不破坏 asm.js 就规定+1n会报错。
```JavaScript
9n / 5n // 1n

// Math.sqrt的参数预期是 Number 类型
Math.sqrt(4n) // 报错
Math.sqrt(Number(4n)) // 2
```

* 不能与普通数值进行混合运算，无论返回的是 BigInt 或 Number，都会导致丢失精度信息。
* 比较运算符（比如>）和相等运算符（==）允许 BigInt 与其他类型的值混合计算，因为这样做不会损失精度。
* BigInt 与字符串混合运算时，会先转为字符串，再进行运算。
```JavaScript
1n + 1.3 // 报错
1n | 0 // 报错

if (0n) {
  console.log('if');
}

0n < 1 // true
0n < true // true
0n == 0 // true
0n == false // true
0n === 0 // false

'' + 123n // "123"
```