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





### String

字符串的模式匹配方法

- match()：接收参数是一个正则表达式或RegExp对象，返回匹配结果的数组。等效于RegExp 对象的 exec()方法。

- search()：参数同match()，返回字符串中第一个匹配项的索引，否则返回-1。

```javascript
var text = "cat, bat, sat, fat";
var pattern = /.at/;
//与 pattern.exec(text)相同
var matches = text.match(pattern); 
alert(matches.index); //0
alert(matches[0]); //"cat"
alert(pattern.lastIndex); //0 

var pos = text.search(/at/);
alert(pos); //1 
```

- replace()：第一个参数RegExp 对象或者一个字符串，第二个参数是一个字符串或者一个函数。

  ![](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/07/14/EeOd3J.png)

replace() 第二个参数是一个函数时，该函数返回一个字符串。在只有一个匹配项（即与模式匹配的字符串）的情况下，会向这个函数传递 3 个参数：模式的匹配项、模式匹配项在字符串中的位置和原始字符串。在正则表达式中定义了多个捕获组的情况下，传递给函数的参数依次是模式的匹配项、第一个捕获组的匹配项、第二个捕获组的匹配项……，但最后两个参数仍然分别是模式的匹配项在字符串中的位置和原始字符串。

> 该模式匹配有关的方法是 split()，可基于指定的分隔符将一个字符串分割成 多个子字符串，并将结果放在一个数组中。

参见[MDN说明](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/replace#%E6%8C%87%E5%AE%9A%E4%B8%80%E4%B8%AA%E5%87%BD%E6%95%B0%E4%BD%9C%E4%B8%BA%E5%8F%82%E6%95%B0)和示例：

```javascript
function htmlEscape(text) {
  return text.replace(/[<>"&]/g, function(match, pos, originalText) {
    switch (match) {
      case '<':
        return '&lt;';
      case '>':
        return '&gt;';
      case '&':
        return '&amp;';
      case '"':
        return '&quot;';
    }
  });
}
alert(htmlEscape('<p class="greeting">Hello world!</p>'));
//&lt;p class=&quot;greeting&quot;&gt;Hello world!&lt;/p&gt;
```

localeCompare()：比较两个字符串大小，返回-1、0、1。

fromCharCode()：String构造函数的静态方法，接收N个字符编码并转换成一个字符串。从本质上来看与charCodeAt() 执行的是相反的操作。

```javascript
alert(String.fromCharCode(104, 101, 108, 108, 111)); //"hello" 
```

## Global对象

事实上，没有全局变量或全局函数；所有在全局作用域中定义的属性和函数，都是 Global 对象的属性。换句话说，不属于任何其他对象的属性和方法，最终都是它的属性和方法。

#### URI 编码方法

encodeURI() 和 encodeURIComponent() 用特殊的 UTF-8 编码替换 URI（Uniform Resource Identifiers，通用资源标识符）中所有无效的字符， 从而让浏览器能够接受和理解。对应的解码方法分别是：decodeURI() 和 decodeURIComponent()。

- encodeURI() 不会对本身属于 URI 的特殊字符进行编码，例如冒号、正斜杠、 问号和井字号。
- encodeURIComponent() 会对它发现的任何非标准字符进行编码。一般使用此方法！

####  eval()方法

像是一个完整的 ECMAScript 解析器，它只接受一个参数，即要执行的 ECMAScript（或 JavaScript） 字符串，然后把执行结果插入到原位置。eval() 执行的代码具有与该执行环境相同的作用域链。

> 在非严格模式下，eval() 执行的代码可以引用在包含环境中定义的变量，eval() 执行的代码中定义的函数和变量，也能在外部访问（但不存在变量提升）。

![](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/07/14/7X2hFk.png)

Web 浏览器都是将这个全局对象作为 window 对象的一部分加以实现的，在全局作用域中声明的所有变量和函数，就都成为了 window 对象的属性。



## Math对象

Math 对象提供的计算功能执行速度，比在 JavaScript 直接编写的计算快得多。

![icwZZW](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/07/14/icwZZW.png)

min() 和 max() 用于确定一组数值中的最小值和最大值，可以接收任意多个数值参数。

```javascript
var max = Math.max(3, 54, 32, 16);
alert(max); //54
var min = Math.min(3, 54, 32, 16);
alert(min); //3 

var values = [1, 2, 3, 4, 5, 6, 7, 8];
var max2 = Math.max.apply(Math, values);  // 使用apply
```

舍入方法：

- Math.ceil()：执行向上舍入，即它总是将数值向上舍入为最接近的整数；
- Math.floor()：执行向下舍入，即它总是将数值向下舍入为最接近的整数；
- Math.round()：执行标准舍入，即它总是将数值四舍五入为最接近的整数（这也是我们在数学课上学到的舍入规则）。

Math.random()方法返回大于等于 0 小于 1 的一个随机数。从某个整数范围内随机选择一个值：

```
值 = Math.floor(Math.random() * 可能值的总数 + 第一个可能的值) 
```

![cuClTw](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/07/14/cuClTw.png)

