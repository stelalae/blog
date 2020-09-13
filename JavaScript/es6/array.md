#  数组的扩展

[ECMAScript 6 入门 - 数组的扩展](http://es6.ruanyifeng.com/#docs/array)

[toc]


### 数组的扩展

扩展运算符（spread）是三个点（...）。它好比 rest 参数的逆运算，将一个数组转为用逗号分隔的参数序列，主要用于函数调用。
```JavaScript
function f(v, w, x, y, z) { }
const args = [0, 1];
f(-1, ...args, 2, ...[3]);

const arr = [
  ...(x > 0 ? ['a'] : []),
  'b',
];

[...[], 1]

(...[1, 2]) // Uncaught SyntaxError: Unexpected number

console.log((...[1, 2])) // Uncaught SyntaxError: Unexpected number

console.log(...[1, 2]) // 1 2
```
扩展运算符后面还可以放置表达式。只有函数调用时，扩展运算符才可以放在圆括号中，否则会报错。

扩展运算符的应用：
1、复制数组：避免浅拷贝
```JavaScript
const a1 = [1, 2];
// 写法一
const a2 = [...a1];
// 写法二
const [...a2] = a1;
```

2、合并数组
```JavaScript
const a1 = [{ foo: 1 }], a2 = [{ bar: 2 }];

const a3 = a1.concat(a2);
const a4 = [...a1, ...a2];
```
a3和a4是用两种不同方法合并而成的新数组，但是它们的成员都是对原数组成员的引用，这就是浅拷贝。

3、与解构赋值结合：扩展运算符用于数组赋值，只能放在参数的最后一位。

4、字符串：将字符串转为真正的数组，能够正确识别四个字节的 Unicode 字符。
```JavaScript
'x\uD83D\uDE80y'.length // 4，error
[...'x\uD83D\uDE80y'].length // 3，ok

function length(str) {
  return [...str].length;
}
length('x\uD83D\uDE80y') // 3
```

5、实现了 Iterator 接口的对象：任何定义了遍历器（Iterator）接口的对象（参阅 Iterator 一章），都可以用扩展运算符转为真正的数组。
6、Map 和 Set 结构，Generator 函数：扩展运算符内部调用的是数据结构的 Iterator 接口，因此只要具有 Iterator 接口的对象，都可以使用扩展运算符，比如 Map 结构。

### Array.from

将两类对象转为真正的数组：类似数组的对象（array-like object）和可遍历（iterable）的对象（包括 ES6 新增的数据结构 Set 和 Map）。扩展运算符（...）通过调用遍历器接口（Symbol.iterator）将某些数据结构转为数组，Array.from方法还支持有length属性类似数组的对象转为数组，而此时扩展运算符就无法转换。Array.from还可以接受第二个参数，作用类似于数组的map方法，用来对每个元素进行处理，将处理后的值放入返回的数组。
```JavaScript
[...document.querySelectorAll('div')]

Array.from({ length: 3 });
// [ undefined, undefined, undefined ]

Array.from([1, , 2, , 3], (n) => n || 0)
// [1, 0, 2, 0, 3]
```

### Array.of

用于将一组值，转换为数组。
```JavaScript
Array() // []
Array(3) // [, , ,]
Array(3, 11, 8) // [3, 11, 8]

Array.of() // []
Array.of(undefined) // [undefined]
Array.of(1) // [1]
Array.of(1, 2) // [1, 2]

// Array.of的模型
function ArrayOf(){
  return [].slice.call(arguments);
}
```
上面代码中，Array方法没有参数、一个参数、三个参数时，返回结果都不一样。只有当参数个数不少于 2 个时，Array()才会返回由参数组成的新数组。参数个数只有一个时，实际上是指定数组的长度。Array.of基本上可以用来替代Array()或new Array()，并且不存在由于参数不同而导致的重载。它的行为非常统一。

### copyWithin()

在当前数组内部，将指定位置的成员复制到其他位置（会覆盖原有成员），然后返回当前数组。使用这个方法，会修改当前数组。它接受三个参数：
* target（必需）：从该位置开始替换数据。如果为负值，表示倒数。
* start（可选）：从该位置开始读取数据，默认为 0。如果为负值，表示从末尾开始计算。
* end（可选）：到该位置前停止读取数据，默认等于数组长度。如果为负值，表示从末尾开始计算。

这三个参数都应该是数值，如果不是，会自动转为数值。
```JavaScript
// 将3号位复制到0号位
[1, 2, 3, 4, 5].copyWithin(0, 3, 4)
// [4, 2, 3, 4, 5]

// -2相当于3号位，-1相当于4号位
[1, 2, 3, 4, 5].copyWithin(0, -2, -1)
// [4, 2, 3, 4, 5]

// 将3号位复制到0号位
[].copyWithin.call({length: 5, 3: 1}, 0, 3)
// {0: 1, 3: 1, length: 5}

// 将2号位到数组结束，复制到0号位
let i32a = new Int32Array([1, 2, 3, 4, 5]);
i32a.copyWithin(0, 2);
// Int32Array [3, 4, 5, 4, 5]
```

### find() 、 findIndex()

数组实例的find方法，用于找出第一个符合条件的数组成员。它的参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为true的成员，然后**返回该成员**。如果没有符合条件的成员，则返回undefined。**find方法的回调函数可以接受三个参数，依次为当前的值、当前的位置和原数组。**
```JavaScript
[1, 5, 10, 15].find(function(value, index, arr) {
  return value > 9;
}) // 10
```
数组实例的findIndex方法的用法与find方法非常类似，返回第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回-1。
```JavaScript
[1, 5, 10, 15].findIndex(function(value, index, arr) {
  return value > 9;
}) // 2
```

### fill

使用给定值，填充一个数组。数组中已有的元素，会被全部抹去。fill方法还可以接受第二个和第三个参数，用于指定填充的起始位置和结束位置。**如果填充的类型为对象，那么被赋值的是同一个内存地址的对象，而不是深拷贝对象。**
```JavaScript
['a', 'b', 'c'].fill(7, 1, 2)
// ['a', 7, 'c']

let arr = new Array(3).fill({name: "Mike"});
arr[0].name = "Ben";
arr
// [{name: "Ben"}, {name: "Ben"}, {name: "Ben"}]

let arr = new Array(3).fill([]);
arr[0].push(5);
arr
// [[5], [5], [5]]
```

entries()，keys() 和 values()

用于遍历数组。它们都返回一个遍历器对象（详见《Iterator》一章），可以用for...of循环进行遍历，唯一的区别是keys()是对键名的遍历、values()是对键值的遍历，entries()是对键值对的遍历。

includes()：查找数组中是否有某个值，没找到返回false。方法的第二个参数表示搜索的起始位置，默认为0。如果第二个参数为负数，则表示倒数的位置，如果这时它大于数组长度（比如第二个参数为-4，但数组长度为3），则会重置为从0开始。
```JavaScript
[1, 2, 3].includes(3, 3);  // false
[1, 2, 3].includes(3, -1); // true
```

Map 和 Set 数据结构有一个has方法，需要注意与includes区分。
* Map 结构的has方法，是用来查找键名的，比如Map.prototype.has(key)、WeakMap.prototype.has(key)、Reflect.has(target, propertyKey)。
* Set 结构的has方法，是用来查找值的，比如Set.prototype.has(value)、WeakSet.prototype.has(value)。

flat() ：用于将嵌套的数组“拉平”，变成一维的数组。该方法返回一个新数组，对原数据没有影响。如果原数组有空位，flat()方法会跳过空位。方法的第二个参数表示想要拉平的层数，默认为1，用Infinity关键字作为参数表示不管有多少层嵌套都要拉平。
```JavaScript
[1, , [3, [4, 5]]].flat()
// [1, 3, [4, 5]]

[1, 2, [3, [4, 5]]].flat(2)
// [1, 2, 3, 4, 5]

[1, [2, [3]]].flat(Infinity)
// [1, 2, 3]
```

flatMap()：对原数组的每个成员执行一个函数（相当于执行Array.prototype.map()），然后对返回值组成的数组执行flat()方法，只能展开一层数组。该方法返回一个新数组，不改变原数组。该方法的参数是一个遍历函数，该函数可以接受三个参数，分别是当前数组成员、当前数组成员的位置（从零开始）、原数组。
```JavaScript
// 相当于 [[[2]], [[4]], [[6]], [[8]]].flat()
[1, 2, 3, 4].flatMap(x => [[x * 2]])
// [[2], [4], [6], [8]]
```

### 数组的空位

指数组的某一个位置没有任何值。比如，Array构造函数返回的数组都是空位。空位不是undefined，一个位置的值等于undefined，依然是有值的。空位是没有任何值，in运算符可以说明这一点。
```JavaScript
0 in [undefined, undefined, undefined] // true
0 in [, , ,] // false
```

ES5 对空位的处理不一致，大多数情况下会忽略空位。
* forEach(), filter(), reduce(), every() 和some()都会跳过空位。
* map()会跳过空位，但会保留这个值
* join()和toString()会将空位视为undefined，而undefined和null会被处理成空字符串。
```JavaScript
// forEach方法
[,'a'].forEach((x,i) => console.log(i)); // 1

// filter方法
['a',,'b'].filter(x => true) // ['a','b']

// every方法
[,'a'].every(x => x==='a') // true

// reduce方法
[1,,2].reduce((x,y) => x+y) // 3

// some方法
[,'a'].some(x => x !== 'a') // false

// map方法
[,'a'].map(x => 1) // [,1]

// join方法
[,'a',undefined,null].join('#') // "#a##"

// toString方法
[,'a',undefined,null].toString() // ",a,,"
```
ES6 则是明确将空位转为undefined。Array.from、扩展运算符（...）、entries()、keys()、values()、find()和findIndex()将空位转为undefined，copyWithin()会拷贝空位，fill()将空位视为正常的数组位置，for...of循环也会遍历空位。
```JavaScript
Array.from(['a',,'b'])
// [ "a", undefined, "b" ]

[...['a',,'b']]
// [ "a", undefined, "b" ]

// entries()
[...[,'a'].entries()] // [[0,undefined], [1,"a"]]

// keys()
[...[,'a'].keys()] // [0,1]

// values()
[...[,'a'].values()] // [undefined,"a"]

// find()
[,'a'].find(x => true) // undefined

// findIndex()
[,'a'].findIndex(x => true) // 0

[,'a','b',,].copyWithin(2,0) // [,"a",,"a"]

new Array(3).fill('a') // ["a","a","a"]

let arr = [, ,];
for (let i of arr) {
  console.log(1);
}
// 1
// 1
```
由于空位的处理规则非常不统一，所以建议避免出现空位。

### sort() 

排序稳定性（stable sorting）是排序算法的重要属性，指的是排序关键字相同的项目，排序前后的顺序不变。常见的排序算法之中，插入排序、合并排序、冒泡排序等都是稳定的，堆排序、快速排序等是不稳定的。

不稳定排序的主要缺点是，多重排序时可能会产生问题。假设有一个姓和名的列表，要求按照“姓氏为主要关键字，名字为次要关键字”进行排序。开发者可能会先按名字排序，再按姓氏进行排序。如果排序算法是稳定的，这样就可以达到“先姓氏，后名字”的排序效果。如果是不稳定的，就不行。
```JavaScript
const arr = [
  'peach',
  'straw',
  'apple',
  'spork'
];

// 稳定排序
const stableSorting = (s1, s2) => {
  if (s1[0] < s2[0]) return -1;
  return 1;
};
arr.sort(stableSorting) // ["apple", "peach", "straw", "spork"]

// 不稳定排序
const unstableSorting = (s1, s2) => {
  if (s1[0] <= s2[0]) return -1;
  return 1;
};
arr.sort(unstableSorting) // ["apple", "peach", "spork", "straw"]
```
早先 Array.prototype.sort()的默认排序算法是否稳定，留给浏览器自己决定。[ES2019](https://github.com/tc39/ecma262/pull/1340) 明确规定，Array.prototype.sort()的默认排序算法必须稳定。