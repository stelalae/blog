# 基础知识

[TOC]

### "use strict"

`"use strict"`是ES5引入的严格模式，用于处理ES3中不确定的行为，对某些不安全操作也会抛出错误。它是一个编译指示（pragma），告诉JavaScript引擎切换到严格模式，这时为了不破坏ES3语法而特意选定的语法。

在函数内部上方包含`"use strict"`是指定函数在严格模式下执行。



### 行尾分号

- 避免手误；
- 可放心大胆的删除多余空格，来压缩JavaScript代码，避免压缩导致的错误；
- 在某些情况下增加代码性能，减少解析器词法分析的时间；



### 数据类型

- 基本数据类型：Undefined、Null、Boolean、Number、String。

- 复杂数据类型：Object — 本质是一组无序的键值对组成的。

*undefined和null都是只有一个值得数据类型。*

typeof是一个操作符，而不是函数。返回值有：

- "undefined"：表示这个值`未被初始化`和`未被声明(定义)`
- "boolean"：布尔值
- "string"：字符串
- "number"：数值
- "object"：对象或null（一个空的对象引用）
- "function"：函数

```javascript
var a;
a === undefined; // true。未初始化的变量自动赋值为undefined
typeof a === 'undefined' // true
type b === 'undefined' // true
```

注意上面最后一行里，未被声明也会返回undefined，所以无法知道到底是未被初始化，还是未被声明。所以显示的初始化很有必要！

null表示一个空对象指针，可以理解成将来用于保存对象。

#### Number

八进制（第一位是0，字面值中数字超出了0~7就自动转换为十进制。 **在严格模式下无效!!!** ）、十六进制（以0x开头，数字范围0-9及A-F或a-f），进行计算时最终会被转换成十进制数值。*正零(+0)和负零(-0)被认为是想等的。*

```javascript
var octal1 = 070; // 八进制的56
var octal2 = 079, octal3 = 08; // 十进制的79，8

var hex1 = 0xA, hex2 = 0x1f;  // 十进制的10，31
```

浮点数内存空间是整数的两倍，因为会将小数点后无任何数字的浮点当做整数来存储。对于极大或极小的数字，可用`e表示法（科学计数法）`的浮点数值（floatNum）来表示。默认情况下，会将小数点后带6个0以上的浮点数值自动转换为e表示法。浮点数最高精度是17位，计算的误差导致无法和具体的浮点数值比较。浮点小数点前的0可以省略。

```javascript
var float1 = .1; // 表示1.1，但不推荐
var float2 = 1., float3 = 10.0; // 分别解析位整数1，10
var float4 = 3.125e7, float5 = 0.00000000000000003; // 等效于31250000，3e-17

var a = 0.1, b = 0.2;
a + b == 0.3; // false。a+b实际等于0.30000000000000004
```

数值范围最小值`Number.MIN_VALUE` 等于 5e-324，最大值`Number.MAX_VALUE`等于1.7976931348623157e+308，如果超出范围则分别转换成`-Infinity 或 Number.NEGATIVE_INFINITY`（负无穷）和`Infinity 或 Number.POSITIVE_INFINITY`（正无穷），可以使用`isFinite()`判断是否在最大小值之间。

NaN，即非数值（Not aNumber）是一个特殊值。表示本来要返回数值但未返回数值（这样来避免抛出错误），任何NaN参与的运算结果都是NaN。使用isNaN()判断是否能转换成数值。

##### 数值转换

把非数值转为数值：Number()——适用于任何类型，parseInt()和parseFloat()适用于字符串。但同样输入有不同返回结果。

Number()将null、空字符串转为0，undefined转为NaN，Boolean转为1和0，十六进制、正负号、浮点格式等包含数字的字符串均能正常转换，其他转为NaN。

> isNaN接收对象时，先调用对象的valueOf()返回值进行转换，如果不能则使用返回值调用toString()再转换，如果还是失败则返回NaN。
>
> Number()接收对象时，先调用对象的valueOf()返回值按照上述规则转换，如果结果是NaN则调用对象toString()方式再按照规则转换。

