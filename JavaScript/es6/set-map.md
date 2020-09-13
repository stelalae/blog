# set-map

[ECMAScript 6 入门 - Set 和 Map 数据结构](http://es6.ruanyifeng.com/#docs/set-map)

[toc]

### Set

### 基本用法

 Set类似于数组，但是成员的值都是唯一的，没有重复的值。Set本身是一个构造函数，用来生成 Set 数据结构。可以接受一个数组（或者具有 iterable 接口的其他数据结构）作为参数，用来初始化。
```JavaScript
// 初始化
const set = new Set([1, 2, 3, 4, 4]);
[...set] // [1, 2, 3, 4]

const s = new Set();
[2, 3, 5, 4, 5, 2, 2].forEach(x => s.add(x));

s.add(1).add(2).add(2);

// 去除数组的重复成员
[...new Set(array)]
Array.from(new Set([1, 1, 2, 3]));  //  [1, 2, 3]

// 字符串去重
[...new Set('ababbc')].join('') // "abc"
```
向 Set 加入值的时候，不会发生类型转换，所以5和"5"是两个不同的值。Set 内部判断两个值是否不同，使用的算法叫做“Same-value-zero equality”，它类似于精确相等运算符（===），主要的区别是向 Set 加入值时认为NaN等于自身，而精确相等运算符认为NaN不等于自身。两个空对象被认为不相等。

#### Set 实例的属性和方法

Set 结构的实例有以下属性。
* Set.prototype.constructor：构造函数，默认就是Set函数。
* Set.prototype.size：返回Set实例的成员总数。

Set 实例的方法分为两大类：操作方法（用于操作数据）和遍历方法（用于遍历成员）。下面先介绍四个操作方法。
* Set.prototype.add(value)：添加某个值，返回 Set 结构本身。
* Set.prototype.delete(value)：删除某个值，返回一个布尔值，表示删除是否成功。
* Set.prototype.has(value)：返回一个布尔值，表示该值是否为Set的成员。
* Set.prototype.clear()：清除所有成员，没有返回值。

Array.from方法可以将 Set 结构转为数组。

#### 遍历操作

Set 结构的实例有四个遍历方法，可以用于遍历成员。
* Set.prototype.keys()：返回键名的遍历器
* Set.prototype.values()：返回键值的遍历器
* Set.prototype.entries()：返回键值对的遍历器
* Set.prototype.forEach()：使用回调函数遍历每个成员

需要特别指出的是，Set的遍历顺序就是插入顺序。这个特性有时非常有用，比如使用 Set 保存一个回调函数列表，调用时就能保证按照添加顺序调用。
由于 Set 结构没有键名，只有键值（或者说键名和键值是同一个值），所以keys方法和values方法的行为完全一致。entries方法返回的遍历器，同时包括键名和键值，所以每次输出一个数组，它的两个成员完全相等。这意味着，可以省略values方法，直接用for...of循环遍历 Set。
```JavaScript
let set = new Set(['red', 'green', 'blue']);

for (let x of set) {
  console.log(x);
}
```

### WeakSet

WeakSet 结构与 Set 类似，也是不重复的值的集合。但是，它与 Set 有两个区别。
- 首先，WeakSet 的成员只能是对象，而不能是其他类型的值。
- 其次，WeakSet 中的对象都是弱引用，即垃圾回收机制不考虑 WeakSet 对该对象的引用，也就是说，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于 WeakSet 之中。

因此，WeakSet 适合临时存放一组对象，以及存放跟对象绑定的信息。只要这些对象在外部消失，它在 WeakSet 里面的引用就会自动消失。因此 ES6 规定 WeakSet 不可遍历。

这些特点同样适用于本章后面要介绍的 WeakMap 结构。

### Map

### 基本用法

JavaScript 的对象（Object），本质上是键值对的集合（Hash 结构），但是传统上只能用字符串当作键，这给它的使用带来了很大的限制。

ES6 提供了 Map 数据结构。它类似于对象，但是“键”可以是各种类型的值（包括对象）。即Map 结构提供了“值—值”的对应，是一种更完善的 Hash 结构实现。如果你需要“键值对”的数据结构，Map 比 Object 更合适。
```JavaScript
const m = new Map();
const o = {p: 'Hello World'};

m.set(o, 'content')
m.get(o) // "content"

m.has(o) // true
m.delete(o) // true
m.has(o) // false
```

不仅仅是数组，任何具有 Iterator 接口、且每个成员都是一个双元素的数组的数据结构（详见《Iterator》一章）都可以当作Map构造函数的参数。这就是说，Set和Map都可以用来生成新的 Map。如果对同一个键多次赋值，后面的值将覆盖前面的值。读取一个未知的键，则返回undefined。
```JavaScript
const set = new Set([
  ['foo', 1],
  ['bar', 2]
]);
const m1 = new Map(set);
m1.get('foo') // 1

const m2 = new Map([['baz', 3]]);
const m3 = new Map(m2);
m3.get('baz') // 3


const map = new Map();
map.set(1, 'aaa').set(1, 'bbb');
map.get(1) // "bbb"


new Map().get('asfddfsasadf')
// undefined
```
**注意，只有对同一个对象的引用，Map 结构才将其视为同一个键。这一点要非常小心。**
```JavaScript
const map = new Map();
map.set(['a'], 555);
map.get(['a']) // undefined


const map = new Map();
const k1 = ['a'];
const k2 = ['a'];

map.set(k1, 111).set(k2, 222);

map.get(k1) // 111
map.get(k2) // 222
```
由上可知，Map 的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键。这就解决了同名属性碰撞（clash）的问题，我们扩展别人的库的时候，如果使用对象作为键名，就不用担心自己的属性与原作者的属性同名。

如果 Map 的键是一个简单类型的值（数字、字符串、布尔值），则只要两个值**严格相等**，Map 将其视为一个键，比如0和-0就是一个键，布尔值true和字符串true则是两个不同的键。另外，undefined和null也是两个不同的键。虽然NaN不严格相等于自身，但 Map 将其视为同一个键。
```JavaScript
let map = new Map();

map.set(-0, 123);
map.get(+0) // 123

map.set(true, 1);
map.set('true', 2);
map.get(true) // 1

map.set(undefined, 3);
map.set(null, 4);
map.get(undefined) // 3

map.set(NaN, 123);
map.get(NaN) // 123
```

#### 实例的属性和操作方法

（1）size ：返回 Map 结构的成员总数。
（2）set：设置键名key对应的键值为value，然后返回整个 Map 结构，因此可以采用链式写法。如果key已经有值，则键值会被更新，否则就新生成该键。
（3）get：读取key对应的键值，如果找不到key，返回undefined。
（4）has：返回一个布尔值，表示某个键是否在当前 Map 对象之中。
（5）delete：删除某个键，返回true。如果删除失败，返回false。
（6）clear：清除所有成员，没有返回值。

#### 遍历方法

Map 结构原生提供三个遍历器生成函数和一个遍历方法。需要特别注意的是，Map 的遍历顺序就是插入顺序。
* Map.prototype.keys()：返回键名的遍历器。
* Map.prototype.values()：返回键值的遍历器。
* Map.prototype.entries()：返回所有成员的遍历器。
* Map.prototype.forEach()：遍历 Map 的所有成员。

结合数组的map方法、filter方法，可以实现 Map 的遍历和过滤（Map 本身没有map和filter方法）。
```JavaScript
const map0 = new Map()
  .set(1, 'a')
  .set(2, 'b')
  .set(3, 'c');

const map1 = new Map(
  [...map0].filter(([k, v]) => k < 3)
);
// 产生 Map 结构 {1 => 'a', 2 => 'b'}

const map2 = new Map(
  [...map0].map(([k, v]) => [k * 2, '_' + v])
    );
// 产生 Map 结构 {2 => '_a', 4 => '_b', 6 => '_c'}
```

#### 与其他数据结构的互相转换

（1）Map 转为数组：推荐使用扩展运算符（...）。
（2）数组 转为 Map：将数组传入 Map 构造函数。
（3）Map 转为对象：如果所有 Map 的键都是字符串，它可以无损地转为对象。如果有非字符串的键名，那么这个键名会被转成字符串，再作为对象的键名。
（4）对象转为 Map：使用对象的entries。
（5）Map 转为 JSON：Map 的键名都是字符串，这时可以选择转为对象 JSON。Map 的键名有非字符串，这时可以选择转为数组 JSON。
（6）JSON 转为 Map：正常情况下，所有键名都是字符串。整个 JSON 就是一个数组，且每个数组成员本身，又是一个有两个成员的数组。这时，它可以一一对应地转为 Map。这往往是 Map 转为数组 JSON 的逆操作。

### WeakMap

WeakMap与Map的区别有两点。
* WeakMap只接受对象作为键名（null除外），不接受其他类型的值作为键名。
* WeakMap的键名所指向的对象，不计入垃圾回收机制。

注意，WeakMap 弱引用的只是键名，而不是键值。键值依然是正常引用。
```JavaScript
const wm = new WeakMap();
let key = {};
let obj = {foo: 1};

wm.set(key, obj);
obj = null;
wm.get(key)
// Object {foo: 1}
```

其他的参考上面要介绍的 WeakSet 结构。