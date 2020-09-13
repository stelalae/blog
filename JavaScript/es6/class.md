# Class

[ECMAScript 6 入门 - Class 的基本语法](http://es6.ruanyifeng.com/#docs/class)
[ECMAScript 6 入门 - Class 的继承](http://es6.ruanyifeng.com/#docs/class-extends)

[toc]

### 简介

#### 类的由来

JavaScript 语言中，生成实例对象的传统方法是通过构造函数。基本上，ES6 的class可以看作只是一个语法糖，它的绝大部分功能，ES5 都可以做到，新的class写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。
```JavaScript
// ES5写法
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function () {
  return '(' + this.x + ', ' + this.y + ')';
};

var p = new Point(1, 2);

// ES6写法
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}
```
类的数据类型就是函数，类本身就指向构造函数。在类的实例上面调用方法，其实就是调用原型上的方法。ES6 的类，完全可以看作构造函数的另一种写法。**类必须使用new调用，否则会报错。这是它跟普通构造函数的一个主要区别，后者不用new也可以执行。**
```JavaScript
class Point {
  // ...
}
typeof Point // "function"
Point === Point.prototype.constructor // true

class B {}
let b = new B();
b.constructor === B.prototype.constructor // true
```

由于类的方法都定义在prototype对象上面，所以类的新方法可以添加在prototype对象上面。Object.assign方法可以很方便地一次向类添加多个方法。
```JavaScript
class Point {
  constructor(){
    // ...
  }
}

Object.assign(Point.prototype, {
  toString(){},
  toValue(){}
});

Point.prototype.constructor === Point // true
```

prototype对象的constructor属性，直接指向“类”的本身，这与 ES5 的行为是一致的。ES6的类的内部所有定义的方法，都是不可枚举的（non-enumerable），但是ES5的写法是可枚举的。
```JavaScript
var Point = function (x, y) {
  // ...
};

Point.prototype.toString = function() {
  // ...
};

Object.keys(Point.prototype)
// ["toString"]
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString"]
```

constructor方法是类的默认方法，通过new命令生成对象实例时，自动调用该方法。一个类必须有constructor方法，如果没有显式定义，一个空的constructor方法会被默认添加。constructor方法默认返回实例对象（即this），完全可以指定返回另外一个对象。
```JavaScript
class Foo {
  constructor() {
    return Object.create(null);
  }
}

new Foo() instanceof Foo    // false
```

与 ES5 一样，实例的属性除非显式定义在其本身（即定义在this对象上），否则都是定义在原型上（即定义在class上）。
```JavaScript
//定义类
class Point {

  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }

}

var point = new Point(2, 3);

point.toString() // (2, 3)

point.hasOwnProperty('x') // true
point.hasOwnProperty('y') // true
point.hasOwnProperty('toString') // false
point.__proto__.hasOwnProperty('toString') // true
```

与 ES5 一样，类的所有实例共享一个原型对象。这也意味着，可以通过实例的__proto__属性为“类”添加方法。但必须相当谨慎，不推荐使用，因为这会改变“类”的原始定义，影响到所有实例。
```JavaScript
var p1 = new Point(2,3);
var p2 = new Point(3,2);

p1.__proto__ === p2.__proto__   //true
```
> \_\_proto__ 并不是语言本身的特性，这是各大厂商具体实现时添加的私有属性，虽然目前很多现代浏览器的 JS 引擎中都提供了这个私有属性，但依旧不建议在生产中使用该属性，避免对环境产生依赖。生产环境中，我们可以使用 Object.getPrototypeOf 方法来获取实例对象的原型，然后再来为原型添加方法/属性。

与 ES5 一样，在“类”的内部可以使用get和set关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为。

类的属性名，可以采用表达式。——参考[Symbol章节](http://es6.ruanyifeng.com/#docs/symbol)
```JavaScript
let methodName = 'getArea';

class Square {
  constructor(length) {
    // ...
  }

  [methodName]() {
    // ...
  }
}
```

与函数一样，类也可以使用表达式的形式定义。表达式左侧的变量是外部类名，右侧的变量只能内部使用，也可省略。同理，也可以写成一个立即执行的类实例。
```JavaScript
const MyClass = class Me {
  getClassName() {
    return Me.name;
  }
};

let inst = new MyClass();
inst.getClassName() // Me
Me.name // ReferenceError: Me is not defined

let person = new class {
  constructor(name) {
    this.name = name;
  }

  sayName() {
    console.log(this.name);
  }
}('张三');

person.sayName(); // "张三"
```

**注意点**
（1）严格模式：类和模块的内部，默认就是严格模式，而且只有严格模式可用。考虑到未来所有的代码，其实都是运行在模块之中，所以 ES6 实际上把整个语言升级到了严格模式。

（2）不存在提升：因为class可以继承，所以class不存在变量提升。

（3）name 属性：紧跟在class关键字后面的类名。

（4）**Generator 方法**：如果某个方法之前加上星号（\*），就表示该方法是一个 Generator 函数。
```JavaScript
class Foo {
  constructor(...args) {
    this.args = args;
  }
  * [Symbol.iterator]() {
    for (let arg of this.args) {
      yield arg;
    }
  }
}

for (let x of new Foo('hello', 'world')) {
  console.log(x);
}
// hello
// world
```
上面代码中，Foo类的Symbol.iterator方法前有一个星号，表示该方法是一个 Generator 函数。Symbol.iterator方法返回一个Foo类的默认遍历器，for...of循环会自动调用这个遍历器。

（5）**this 的指向**：类的方法内部如果含有this，它默认指向类的实例。但是，必须非常小心，一旦单独使用该方法，很可能报错。
```JavaScript
class Logger {
  printName(name = 'there') {
    this.print(`Hello ${name}`);
  }

  print(text) {
    console.log(text);
  }
}

const logger = new Logger();
const { printName } = logger;
printName(); // TypeError: Cannot read property 'print' of undefined
```
上面代码中，printName方法中的this，默认指向Logger类的实例。但是，如果将这个方法提取出来单独使用，this会指向该方法运行时所在的环境（由于 class 内部是严格模式，所以 this 实际指向的是undefined），从而导致找不到print方法而报错。

一个比较简单的解决方法是，在构造方法中绑定this，这样就不会找不到print方法了。
```JavaScript
class Logger {
  constructor() {
    this.printName = this.printName.bind(this);
  }

  // ...
}
```

另一种解决方法是使用箭头函数。箭头函数内部的this总是指向定义时所在的对象。箭头函数所在的运行环境，肯定是实例对象，所以this会总是指向实例对象。
```JavaScript
class Obj {
  constructor() {
    this.getThis = () => this;
  }
}

const myObj = new Obj();
myObj.getThis() === myObj // true
```

还有一种解决方法是使用Proxy，获取方法的时候，自动绑定this。（跳过）


#### 实例属性及新写法

实例属性除了定义在constructor()方法里面的this上面，也可以定义在类的最顶层。属性定义在类的头部，看上去比较整齐，一眼就能看出这个类有哪些实例属性。
```JavaScript
class IncreasingCounter {
  _count = 0;
  //等效于
  constructor() {
    this._count = 0;
  }
 
  get value() {
    console.log('Getting the current value!');
    return this._count;
  }
  increment() {
    this._count++;
  }
}
```


### 静态方法和静态属性

类相当于实例的原型，所有在类中定义的方法，都会被实例继承。如果在一个方法前，加上static关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称为“静态方法”。**如果静态方法包含this关键字，这个this指的是类，而不是实例。**
```JavaScript
class Foo {
  static bar() {
    this.baz();
  }
  static baz() {
    console.log('hello');
  }
  baz() {
    console.log('world');
  }
}

Foo.bar() // hello
```
父类的静态方法，可以被子类继承。静态方法也是可以从super对象上调用的。


静态属性指的是 Class 本身的属性，即Class.propName，而不是定义在实例对象（this）上的属性。ES6 明确规定，Class 内部只有静态方法，没有静态属性。
```JavaScript
class Foo {
}

Foo.prop = 1;
Foo.prop // 1
```

现在有一个[提案](https://github.com/tc39/proposal-class-fields)提供了类的静态属性，写法是在实例属性的前面，加上static关键字。新写法是显式声明（declarative），而不是赋值处理，语义更好。
```JavaScript
class MyClass {
  static myStaticProp = 42;

  constructor() {
    console.log(MyClass.myStaticProp); // 42
  }
}
```


### 私有方法和私有属性

私有方法和私有属性，是只能在类的内部访问的方法和属性，外部不能访问。这是常见需求，有利于代码的封装。

#### 现有的解决方案

一种做法是在命名上加以区别。在方法前面加下划线（如`_bar`），表示这是一个只限于内部使用的私有方法。但在类的外部，还是可以调用到这个方法。

另一种方法就是索性将私有方法移出模块，因为模块内部的所有方法都是对外可见的。公开方法内部调用了被移除class的方法，即被移除的方法看做是私有方法。
```JavaScript
class Widget {
  foo (baz) {
    bar.call(this, baz);
  }

  // ...
}

function bar(baz) {
  return this.snaf = baz;
}
```

还有一种方法是利用Symbol值的唯一性，将私有方法的名字命名为一个Symbol值。但是也不是绝对不行，Reflect.ownKeys()依然可以拿到它们。——我觉得比第二个方法好，毕竟this等信息是共享的。
```JavaScript
const bar = Symbol('bar');
const snaf = Symbol('snaf');

export default class myClass{

  // 公有方法
  foo(baz) {
    this[bar](baz);
  }

  // 私有方法
  [bar](baz) {
    return this[snaf] = baz;
  }

  // ...
};
```

#### 私有属性的提案

目前，有一个[提案](https://github.com/tc39/proposal-private-methods)，为class加了私有属性。方法是在属性名之前，使用#表示，不仅可以写私有属性，还可以用来写私有方法。由于井号#是属性名的一部分，使用时必须带有#一起使用，所以#x和x是两个不同的属性。
```JavaScript
class Point {
  #x;

  constructor(x = 0) {
    this.#x = +x;
  }

  get x() {
    return this.#x;
  }

  set x(value) {
    this.#x = +value;
  }
  
  #sum(a, b) {
    return a + b;
  }
}

const counter = new Point();
counter.#x // 报错
counter.#x = 42 // 报错
```

之所以要引入一个新的前缀#表示私有属性，而没有采用private关键字，是因为 JavaScript 是一门动态语言，没有类型声明，使用独立的符号似乎是唯一的比较方便可靠的方法，能够准确地区分一种属性是否为私有属性。另外，Ruby 语言使用@表示私有属性，ES6 没有用这个符号而使用#，是因为@已经被留给了 Decorator。

私有属性x也可以设置 getter 和 setter 方法，读写都通过get #x()和set #x()来完成。私有属性不限于从this引用，只要是在类的内部，实例也可以引用私有属性。
```JavaScript
class Counter {
  #xValue = 0;
  #privateValue = 42;

  get #x() { return #xValue; }
  set #x(value) {
    this.#xValue = value;
  }
  
  static getPrivateValue(foo) {
    return foo.#privateValue;
  }
}

Foo.getPrivateValue(new Foo()); // 42
```

私有属性和私有方法前面，也可以加上static关键字，表示这是一个静态的私有属性或私有方法。

### new.target 属性

new是从构造函数生成实例对象的命令。ES6 为new命令引入了一个new.target属性，该属性一般用在构造函数之中，返回new命令作用于的那个构造函数。如果构造函数不是通过new命令或Reflect.construct()调用的，new.target会返回undefined，因此这个属性可以用来确定构造函数是怎么调用的。**利用这个特点，可以写出不能独立使用、必须继承后才能使用的类。**

Class 内部调用new.target，返回当前 Class。子类继承父类时，new.target会返回子类。

在函数外部，使用new.target会报错。

### 继承

Class 可以通过extends关键字实现继承，这比 ES5 的通过修改原型链实现继承，要清晰和方便很多。

**子类必须在constructor方法中调用super方法，否则新建实例时会报错。** 这是因为子类自己的this对象，必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法，然后再对其进行加工，加上子类自己的实例属性和方法。如果不调用super方法，子类就得不到this对象，即只有调用super之后，才可以使用this关键字，否则会报错。——super必须在constructor方法中的第一行
```JavaScript
class Point { /* ... */ }

class ColorPoint extends Point {
  constructor() {
  }
}

let cp = new ColorPoint(); // ReferenceError
```

ES5 的继承，实质是先创造子类的实例对象this，然后再将父类的方法添加到this上面（Parent.apply(this)）。ES6 的继承机制完全不同，实质是先将父类实例对象的属性和方法，加到this上面（所以必须先调用super方法），然后再用子类的构造函数修改this。

如果子类没有定义constructor方法，这个方法会被默认添加，代码如下。也就是说，不管有没有显式定义，任何一个子类都有constructor方法。
```JavaScript
class ColorPoint extends Point {
}

// 等同于
class ColorPoint extends Point {
  constructor(...args) {
    super(...args);
  }
}

let cp = new ColorPoint(25, 8, 'green');
cp instanceof ColorPoint // true
cp instanceof Point // true
```

实例同时是父子两个类的实例，Object.getPrototypeOf方法可以用来从子类上获取父类，来判断一个类是否继承了另一个类。
```JavaScript
Object.getPrototypeOf(ColorPoint) === Point // true
```

[ES5和ES6中的继承](http://keenwon.com/1524.html)

ES5中的继承

![](http://img.keenwon.com/2016/03/20160314212504_39150.png)

```JavaScript
function Super() {}
 
function Sub() {}
Sub.prototype = new Super();
Sub.prototype.constructor = Sub;
 
var sub = new Sub();
 
Sub.prototype.constructor === Sub; // ② true
sub.constructor === Sub; // ④ true
sub.__proto__ === Sub.prototype; // ⑤ true
Sub.prototype.__proto__ == Super.prototype; // ⑦ true
```

ES6中的继承

![](http://img.keenwon.com/2016/01/20160116201909_44777.png)

```JavaScript
class Super {}
 
class Sub extends Super {}
 
var sub = new Sub();
 
Sub.prototype.constructor === Sub; // ② true
sub.constructor === Sub; // ④ true
sub.__proto__ === Sub.prototype; // ⑤ true
Sub.__proto__ === Super; // ⑥ true
Sub.prototype.__proto__ === Super.prototype; // ⑦ true
```

ES6和ES5的继承是一模一样的，只是多了class 和extends ，ES6的子类和父类，子类原型和父类原型，通过__proto__ 连接。

### super 关键字

super这个关键字，既可以当作函数使用，也可以当作对象使用。在这两种情况下，它的用法完全不同。

**第一种情况，super作为函数调用时，代表父类的构造函数。** 但是返回的是子类B的实例，即super内部的this指的是B的实例，因此super()在这里相当于A.prototype.constructor.call(this)。作为函数时，super()只能用在子类的构造函数之中，用在其他地方就会报错。
```JavaScript
class A {
  constructor() {
    console.log(new.target.name);
  }
}
class B extends A {
  constructor() {
    super();
  }
}
new A() // A
new B() // B
```

**第二种情况，super作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。** 由于super指向父类的原型对象，所以定义在父类实例上的方法或属性，是无法通过super调用的。
```JavaScript
class A {
  constructor() {
    this.p = 2;
  }
}

class B extends A {
  get m() {
    return super.p;
  }
}

let b = new B();
b.m // undefined
b.p // 2
```

如果属性定义在父类的原型对象上，super就可以取到。
```JavaScript
class A {}
A.prototype.p = 2;
```

**ES6 规定，在子类普通方法中通过super调用父类的方法时，方法内部的this指向当前的子类实例。** 所以如果通过super对某个属性赋值，赋值的属性会变成子类实例的属性。 
```JavaScript
class A {
  constructor() {
    this.x = 1;
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
    super.x = 3;
    console.log(super.x); // undefined
    console.log(this.x); // 3
  }
}

let b = new B();
```

如果super作为对象，用在静态方法之中，这时super将指向父类，而不是父类的原型对象。

在子类的静态方法中通过super调用父类的方法时，方法内部的this指向当前的子类，而不是子类的实例。
```JavaScript
class A {
  constructor() {
    this.x = 1;
  }
  static print() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  static m() {
    super.print();
  }
}

B.x = 3;
B.m() // 3
```
上面代码中，静态方法B.m里面的this指向的是B，而不是B的实例。

注意，使用super的时候，必须显式指定是作为函数、还是作为对象使用，否则会报错，即`console.log(super);`会报错。

最后，由于对象总是继承其他对象的，所以可以在任意一个对象中，使用super关键字。
```JavaScript
var obj = {
  toString() {
    return "MyObject: " + super.toString();
  }
};

obj.toString(); // MyObject: [object Object]
```

### 类的 prototype 属性和__proto__属性

大多数浏览器的 ES5 实现之中，每一个对象都有__proto__属性，指向对应的构造函数的prototype属性。Class 作为构造函数的语法糖，同时有prototype属性和__proto__属性，因此同时存在两条继承链。

（1）子类的__proto__属性，表示构造函数的继承，总是指向父类。

（2）子类prototype属性的__proto__属性，表示方法的继承，总是指向父类的prototype属性。
```JavaScript
class A {
}

class B extends A {
}

B.__proto__ === A // true
B.prototype.__proto__ === A.prototype // true
```

这两条继承链，可以这样理解：作为一个对象，子类（B）的原型（__proto__属性）是父类（A）；作为一个构造函数，子类（B）的原型对象（prototype属性）是父类的原型对象（prototype属性）的实例。

extends关键字后面可以跟多种类型的值，下面讨论两种情况。第一种，子类继承Object类。
```JavaScript
class A extends Object {
}

A.__proto__ === Object // true
A.prototype.__proto__ === Object.prototype // true
```
这种情况下，A其实就是构造函数Object的复制，A的实例就是Object的实例。

第二种情况，不存在任何继承。
```JavaScript
class A {
}

A.__proto__ === Function.prototype // true
A.prototype.__proto__ === Object.prototype // true
```
这种情况下，A作为一个基类（即不存在任何继承），就是一个普通函数，所以直接继承Function.prototype。但是，A调用后返回一个空对象（即Object实例），所以A.prototype.\_\_proto__指向构造函数（Object）的prototype属性。


#### 实例的\_\_proto__ 属性

子类实例的__proto__属性的__proto__属性，指向父类实例的__proto__属性。也就是说，子类的原型的原型，是父类的原型。
```JavaScript
var p1 = new Point(2, 3);
var p2 = new ColorPoint(2, 3, 'red');

p2.__proto__ === p1.__proto__ // false
p2.__proto__.__proto__ === p1.__proto__ // true
```

### 原生构造函数的继承

原生构造函数是指语言内置的构造函数，通常用来生成数据结构。ECMAScript 的原生构造函数大致有下面这些。
* Boolean()
* Number()
* String()
* Array()
* Date()
* Function()
* RegExp()
* Error()
* Object()

以前，这些原生构造函数是无法继承的。因为子类无法获得原生构造函数的内部属性，通过Array.apply()或者分配给原型对象等方式都不行。原生构造函数会忽略apply方法传入的this，也就是说，原生构造函数的this无法绑定，导致拿不到内部属性。

ES6 允许继承原生构造函数定义子类，因为 ES6 是先新建父类的实例对象this，然后再用子类的构造函数修饰this，使得父类的所有行为都可以继承。
```JavaScript
class ExtendableError extends Error {
  constructor(message) {
    super();
    this.message = message;
    this.stack = (new Error()).stack;
    this.name = this.constructor.name;
  }
}

class MyError extends ExtendableError {
  constructor(m) {
    super(m);
  }
}

var myerror = new MyError('ll');
myerror.message // "ll"
myerror instanceof Error // true
myerror.name // "MyError"
myerror.stack
```

注意，继承Object的子类，有一个[行为差异](http://stackoverflow.com/questions/36203614/super-does-not-pass-arguments-when-instantiating-a-class-extended-from-object)。继承了Object，但是无法通过super方法向父类Object传参。这是因为 ES6 改变了Object构造函数的行为，一旦发现Object方法不是通过new Object()这种形式调用，ES6 规定Object构造函数会忽略参数。
```JavaScript
class NewObj extends Object{
  constructor(){
    super(...arguments);
  }
}
var o = new NewObj({attr: true});
o.attr === true  // false
```

### Mixin 模式的实现

Mixin 指的是多个对象合成一个新的对象，新对象具有各个组成成员的接口。它的最简单是通过扩展运算符(...)来合成一个新对象。

通过此方式可以将多个对象合成为一个类。使用的时候，只要继承这个类即可，即实现多重继承？？？