parseInt()、parseFloat() 更多是看字符串符合数值模式，即逐个字符解析，直到解析完毕或遇见非数字字符。区别在于后者①遇到第二个小数点就停止解析②不接受第二个参数，只能按照十进制解析。

#### String

0个及以上16位unicode字符组成的字符序列。如果包含双字节字符，则length可能不会精确返回字符数目。

> 字符串一旦创建就不能改变。要改变某个变量的字符串，则是先销毁原有字符串，再用新字符串填充该变量。
>
> ```javascript
> var lang = 'Java';
> lang = lang + 'Script'; 
> 
> // ①先创建新长度的字符串，然后用'Java'和'Script'去填充，接着销销原字符串'Java'和'Script'
> ```

Number、Boolean、Object和String使用toString() 返回新字符串，Null和Undefined除外。*当是Number时，可指定有效进制格式的字符串输出*。

String()接收任何类型的值进行字符串转换，内部调用toString()方法，Null返回'null'，Undefined返回'undefined'。

```javascript
var num = 10;
num.toString(); // '10'
num.toString(2); // '1010'

var value1 = true;
value1.toString(); // 'true'

String(null); // 'null'
```

#### Object

Object是所有对象的基础，具有下列属性和方法。

- constructor：构造函数。
- hasOwnProperty(propertyName)：用于检查给定的属性在当前对象实例（而不是在实例的原型中）是否存在。
- isPropertyOf(object)：检查传入的对象是否是当前对象的原型。
- propertyIsEnumerable(propertyName)：检查给定的属性是否能够使用for-in语句来枚举。
- toLocaleString()：返回与执行环境对应的字符串。
- toString()：返回对象的字符串表示。
- toValue()：返回对象的字符串、数值或布尔值表示。通常与toString()方法返回值相同。

### 操作符

#### 一元操作符

用于基本的算数运算。若是非数值时，则先转换为Number再运算。

- 递增递减：共4种。
- 一元加减：共2种，如a = -a, a = +a;

#### 位操作符

ECMAScript所有数值都是以IEEE-754 64位格式存储的。但位运算是将64位值转换为32位，执行操作后再转回64位，因此我们只关心32位的整数存储。

有符号的整数，最高位是符号位，表示数值符号：0 - 正数、1 - 负数。负数使用的格式是二进制补码，计算补码的三个步骤（以-18为例）：
- 求得负数的绝对值的二进制码。先得到18的二进制：1 0010。
- 求二进制反码。即0替换为1，1替换为0：1111 1111 1111 1111 1111 1111 1110 1101。
- 反码加1。得到1111 1111 1111 1111 1111 1111 1110 1110，即为-18的二进制表示。

> 在处理有符号整数时，不能访问位31的。默认情况下ECMAScript所有整数都是有符号的，但也有无符号的。`NaN`和`Infinity`进行位操作时，均被看作0来参与运算。

实际中负数的展现形式是其绝对值前面加了一个符号。

```javascript
var num = -18;
num.toString(2); // '-10010'
```

按位非（~），即取反码。下面代码验证了按位非得本质：操作数的负值减1，但~运算速度更快。

```javascript
var num1 = 25;
~num1 === -26; // true
-num1 - 1 === -26; // true
```

按位与（&）：都是1时才是1，其他均为0。

按位或（|）：有1则为1，其他为0。

按位异或（^）：只有1个1时为1，其他均为0。

左移（<<）、有符号的右移（>>）：符号位不变。符号位不变。

无符号位右移（>>>）：正整数时与有符号右移>>结果一样，负整数时符号位也要参与移位。*不存在无符号左移！*

#### 布尔运算符

逻辑非（！）：空字符串、数值0、null、NaN、undefined等返回true，对象、非空字符串、非0数值、Infinity等返回false。

逻辑与（&&）、逻辑或（||）：属于短路操作，即第一个操作数（即是false或true时，参考逻辑非）能决定结果，那不对第二个操作数求值。

