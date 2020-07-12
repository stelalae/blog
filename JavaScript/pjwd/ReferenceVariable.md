# 引用类型

[TOC]

## Object类型

Object的属性名都是字符串，即数值等其他类型会转换为字符串。创建实例的方法：使用new操作符、字面量表示法，后者不会调用构造函数。

```javascript
var p1 = new Object(); // new操作符
p1.name = 'lyc';

var p2 = { name: 'lei' }; // 字面量表示法
```

## Array类型

JavaScript数组大小可动态调整的，可随着数据的添加自动扩容。创建实例的方法：使用new操作符、字面量表示法，后者不会调用构造函数，未初始化的数据项都是undefined。

```javascript
var arr1 = new Array(); // new操作符
var arr2 = [];	// 字面量表示法

var arr3 = new Array(3);	// 创建长度为3的数组，且每项值为undefined
var arr4 = new Array('1', '2'); // 创建包含2个内容的数组
var arr5 = new Array(4).fill('');  // 创建长度为4，且每项初始化为空字符串
```

数组的length不是只读的，可以改变其值来更改数组长度。可通过操作符`instanceof`和函数`Array.isArray()`来判断是否是数组，建议使用后者。

- `valueOf()`： 返回数组本身。
- `toString()/toLocalString()`：数组中每项都调用自身的toString()/toLocalString()，拼接成以逗号分割的字符串。等效于`join()、join(',')`。如果数组某一项是null、undefined、则在前面几个函数的结果均以空字符串表示。

### 栈与队列

栈是一种LIFO（Last-In-First-Out，后进先出）的数据结构。栈中项的插入（推入）和移除（弹出）只发生在栈的顶端，数组的 push() 和 pop() 分别来实现对应行为。*push()可以接受任意数量参数。*

队列是FIFO（First-In-First-Out，先进先出）的数据结构，分别用 push() 和 shift() 来实现添加和移除功能。另 unshift() 可以接受任意数量参数，实现在shift()操作的同侧的插入。

### 操作方法

排序：reverse()反转数组、sort()/sort(v1, v2) 按照升序（调用每个数组项toString()得到字符串后进行比较）或自定义排序方式（根据-1升序、1降序、0不变），均会改变原数组。

concat()：合并数组。创建原数组的副本，然后追加要被加入的数据项。

slice()：选取子数组。创建新数组，接收起始位置（可正可负）和长度，可能返回空数组。

splice：删除、插入、替换。返回从原数组删除的数据项，没有删除则返回空数组。参数：(start, offset, ...)，如果offset不能为0就会删除数据，offset之后的任意数量的参数表示被插入的数据。

indexOf()/lastIndexOf()：查找的项和（可选的）查找起始位置，没找到返回-1，使用全等操作符。

迭代方法，接收三个参数：数组项的值、该项在数组中的位置、数组本身。

- ervery()：每项返回true，则返回true。
- filter()：由返回true的项组成新数组。
- forEach()：便利数组，没有返回值。
- map()：创建新的数组，长度与原数组保持一致。
- some()：有一项返回true，则返回true。

reduce()/reduceRight()：数组归并，接收两个参数：遍历函数（前一个值、当前值、当前值的索引值、数组）、归并的基础初始值（可选）。

> ①如果基础初始值不为空时，第一次遍历的【前一个值】就是基础初始值；
>
> ②如果没有初始值，那第一次遍历的当前值是从数组的第二项，即遍历函数会执行length()-1次。所以当数组只有一项且没有初始值，数组不会遍历。

## Date类型

构造方式：

```javascript
var now1 = new Date(); // 取得当前时间
var now11 = Date.now(); // 同上

var now2 = new Date(Date.parse('May 25, 2004')); // parse()没有指定日期格式，通常因地区而已
var now22 = new Date('May 25, 2004'); // 内部调用的parse()，入参同parse()

var now3 = new Date(Date.UTC(2000, 4, 5, 17, 55, 55, 999)); // 只有年是必填的，其他默认对应取值范围的最小值。
// 除年、日（取值范围是[1,28/29/30/31]）外，其他参数取值范围均是[0, 该值与上一级的换算值减一]，如月为[0,11]，时是[0,23]。
var now3 = new Date(2000, 4, 5, 17, 55, 55, 999); // 内部模仿UTC()，区别在于基于时区不一致。UTC()是基于GMT，而本构造函数是基于本地时区。
```

> 笔者注，在UTC()除年外的参数里，如果取值不是对应范围，当超过最大值时会往上一级累加，当小于最小值时会找上一级借用（抹平负数，剩余的留着）。

toLocaleString() 返回当前时区的相适应格式字符串，toString() 返回带有时区信息的字符串，toUTCString() 返回UTC日期字符串（常用！），valueOf() 返回日期的毫秒表示。

toDateString()/toLocaleString()、toTimeString()/toLocaleTimeString()：分别返回日期（带星期）、时间（带时区）的字符串。

其他对Date的 get/set、getUTC/setUTC 操作有：Time（毫秒，等效于valueOf()。没有UTC版本）、FullYear（四位数的年份）、Month、Date（天数）、Day（星期）、Hours、Minutes、Seconds、Milliseconds，getTimezoneOffset() 返回本地时区与UTC时间相差的分钟数。

## RegExp类型

（跳过）

## Function类型

**函数实际上是对象，每个函数都是Function类型的实例。因此函数名实际上一个指向函数对象的指针，不会与某个函数绑定！！！** 

```javascript
function sum1 (num1, num2) {
	return num1 + num2;
}
// 等效于。下面使用了Function构造函数，可以接受任意数量参数
var sum2 = function(num1, num2) {
	return num1 + num2;
}
```

所以函数没有重载，后定义的函数名称会重新指向第二个函数实例。

上面代码中 sum1 是函数声明，sum2 是函数表达式。

> 在实际的执行环境中，解析器优先读取函数声明并使其在任何代码之前可用，而函数表达式则必须等到解析器执行它所在的代码行才会被真正解析执行。除此之外，其他都是等价的。

### 函数内部属性

**有两个特殊的对象：arguments 和 this。**

arguments有一个名叫 `callee` 的属性，指向拥有arguments对象的函数。经典应用：消除递归函数的函数名耦合。

```javascript
// 阶乘函数
function factorial(num) {
	if(num <= 1) {
		return 1;
	} else {
		return num * arguments.callee(num - 1);
	}
}

var trueFactorial = factorial;
factorial = function() {
  return 0;
}

trueFactorial(5); // 25
factorial(5); // 0
```

> 在ES6里默认严格模式，callee 不被推荐使用，参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/arguments)。还有`arguments.caller`也不推荐使用，原因是加强语言安全性，第三方代码不能窥视其他代码了。

this 引用的是函数执行的环境对象——或说是this值。函数名称仅仅是一个包含指针的变量而已，即使在不同环境中执行，但下面sayColor()和o.sayColor()仍然执行同一个函数代码。

```javascript
window.color = 'red';
var o = { color: 'blue' };

function sayColor() {
	console.log(this.color);
}

sayColor(); // red

o.sayColor = sayColor;
o.sayColor(); // blue
```

### 函数属性和方法

每个函数均有length、property属性，前者表示希望接收的命名参数的个数。

