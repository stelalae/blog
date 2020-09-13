# 对象的扩展

[ECMAScript 6 入门 - 对象的扩展](http://es6.ruanyifeng.com/#docs/object)
[ECMAScript 6 入门 - 对象的新增方法](http://es6.ruanyifeng.com/#docs/string-methods)

[toc]

### 简洁表示法

```JavaScript
const foo = 'bar';
const baz = {foo};
// 等同于
const baz = {foo: foo};

const o = {
  name: "张三",
  method() {
    return "Hello!" + this.name;
  }
};
// 等同于
const o = {
  method: function() {
    return "Hello!" + this.name;
  }
};

// 不等同于。因为对象不单独构成作用域，this指向全局
const o = {
  method: () => {
    return "Hello!" + this.name;
  }
};
```

this关键字总是指向函数所在的当前对象，ES6 又新增了另一个类似的关键字super，指向当前对象的原型对象。
```JavaScript
const proto = {
  foo: 'hello'
};

const obj = {
  foo: 'world',
  find() {
    return super.foo;
  }
};

Object.setPrototypeOf(obj, proto);
obj.find() // "hello"
```

注意，super关键字表示原型对象时，只能用在对象的方法之中，用在其他地方都会报错。JavaScript 引擎内部，super.foo等同于Object.getPrototypeOf(this).foo（属性）或Object.getPrototypeOf(this).foo.call(this)（方法）。下面代码中，super.foo指向原型对象proto的foo方法，但是绑定的this却还是当前对象obj，因此输出的就是world。
```JavaScript
const proto = {
  x: 'hello',
  foo() {
    console.log(this.x);
  },
};

const obj = {
  x: 'world',
  foo() {
    super.foo();
  }
}

Object.setPrototypeOf(obj, proto);

obj.foo() // "world"
```


属性的赋值器（setter）和取值器（getter），事实上也是采用这种写法。注意，简写的对象方法不能用作构造函数，会报错。
```JavaScript
const cart = {
  _wheels: 4,

  f() {
    this.foo = 'bar';
  }
  
  get wheels () {
    return this._wheels;
  },

  set wheels (value) {
    if (value < this._wheels) {
      throw new Error('数值太小了！');
    }
    this._wheels = value;
  }
}

new cart.f() // 报错
```

JavaScript 定义对象的属性，有两种方法。方法一是直接用标识符作为属性名，方法二是用表达式作为属性名，这时要将表达式放在方括号之内。注意，属性名表达式与简洁表示法，不能同时使用。

### 属性的可枚举性和遍历

#### 可枚举性

对象的每个属性都有一个描述对象（Descriptor），用来控制该属性的行为。`Object.getOwnPropertyDescriptor`方法可以获取该属性的描述对象。
```JavaScript
let obj = { foo: 123 };
Object.getOwnPropertyDescriptor(obj, 'foo')
//  {
//    value: 123,
//    writable: true,
//    enumerable: true,
//    configurable: true
//  }
```
描述对象的`enumerable`属性，称为“可枚举性”，如果该属性为false，就表示某些操作会忽略当前属性。目前，有四个操作会忽略enumerable为false的属性。
* for...in循环：只遍历对象自身的和继承的可枚举的属性。
* Object.keys()：返回对象自身的所有可枚举的属性的键名。
* JSON.stringify()：只串行化对象自身的可枚举的属性。
* Object.assign()： 忽略enumerable为false的属性，只拷贝对象自身的可枚举的属性。

前三个是 ES5 就有的，其中只有for...in会返回继承的属性，其他三个方法都会忽略继承的属性，只处理对象自身的属性。
> 实际上，引入“可枚举”（enumerable）这个概念的最初目的，就是让某些属性可以规避掉for...in操作，不然所有内部属性和方法都会被遍历到。比如，对象原型的toString方法，以及数组的length属性，就通过“可枚举性”，从而避免被for...in遍历到。

另外ES6 规定所有 Class 的原型的方法都是不可枚举的。总的来说，引入继承的属性会让问题复杂化，大多数时候我们只关心对象自身的属性，所以**尽量不要用for...in循环，而用Object.keys()代替**。

#### 属性的遍历

ES6 一共有 5 种方法可以遍历对象的属性。
（1）for...in：for...in循环遍历对象自身的和继承的可枚举属性（不含 Symbol 属性）。
（2）Object.keys(obj)：返回一个数组，包括对象自身的（不含继承的）所有可枚举属性（不含 Symbol 属性）的键名。
（3）Object.getOwnPropertyNames(obj)：返回一个数组，包含对象自身的所有属性（不含 Symbol 属性，但是包括不可枚举属性）的键名。
（4）Object.getOwnPropertySymbols(obj)：返回一个数组，包含对象自身的所有 Symbol 属性的键名。
（5）Reflect.ownKeys(obj)：返回一个数组，包含对象自身的所有键名，不管键名是 Symbol 或字符串，也不管是否可枚举。

以上的 5 种方法遍历对象的键名，都遵守同样的属性遍历的次序规则。
- 首先遍历所有数值键，按照数值升序排列。
- 其次遍历所有字符串键，按照加入时间升序排列。
- 最后遍历所有 Symbol 键，按照加入时间升序排列。
```JavaScript
Reflect.ownKeys({ [Symbol()]:0, b:0, 10:0, 2:0, a:0 })
// ['2', '10', 'b', 'a', Symbol()]
```
上面代码中，Reflect.ownKeys方法返回一个数组，包含了参数对象的所有属性。这个数组的属性次序是这样的，首先是数值属性2和10，其次是字符串属性b和a，最后是 Symbol 属性。


#### 扩展运算符

扩展运算符（...）用于取出参数对象的所有可遍历属性，拷贝到当前对象之中。数组是特殊的对象，如果扩展运算符后面不是对象，则会自动将其转为对象。
```JavaScript
// 等同于 {...Object(1)}
{...1} // {}

// 等同于 {...Object(true)}
{...true} // {}

// 等同于 {...Object(undefined)}
{...undefined} // {}

// 等同于 {...Object(null)}
{...null} // {}

{...'hello'}
// {0: "h", 1: "e", 2: "l", 3: "l", 4: "o"}
```
上面第一个扩展运算符后面是整数1，会自动转为数值的包装对象Number{1}。由于该对象没有自身属性，所以返回一个空对象。

对象的扩展运算符等同于使用[Object.assign()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)方法，只是拷贝了对象实例的属性值，假如源对象的属性值是一个对象的引用，那么它也只指向那个引用。如果想完整克隆一个对象，还拷贝对象原型的属性，可以采用下面的写法。
```JavaScript
// 写法一
const clone1 = {
  __proto__: Object.getPrototypeOf(obj),
  ...obj
};

// 写法二，推荐
const clone2 = Object.assign(
  Object.create(Object.getPrototypeOf(obj)),
  obj
);

// 写法三，推荐
const clone3 = Object.create(
  Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj)
)
```
扩展运算符的参数对象之中，如果有取值函数get，这个函数是会执行的。
```JavaScript
// 并不会抛出错误，因为 x 属性只是被定义，但没执行
let aWithXGetter = {
  ...a,
  get x() {
    throw new Error('not throw yet');
  }
};

// 会抛出错误，因为 x 属性被执行了
let runtimeError = {
  ...a,
  ...{
    get x() {
      throw new Error('throw now');
    }
  }
};
```

### 链判断运算符

[ES2020](https://github.com/tc39/proposal-optional-chaining) 引入了“链判断运算符”（optional chaining operator）?.，调用的时候判断，左侧的对象是否为null或undefined。如果是的，就不再往下运算，而是**返回undefined。**
* obj?.prop // 对象属性
* obj?.[expr] // 同上
* func?.(...args) // 函数或对象方法的调用。不是返回false！！！
```JavaScript
const firstName = (message
  && message.body
  && message.body.user
  && message.body.user.firstName) || 'default';
// 等同于
const firstName = message?.body?.user?.firstName || 'default';
```

注意点：
（1）短路机制：`a?.[++x]`，如果a是undefined或null，那么x不会进行递增运算。链判断运算符一旦为真，右侧的表达式就不再求值。
（2）delete 运算符：`delete a?.b`，如果a是undefined或null，会直接返回undefined，而不会进行delete运算。
（3）括号的影响：`(a?.b).c`，.c总是会执行。一般来说，使用?.运算符的场合，不应该使用圆括号。
（4）报错场合：以下写法是禁止的，会报错。
```JavaScript
// 构造函数
new a?.()
new a?.b()

// 链判断运算符的右侧有模板字符串
a?.`{b}`
a?.b`{c}`

// 链判断运算符的左侧是 super
super?.()
super?.foo

// 链运算符用于赋值运算符左侧
a?.b = c
```
（5）右侧不得为十进制数值：为保证兼容以前的代码，允许`foo?.3:0`被解析成`foo ? .3 : 0`，因此规定如果?.后面紧跟一个十进制数字，那么?.不再被看成是一个完整的运算符，而会按照三元运算符进行处理。

### Null 判断运算符

对象属性的值是null或undefined，有时候需要为它们指定默认值。常见做法是通过||运算符指定默认值，但属性的值如果为空字符串或false或0，默认值也会生效。
[ES2020](https://github.com/tc39/proposal-nullish-coalescing) 引入 Null 判断运算符??，只有运算符左侧的值为null或undefined时，才会返回右侧的值。
??与&&和||的优先级孰高孰低？现在的规则是，如果多个逻辑运算符一起使用，必须用括号表明优先级，否则会报错。
```JavaScript
const animationDuration = response.settings?.animationDuration ?? 300;

 // 报错
lhs && middle ?? rhs
lhs ?? middle && rhs

// ok
(lhs && middle) ?? rhs;
lhs && (middle ?? rhs);
```

### 新增的方法

#### Object.is() 

相等运算符（==）和严格相等运算符（===）比较两个值时，前者会自动转换数据类型，后者的NaN不等于自身，以及+0等于-0。其实应该只要两个值是一样的，它们就应该相等。ES6 提出“Same-value equality”（同值相等）算法，Object.is就是用来比较两个值是否严格相等，与严格比较运算符（===）的行为基本一致，不同之处只有两个：一是+0不等于-0，二是NaN等于自身。
```JavaScript
Object.is('foo', 'foo')
// true
Object.is({}, {})
// false

+0 === -0 //true
NaN === NaN // false

Object.is(+0, -0) // false
Object.is(NaN, NaN) // true
```

#### Object.assign()

用于对象的合并，将源对象（source）的所有可枚举属性，复制到目标对象（target）。如果只有一个参数，会直接返回该参数。
如果该参数不是对象，则会先转成对象（undefined和null无法转成对象），然后返回。但如果非对象参数出现在源对象的位置（即非首参数），这时无法转成对象，就会跳过，即就不会报错。
```JavaScript
const obj = {a: 1};
Object.assign(obj) === obj // true

typeof Object.assign(2) // "object"
Object.assign(undefined) // 报错
Object.assign(null) // 报错

let obj = {a: 1};
Object.assign(obj, undefined) === obj // true
Object.assign(obj, null) === obj // true
```

其他类型的值（即数值、字符串和布尔值）不在首参数，也不会报错。但除了字符串会以数组形式，拷贝入目标对象，其他值都不会产生效果。因为只有字符串的包装对象，会产生可枚举属性。属性名为 Symbol 值的属性，也会被Object.assign拷贝。
```JavaScript
const v1 = 'abc';
const v2 = true;
const v3 = 10;
const obj = Object.assign({}, v1, v2, v3);
console.log(obj); // { "0": "a", "1": "b", "2": "c" }

Object.assign({ a: 'b' }, { [Symbol('c')]: 'd' })
// { a: 'b', Symbol(c): 'd' }
```

##### 注意点

###### （1）浅拷贝

Object.assign方法实行的是浅拷贝，就是说如果源对象某个属性的值是对象，那么目标对象拷贝得到的是这个对象的引用。
```JavaScript
const obj1 = {a: {b: 1}};
const obj2 = Object.assign({}, obj1);

obj1.a.b = 2;
obj2.a.b // 2
```

###### （2）同名属性的替换

对于同名属性的嵌套对象，Object.assign的处理方法是替换，而不是添加。**一些函数库提供Object.assign的定制版本（比如 Lodash 的_.defaultsDeep方法），可以得到深拷贝的合并。**
```JavaScript
const target = { a: { b: 'c', d: 'e' } }
const source = { a: { b: 'hello' } }
Object.assign(target, source)
// { a: { b: 'hello' } }
```

###### （3）数组的处理

Object.assign可以用来处理数组，但是会把数组视为对象。因为属性名是序号！
```JavaScript
Object.assign([1, 2, 3], [4, 5])
// [4, 5, 3]
```

###### （4）取值函数的处理

Object.assign只能进行值的复制，如果要复制的值是一个取值函数，那么将求值后再复制，而不会复制这个取值函数。
```JavaScript
const source = {
  get foo() { return 1 }
};
const target = {};

Object.assign(target, source)
// { foo: 1 }
```

##### 常见用途

（1）为对象添加属性
（2）为对象添加方法
（3）克隆对象：只能克隆原始对象自身的值，不能克隆它继承的值。
（4）合并多个对象
（5）为属性指定默认值：由于浅拷贝，DEFAULTS对象和options对象的所有属性的值，最好都是简单类型，不要指向另一个对象。否则，DEFAULTS对象的该属性很可能不起作用。
```JavaScript
const DEFAULTS = {
  logLevel: 0,
  outputFormat: 'html'
};

function processContent(options) {
  options = Object.assign({}, DEFAULTS, options);
  console.log(options);
  // ...
}
```

#### Object.getOwnPropertyDescriptors()

ES2017 引入了[Object.getOwnPropertyDescriptors()](http://es6.ruanyifeng.com/#docs/object-methods#Object-getOwnPropertyDescriptors)方法，返回指定对象所有自身属性（非继承属性）的描述对象。[MDN介绍](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor)引入目的：
* 为了解决Object.assign()无法正确拷贝get属性和set属性的问题。
* 配合Object.create()方法，将对象属性克隆到一个新对象。这属于浅拷贝。

#### __proto__属性，Object.setPrototypeOf()，Object.getPrototypeOf()

JavaScript 语言的对象继承是通过原型链实现的。ES6 提供了更多原型对象的操作方法。

__proto__属性（前后各两个下划线），用来读取或设置当前对象的prototype对象。目前，所有浏览器（包括 IE11）都部署了这个属性。无论从语义的角度，还是从兼容性的角度，都不要使用这个属性，而是Object.setPrototypeOf()（写操作）、Object.getPrototypeOf()（读操作）、Object.create()（生成操作）代替。实现上，__proto__调用的是Object.prototype.__proto__。

##### Object.setPrototypeOf()

用来设置一个对象的prototype对象，返回参数对象本身。

```JavaScript
// 格式
Object.setPrototypeOf(object, prototype)

// 用法
const o = Object.setPrototypeOf({}, null);

// 示例
let proto = {};
let obj = { x: 10 };
Object.setPrototypeOf(obj, proto);

proto.y = 20;
proto.z = 40;

obj.x // 10
obj.y // 20
obj.z // 40
```
上面代码将proto对象设为obj对象的原型，所以从obj对象可以读取proto对象的属性。如果第一个参数不是对象，会自动转为对象。但是由于返回的还是第一个参数，所以这个操作不会产生任何效果。由于undefined和null无法转为对象，所以如果第一个参数是undefined或null，就会报错。
```JavaScript
Object.setPrototypeOf(1, {}) === 1 // true
Object.setPrototypeOf('foo', {}) === 'foo' // true
Object.setPrototypeOf(true, {}) === true // true

Object.setPrototypeOf(undefined, {})
// TypeError: Object.setPrototypeOf called on null or undefined

Object.setPrototypeOf(null, {})
// TypeError: Object.setPrototypeOf called on null or undefined
```

##### Object.getPrototypeOf()

与Object.setPrototypeOf方法配套，用于读取一个对象的原型对象。如果参数不是对象，会被自动转为对象。如果参数是undefined或null，会报错。
```JavaScript
function Rectangle() {
  // ...
}
const rec = new Rectangle();
Object.getPrototypeOf(rec) === Rectangle.prototype
// true

Object.setPrototypeOf(rec, Object.prototype);
Object.getPrototypeOf(rec) === Rectangle.prototype
// false

Object.getPrototypeOf(1) === Number.prototype // true
Object.getPrototypeOf('foo') === String.prototype // true
Object.getPrototypeOf(true) === Boolean.prototype // true
```

#### Object.keys()，Object.values()，Object.entries()

ES2017 [引入](https://github.com/tc39/proposal-object-values-entries)了跟Object.keys配套的Object.values和Object.entries，作为遍历一个对象的补充手段，供for...of循环使用。

##### Object.keys()

返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键名。
```JavaScript
var obj = { foo: 'bar', baz: 42 };
Object.keys(obj)
// ["foo", "baz"]

let {keys, values, entries} = Object;
let obj = { a: 1, b: 2, c: 3 };

for (let key of keys(obj)) {
  console.log(key); // 'a', 'b', 'c'
}

for (let value of values(obj)) {
  console.log(value); // 1, 2, 3
}

for (let [key, value] of entries(obj)) {
  console.log([key, value]); // ['a', 1], ['b', 2], ['c', 3]
}
```

##### Object.values()

返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键值。返回数组的成员顺序，与本章的《属性的遍历》部分介绍的排列规则一致。
```JavaScript
const obj = { 100: 'a', 2: 'b', 7: 'c' };
Object.values(obj)
// ["b", "c", "a"]
```

Object.create方法的第二个参数添加的对象属性（属性p），如果不显式声明，默认是不可遍历的，因为p的属性描述对象的enumerable默认是false，Object.values不会返回这个属性。只要把enumerable改成true，Object.values就会返回属性p的值。
```JavaScript
const obj = Object.create({}, {p: {value: 42}});
Object.values(obj) // []

const obj = Object.create({}, {p:
  {
    value: 42,
    enumerable: true
  }
});
Object.values(obj) // [42]
```
会过滤属性名为 Symbol 值的属性。参数是一个字符串，会返回各个字符组成的一个数组。如果参数不是对象，Object.values会先将其转为对象。由于数值和布尔值的包装对象，都不会为实例添加非继承的属性。所以，Object.values会返回空数组。
```JavaScript
Object.values({ [Symbol()]: 123, foo: 'abc' });
// ['abc']

Object.values('foo')
// ['f', 'o', 'o']

Object.values(42) // []
Object.values(true) // []
```

##### Object.entries()

返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键值对数组。除了返回值不一样，该方法的行为与Object.values基本一致。Object.entries的基本用途是遍历对象的属性，另一个用处是将对象转为真正的Map结构。
```JavaScript
Object.entries({ [Symbol()]: 123, foo: 'abc' });
// [ [ 'foo', 'abc' ] ]

const obj = { foo: 'bar', baz: 42 };
const map = new Map(Object.entries(obj));
map // Map { foo: "bar", baz: 42 }
```

#### Object.fromEntries()

Object.fromEntries()方法是Object.entries()的逆操作，用于将一个键值对数组转为对象。该方法的主要目的，是将键值对的数据结构还原为对象，因此特别适合将 Map 结构转为对象。配合[URLSearchParams](https://developer.mozilla.org/zh-CN/docs/Web/API/URLSearchParams/URLSearchParams)对象，将查询字符串转为对象。
```JavaScript
Object.fromEntries([
  ['foo', 'bar'],
  ['baz', 42]
])
// { foo: "bar", baz: 42 }

// 例一
const entries = new Map([
  ['foo', 'bar'],
  ['baz', 42]
]);

Object.fromEntries(entries)
// { foo: "bar", baz: 42 }

// 例二
const map = new Map().set('foo', true).set('bar', false);
Object.fromEntries(map)
// { foo: true, bar: false }

Object.fromEntries(new URLSearchParams('foo=bar&baz=qux'))
// { foo: "bar", baz: "qux" }
```
[JavaScript URLSearchParams](https://blog.csdn.net/chy555chy/article/details/84879101)