#### 乘性操作符

乘法（*）、除法（/）、求模（%，也称求余）遇到非数值时会自动使用Number进行转换。结果有可能是+Infinity、-Infinity、NaN或正常结果，符号位取决于操作数的符号位。

加法操作符

加法（+）：结果可能有+Infinity、-Infinity、NaN、+0、-0、字符串、数值。有操作数是字符串时，则将是对象、数值、布尔值、undefined、null的操作数使用String()取得字符串后，再进行字符串拼接。

```javascript
var ret1 = 5 + 5; // 10
var ret2 = 5 + '5'; // '55'
var ret3 = 'this' + 5 + 10; // 'this510'，而不是'this15'
```

减法（-）：结果可能有+Infinity、-Infinity、NaN、+0、-0、数字。当操作数是字符串、布尔值、null、undefined时调用Number()得到数值，当操作数是对象时优先使用valueOf()得到数值、没有valueOf()时则使用toString()得到字符串再Number()得到数值，然后再进行数值减法计算。

#### 关系操作符

小于（<）、大于（>）、小于等于（<=）、大于等于（>=），有下面注意点（优先级从上到下）：

- 有数值时，其他类型转为数值后进行比较；
- 有字符串时，对字符串同位置的字符进行字符编码比较；
- 有对象时，使用 valueOf() || toString() 结果按照规则来比较；
- 有Boolean时，转为数值比较；

#### 相等操作符

相等（==）、不相等（！=）：会先转换操作数（通常称为**强制转型**）后再比较相等性。

- 有数值、布尔、字符串是，转为数值再比较；
- 对象使用valueOf()转换，再使用第一条规则；
- null和undefined是相等的，也不能转为其他值；
- NaN在==时返回false，!=时返回false，所以NaN不等于NaN；
- 如果两个操作数都是对象，那比较它们是不是同一个对象，即指向同一个对象；

全等（===）、不全等（!==）：不会类型转换，直接比较。

#### 其他操作符

条件操作符，即三目运算符。

赋值操作符，即=、*=、/=、%=、+=、-=、<<=、>>=、>>>=，除=外其他均是简化赋值操作，但不会有任何性能的提升。

逗号操作符，即同一个语句中执行多个操作，如`var a = (5, 1, 0); // a===0`。

### 语句

if语句里的 condition（条件）求值结果不一定是布尔值，但会被自动调用Boolean()进行转换。

for-in 是一种精准迭代语句，可用来枚举对象的属性，但属性名的顺序不可预测。

```javascript
for (var propName in obj) {
	console.log(propName, obj[propName]);
}
```

switch的case值，可以是常量、变量、表达式等其中一种。

### 函数

JavaScript函数的参数提供使用便利，也可以通过函数体内的arguments参数数组来访问传入的参数，所以但参数名不是必须的。没有传递值得参数自动被赋予为undefined。

>  所有参数都是值传递，不是引用传递。
>
> 没有函数重载，后定义的会覆盖先定义的。如果想实现重载，则需要利用参数的类型和数量，然后内部实现不同逻辑。

```javascript
function sum(num1, num2) {
	arguments[1] = 10;
	return arguments[0] + num2;
}
var a = sum(1, 2); // 11
var b = sum(1, undefined); // 11
var c = sum(1, null); // 11
var d = sum(1); // NaN。undefined被转为NaN
```

arguments对象中的值会自动映射到对应命名参数，所以修改arguments[1]，也就修改了num2。但这两个值的内存空间是独立的，不过它们的值会同步。如果只传入一个参数，那么修改arguments[1]不会同步到num2，因为arguments数组的长度是由实参决定的。

```javascript
function sum(num1, num2 = 1) {
	arguments[1] = 10;
	return arguments[0] + num2;
}
var a = sum(1, 2); // 3
var b = sum(1, undefined); // 2
var c = sum(1, null); // 1。null被转为0
var d = sum(1); // 2
```

