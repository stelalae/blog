# 字符串的扩展

[ECMAScript 6 入门 - 字符串的扩展](http://es6.ruanyifeng.com/#docs/string)
[ECMAScript 6 入门 - 字符串的新增方法](http://es6.ruanyifeng.com/#docs/object-methods)

[TOC]

### 杂项介绍

字符的 Unicode 表示法：**将码点放入大括号**。
```JavaScript
"\u{20BB7}"
// "𠮷"

"\u{41}\u{42}\u{43}"
// "ABC"

let hello = 123;
hell\u{6F} // 123

'\u{1F680}' === '\uD83D\uDE80'
// true

// JavaScript 共有 6 种方法可以表示一个字符
'\z' === 'z'  // true
'\172' === 'z' // true
'\x7A' === 'z' // true
'\u007A' === 'z' // true
'\u{7A}' === 'z' // true
```

ES6 为字符串添加了遍历器接口`for...of`，这个遍历器最大的优点是可以识别大于0xFFFF的码点。
```JavaScript
for (let codePoint of 'foo') {
  console.log(codePoint) // "f"     "o"     "o"
}

let text = String.fromCodePoint(0x20BB7);  //  "\uD842\uDFB7" "\u{20BB7}"
for (let i = 0; i < text.length; i++) {
  console.log(text[i]);     // " "      " "
}
for (let i of text) {
  console.log(i);   // "𠮷"
}
```

根据标准JSON 数据必须是 UTF-8 编码。但JSON.stringify()方法有可能返回不符合 UTF-8 标准的字符串。[ES2019](https://github.com/tc39/proposal-well-formed-stringify) 改变了JSON.stringify()的行为。
```JavaScript
JSON.stringify('\u{D834}') // ""\\uD834""
JSON.stringify('\uDF06\uD834') // ""\\udf06\\ud834""
```

### 模板字符串

**模板字符串**表示多行字符串，所有的空格和缩进都会被保留在输出之中，可以使用trim方法消除换行。大括号内部可以放入任意的 JavaScript 表达式，可以进行运算和调用函数，以及引用对象属性，甚至还能嵌套。如果大括号中的值不是字符串，将按照一般的规则转为字符串。比如，大括号中是一个对象，将默认调用对象的toString方法。

“标签模板”功能（tagged template），指模板字符串紧跟在一个函数名后面，该函数将被调用来处理这个模板字符串。标签模板其实不是模板，而是函数调用的一种特殊形式。“标签”指的就是函数，紧跟在后面的模板字符串就是它的参数。如果模板字符里面有变量，会**将模板字符串先处理成多个参数，再调用函数**。
```JavaScript
let a = 5;
let b = 10;

tag`Hello ${ a + b } world ${ a * b }`;
// 等同于
tag(['Hello ', ' world ', ''], 15, 50);
```

“标签模板”的一个重要应用，就是过滤 HTML 字符串，防止用户输入恶意内容。
```JavaScript
function SaferHTML(templateData) {
  let s = templateData[0];
  for (let i = 1; i < arguments.length; i++) {
    let arg = String(arguments[i]);

    // Escape special characters in the substitution.
    s += arg.replace(/&/g, "&amp;")
            .replace(/</g, "&lt;")
            .replace(/>/g, "&gt;");

    // Don't escape special characters in the template.
    s += templateData[i];
  }
  return s;
}

let sender = '<script>alert("abc")</script>'; // 恶意代码
let message = SaferHTML`<p>${sender} has sent you a message.</p>`;
```

标签模板的另一个应用，就是多语言转换（国际化处理）。
```JavaScript
i18n`Welcome to ${siteName}, you are visitor number ${visitorNumber}!`
// "欢迎访问xxx，您是第xxxx位访问者！"
```

使用标签模板，在 JavaScript 语言之中嵌入其他语言。通过[sx函数](https://gist.github.com/lygaret/a68220defa69174bdec5)，将一个 DOM 字符串转为 React 对象。
```JavaScript
jsx`
  <div>
    <input
      ref='input'
      onChange='${this.handleChange}'
      defaultValue='${this.state.value}' />
      ${this.state.value}
   </div>
`
```

模板处理函数的第一个参数（模板字符串数组），还有一个raw属性，保存的是转义后的原字符串。可以看[String.raw()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/raw)
```JavaScript
console.log`123`
// ["123", raw: Array[1]]

tag`First line\nSecond line`

function tag(strings) {
  console.log(strings.raw[0]);
  // strings.raw[0] 为 "First line\\nSecond line"
  // 打印输出 "First line\nSecond line"
}
```
上面代码中，tag函数的第一个参数strings，有一个raw属性，也指向一个数组。该数组的成员与strings数组完全一致。比如，strings数组是["First line\nSecond line"]，那么strings.raw数组就是["First line\\nSecond line"]。两者唯一的区别，就是字符串里面的斜杠都被转义了。比如，strings.raw 数组会将\n视为\\和n两个字符，而不是换行符。这是为了方便取得转义之前的原始模板而设计的。

但是模板字符串默认会将字符串转义，导致无法嵌入其他语言。ES2018 [放松](https://tc39.github.io/proposal-template-literal-revision/)了对标签模板里面的字符串转义的限制。如果遇到不合法的字符串转义，就返回undefined，而不是报错，并且从raw属性上面可以得到原始字符串。

### 新增方法

#### String.fromCodePoint()

用于从 Unicode 码点返回对应字符，如果String.fromCodePoint方法有多个参数，则它们会被合并成一个字符串返回。
```JavaScript
String.fromCodePoint(0x20BB7)
// "𠮷"
String.fromCodePoint(0x78, 0x1f680, 0x79) === 'x\uD83D\uDE80y'
// true
```

#### String.raw()

返回一个斜杠都被转义（即斜杠前面再加一个斜杠）的字符串，往往用于模板字符串的处理方法。可以作为处理模板字符串的基本方法，它会将所有变量替换，而且对斜杠进行转义，方便下一步作为字符串来使用。

String.raw()本质上是一个正常的函数，只是专用于模板字符串的标签函数。如果写成正常函数的形式，它的第一个参数，应该是一个具有raw属性的对象，且raw属性的值应该是一个数组，对应模板字符串解析后的值。
```JavaScript
// `foo${1 + 2}bar`
// 等同于
String.raw({ raw: ['foo', 'bar'] }, 1 + 2) // "foo3bar"
```
上面代码中，String.raw()方法的第一个参数是一个对象，它的raw属性等同于原始的模板字符串解析后得到的数组。

#### charCodeAt

正确处理 4 个字节储存的字符，返回一个字符的码点。
```JavaScript
let s = '𠮷a';

s.codePointAt(0) // 134071
s.codePointAt(1) // 57271

s.codePointAt(2) // 97

s.codePointAt(0).toString(16) // "20bb7"
s.codePointAt(2).toString(16) // "61"
```
上面代码中，字符a在字符串s的正确位置序号应该是 1，但是必须向codePointAt()方法传入 2。解决这个问题的一个办法是使用for...of循环，因为它会正确识别 32 位的 UTF-16 字符。
```JavaScript
let s = '𠮷a';
for (let ch of s) {
  console.log(ch.codePointAt(0).toString(16));
}

// 等效于

let arr = [...'𠮷a']; // arr.length === 2
arr.forEach(
  ch => console.log(ch.codePointAt(0).toString(16))
);
```
codePointAt()方法是测试一个字符由两个字节还是由四个字节组成的最简单方法。
```JavaScript
function is32Bit(c) {
  return c.codePointAt(0) > 0xFFFF;
}

is32Bit("𠮷") // true
is32Bit("a") // false
```

#### normalize

许多欧洲语言有语调符号和重音符号，ES6 提供字符串实例的normalize()方法，用来将字符的不同表示方法统一为同样的形式，这称为 Unicode 正规化。可以接受一个参数来指定normalize的方式：
* NFC，默认参数，表示“标准等价合成”（Normalization Form Canonical Composition），返回多个简单字符的合成字符。所谓“标准等价”指的是视觉和语义上的等价
* NFD，表示“标准等价分解”（Normalization Form Canonical Decomposition），即在标准等价的前提下，返回合成字符分解的多个简单字符。
* NFKC，表示“兼容等价合成”（Normalization Form Compatibility Composition），返回合成字符。所谓“兼容等价”指的是语义上存在等价，但视觉上不等价，比如“囍”和“喜喜”。（这只是用来举例，normalize方法不能识别中文。）
* NFKD，表示“兼容等价分解”（Normalization Form Compatibility Decomposition），即在兼容等价的前提下，返回合成字符分解的多个简单字符。
```JavaScript
'\u01D1'==='\u004F\u030C' //false
'\u01D1'.normalize() === '\u004F\u030C'.normalize() // true

'\u01D1'.length // 1
'\u004F\u030C'.length // 2
```
normalize方法目前不能识别三个或三个以上字符的合成。这种情况下，还是只能使用正则表达式，通过 Unicode 编号区间判断。

#### includes(), startsWith(), endsWith()

确定一个字符串是否包含在另一个字符串中，除indexOf外，ES6新增：
* includes()：返回布尔值，表示是否找到了参数字符串。
* startsWith()：返回布尔值，表示参数字符串是否在原字符串的头部。
* endsWith()：返回布尔值，表示参数字符串是否在原字符串的尾部。
```JavaScript
let s = 'Hello world!';

s.startsWith('Hello') // true
s.endsWith('!') // true
s.includes('o') // true

// 支持指定搜索的位置。
s.startsWith('world', 6) // true
s.endsWith('Hello', 5) // true
s.includes('Hello', 6) // false
```
使用第二个参数n时，endsWith的行为与其他两个方法有所不同。它针对前n个字符，而其他两个方法针对从第n个位置直到字符串结束。

#### repeat()

返回一个新字符串，表示将原字符串重复n次。
```JavaScript
'x'.repeat(3) // "xxx"
'hello'.repeat(2) // "hellohello"
'na'.repeat(0) // ""

// 小数会被取整
'na'.repeat(2.9) // "nana"

// 负数或者Infinity，会报错RangeError
'na'.repeat(Infinity)
'na'.repeat(-1)
```
如果参数是 0 到-1 之间的小数，则等同于 0，这是因为会先进行取整运算。0 到-1 之间的小数，取整以后等于-0视同为 0，NaN等同于 0。如果是字符串，则会先转换成数字。
```JavaScript
'na'.repeat(-0.9) // ""
'na'.repeat(NaN) // ""
'na'.repeat('na') // ""
'na'.repeat('3') // "nanana"
```

#### padStart()，padEnd()

ES2017 引入了字符串补全长度的功能。如果某个字符串不够指定长度，**默认使用空格在头部或尾部补全**。
```
'x'.padStart(5, 'ab') // 'ababx'
'x'.padStart(4, 'ab') // 'abax'
'x'.padEnd(5, 'ab') // 'xabab'
'x'.padEnd(4, 'ab') // 'xaba'

'xxx'.padStart(2, 'ab') // 'xxx'
'xxx'.padEnd(2, 'ab') // 'xxx'
'abc'.padStart(10, '0123456789') // '0123456abc'

// 常见用途 - 数值补全指定位数
'1'.padStart(10, '0') // "0000000001"
'12'.padStart(10, '0') // "0000000012"

// 常见用途 - 提示字符串格式
'12'.padStart(10, 'YYYY-MM-DD') // "YYYY-MM-12"
'09-12'.padStart(10, 'YYYY-MM-DD') // "YYYY-09-12"
```

#### trimStart()，trimEnd()

[ES2019](https://github.com/tc39/proposal-string-left-right-trim) 对字符串实例新增了trimStart()消除字符串头部的空格，trimEnd()消除尾部的空格。它们的行为与trim()一致，返回的都是新字符串，不会修改原始字符串。除了空格键，对字符串头部（或尾部）的 tab 键、换行符等不可见的空白符号也有效。

#### matchAll()

返回一个正则表达式在当前字符串的所有匹配，详见[《正则的扩展》](http://es6.ruanyifeng.com/#docs/regex)的一章。