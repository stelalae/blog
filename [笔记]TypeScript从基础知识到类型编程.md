# [笔记]TypeScript从基础知识到类型编程

[掘金小册地址](https://juejin.im/book/5da08714518825520e6bb810)，购于2019-11-11，开始学习于2019-12-10

法宝：

* 在TS项目中的`.d.ts`声明文件里，不完全支持js的灵活写法，那就将后缀改为.ts，文件内容ts的声明保持不变。即，将在各个业务代码中的ts声明单独提到专门的ts文件，等效于从`tsconfig.json`中配置的.d.ts，但支持js的灵活管理。

### 到底为什么要学习 TypeScript？

反对声音：
- 静态语言会丧失 JavaScript 的灵活性
- 静态类型不是银弹，大型项目依然可以用 JavaScript 编写
- TypeScript 必定赴 coffeescript 后尘，会被标准取代

优势：
- 规避大量低级错误，避免时间浪费，省时
- 减少多人协作项目的成本，大型项目友好，省力
- 良好代码提示，不用反复文件跳转或者翻文档，省心

虽说有 eslint，但在多人协作时，静态检查 + eslint，会更能产生优良代码。如果是旧项目，可以逐步由AnyScript到TypeScript。

### 开始使用 TypeScript

```bash
yarn global add typescript  // 全局安装
mkdir demo && cd demo
touch index.ts 
yarn init   // 初始化项目环境
tsc --init  // 初始化 ts，文件目录下多了一个tsconfig.json
```

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "es5",                            // 指定 ECMAScript 目标版本: 'ES5'
    "module": "commonjs",                       // 指定使用模块: 'commonjs', 'amd', 'system', 'umd' or 'es2015'
    "moduleResolution": "node",                 // 选择模块解析策略
    "experimentalDecorators": true,             // 启用实验性的ES装饰器
    "allowSyntheticDefaultImports": true,       // 允许从没有设置默认导出的模块中默认导入。
    "sourceMap": true,                          // 把 ts 文件编译成 js 文件的时候，同时生成对应的 map 文件
    "strict": true,                             // 启用所有严格类型检查选项
    "noImplicitAny": true,                      // 在表达式和声明上有隐含的 any类型时报错
    "alwaysStrict": true,                       // 以严格模式检查模块，并在每个文件里加入 'use strict'
    "declaration": true,                        // 生成相应的.d.ts文件
    "removeComments": true,                     // 删除编译后的所有的注释
    "noImplicitReturns": true,                  // 不是函数的所有返回路径都有返回值时报错
    "importHelpers": true,                      // 从 tslib 导入辅助工具函数
    "lib": ["es6", "dom"],                      // 指定要包含在编译中的库文件
    "typeRoots": ["node_modules/@types"],
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": [                                  // 需要编译的ts文件一个*表示文件匹配**表示忽略文件的深度问题
    "./src/**/*.ts"
  ],
  "exclude": [
    "node_modules",
    "dist",
    "**/*.test.ts",
  ]
}

// package.json
"scripts": {
    "build": "tsc", // 编译
    "build:w": "tsc -w" // 监听文件，有变动即编译
  },
```

### Typescript 的原始类型

- 布尔类型：`boolean`
- 数字类型：`number`
- 字符串类型：`string`
- 空值：`void`
- Null 和 Undefined：`null 和 undefined`
- Symbol 类型：`symbol`
- BigInt 大数整数类型：`bigint`

```javascript
function warnUser(): void {
    alert("This is my warning message");
}

const a: void = undefined || null;  // 只有null和undefined可以赋给void
```

默认情况下 null 和 undefined 是所有类型的子类型，就是说你可以把 null 和 undefined 赋值给 number 类型的变量。
但是在正式项目中一般都是开启 --strictNullChecks 检测的，即 null 和 undefined 只能赋值给 void 和它们各自，可以规避非常多的问题。
```javascript
let u: undefined = undefined;
let n: null = null;
```

### Typescript 中其他常见类型

- any：不希望类型检查器对这些值进行检查而是直接让它们通过编译阶段的检查。
- unknown：是 TypeScript 3.0 引入了新类型,是 any 类型对应的安全类型。区别在于对unknown类型的值执行大多数操作之前，我们必须进行某种形式的检查。
- never：表示那些永不存在的值的类型，never 类型是任何类型的子类型，可赋值给任何类型。但没有类型（包含 any）是 never 的子类型或可以赋值给 never 类型（除了never本身之外）。 

```javascript
// 抛出异常的函数永远不会有返回值
function error(message: string): never {
    throw new Error(message);
}

// 空数组，而且永远是空的
const empty: never[] = []
```

- 数组
```javascript
// 泛型
const list: Array<number> = [1, 2, 3]

// 非泛型
const list: number[] = [1, 2, 3]
```

- 元组（Tuple）：元组中包含的元素，必须与声明的类型一致，而且不能多、不能少，顺序也必须匹配。元组继承于数组，但是比数组拥有更严格的类型检查。
```javascript
let x: [string, number];
x = ['hello', 10]; // OK
x = [10, 'hello']; // Error
x = ['hello', 10, false] // Error
x = ['hello'] // Error
```

- Object：表示非原始类型，也就是除 number，string，boolean，symbol，null 或 undefined 之外的类型。

### 深入理解枚举类型

- 数字枚举：默认类型，且默认从0开始，后面逐步递增。
- 字符串枚举：（无）。
- 异构枚举：混合有数字、字符串的枚举。
- 常量枚举：类似宏定义，被性能优化编译后，直接是"宏替换"了，只是可读性更好。

枚举的本质，可以看成一个JavaScript对象，而由于其特殊的构造，导致其拥有正反向同时映射的特性：`name <=> value`。所以`enumXX[name] = value ，enumXX[value] = name`。

#### 枚举的高阶用法

联合枚举类型，即用枚举类型去声明变量后，那么该变量的值只能是对应枚举中的一个。

枚举合并，当定义多个相同名称的枚举时，会自动合并内部所有成员。（注，name不能相同，value可以一样！）。。好像只能在单文件中进行合并？不同文件是不同模块了？所以用处也不大。

借助 namespace 命名空间，可以给枚举添加静态方法。（不常用，跳过。）

### 接口(interface)

可选属性：`?`。如果实参中多了一个与可选属性一样类型的属性，那么会引发属性检查。可采用下面方式解决：
```javascript

// 示例：
interface Config {
  width?: number;
}

function  CalculateAreas(config: Config): { area: number} {
  let square = 100;
  if (config.width) {
      square = config.width * config.width;
  }
  return {area: square};
}

let mySquare = CalculateAreas({ width: 5 });

// 方法一：使用类型断言
let mySquare = CalculateAreas({ width: 5 } as Config);

// 方法二：添加字符串索引签名。（即：可索引类型）
interface Config {
   width?: number;
   [propName: string]: any;
}

// 方法三：将字面量赋值给另外一个变量。（慎用！！！）
let options = { width: 5 };
let mySquare = CalculateAreas(options);
```

只读属性：`readonly`。

函数类型：
```javascript
// 内部定义
interface User {
    name: string
    age?: number
    readonly isMale: boolean
    say: (words: string) => string
}

// 外部定义
interface Say {
    (words: string) : string
}

interface User {
    name: string
    age?: number
    readonly isMale: boolean
    say: Say
}

interface Other {
    address: string
}
```

继承接口，interface可同时继承一个或多个，但class仅能同时继承一个。
```javascript
interface VIPUser extends User, SupperUser, Other {
    broadcast: () => void
}
```

如果要继承多个，且要使用class的特性，就要利用对象合并，同时使用interface和class。
```javascript
// 不使用class特性（见上一个interface）

// 使用class特性，下面interface和class的VIPUser会进行合并
interface VIPUser extends User, SupperUser {
    broadcast: () => void
}
class VIPUser extends Other {
    other: string = ''
    // 其他class特性
}

// 或将extends全部放在interface
interface VIPUser extends User, SupperUser, Other {}
class VIPUser {
    broadcast: () => void
    other: string = ''
    // 其他class特性
}
```

### 类(Class)

abstract 关键字是用于定义抽象类，和在抽象类内部定义抽象方法。
```javascript
abstract class Animal {
    abstract makeSound(): void;
    move(): void {
        console.log('roaming the earch...');
    }
}
```
TypeScript 中有三类访问限定符，分别是: public、private、protected。

**相比  interface 作为类型声明时，class 具有类的特性，所以实际应用中要多使用 class。**
```javascript

// props的类型
export default class Props {
  public children: Array<React.ReactElement<any>> | React.ReactElement<any> | never[] = []
  public speed: number = 500
}

// 类型当着默认值使用
public static defaultProps = new Props()
```

### 函数(Function)

显式定义函数类型：
```javascript
const add: (a: number, b: number) => number = (a: number, b: nuber) => a + b;

// 等效于 
interface IAdd {
    (a: number, b: number) : number
}
const add: IAdd = (a: number, b: nuber) => a + b;
```

可选参数、默认参数、剩余参数（rest是个数组）。

函数重，根据传入不同的参数而返回不同类型的数据。[TS 函数重载](https://www.jianshu.com/p/b11e24dec350)
```javascript
// 上边是声明
function add (arg1: string, arg2: string): string
function add (arg1: number, arg2: number): number
// 因为我们在下边有具体函数的实现，所以这里并不需要添加 declare 关键字

// 下边是实现
function add (arg1: string | number, arg2: string | number) {
  // 在实现上我们要注意严格判断两个参数的类型是否相等，而不能简单的写一个 arg1 + arg2
  if (typeof arg1 === 'string' && typeof arg2 === 'string') {
    return arg1 + arg2
  } else if (typeof arg1 === 'number' && typeof arg2 === 'number') {
    return arg1 + arg2
  }
}
```

### 泛型（generic）的妙用


```javascript
// 泛型变量
function returnItem<T>(para: T): T {
    return para
}
const a = returnItem('a');
const b = returnItem(1);
a.length, b.toFixed(2);  // 1 '1.00'

// 多个类型参数
function swap<T, U>(tuple: [T, U]): [U, T] {
    return [tuple[1], tuple[0]];
}
swap([7, 'seven']); // ['seven', 7]

// 泛型变量
function getArrayLength<T>(arg: Array<T>) {
  console.log(arg.length); // ok
  return arg;
}
```

泛型类
```javascript
class Stack<T>
{
    private arr: T[] = []

    public push(item: T) {
        this.arr.push(item)
    }

    public pop() {
        this.arr.pop()
    }
}
```

泛型约束 - 类
```javascript
type Params = number | string;

class Stack<T extends Params> {}

const stack1 = new Stack<number>();     // ok
const stack2 = new Stack<boolean>();    // error
```

泛型约束 - 索引类型。用索引类型 keyof T 把传入的对象的属性类型取出生成一个联合类型，这里的泛型 U 被约束在这个联合类型中。如一个函数接受两个参数，一个参数为对象，另一个参数为对象上的属性，我们通过这两个参数返回这个属性的值：
```javascript
function getValue<T extends object, U extends keyof T>(obj: T, key: U) {
  return obj[key]
}

const a = { name: 'xiaomuzhu', id: 1 }
getValue(a, 'id')  // ok。key 的类型被约束为一个联合类型 name | id
getValue(a, 'xxx') // error
```

多重类型的泛型约束，即要先定义多重类型为一个类型。
```javascript
interface ChildInterface extends FirstInterface, SecondInterface {}
class Demo<T extends ChildInterface> {}
```

泛型 - new。通过制定 new() 返回的类型，以实现泛型的构造函数。
```javascript
function factory<T>(type: {new(): T}): T {
  return new type() // ok
}

class BeeKeeper {
  hasMask: boolean = false;
}

factory(BeeKeeper) // ok
factory(string) // error。难道不接受原始类型？
```

### 类型守卫

类型守卫就是缩小类型的范围，使用instanceof、in、字面量类型守卫等。
```javascript
class Person {
  name = 'xiaomuzhu';
  age = 20;
}
class Animal {
  name = 'petty';
  color = 'pink';
}

function getSometing(arg: Person | Animal) {
	if (arg instanceof Animal) {
		console.log(arg.color); // ok
		console.log(arg.age); // Error
	}
	if (arg instanceof Person) {
		console.log(arg.age); // Error
		console.log(arg.color); // ok
	}
}
function getSometing(arg: Person | Animal) {
  if ('age' in arg) {
    console.log(arg.color); // error
    console.log(arg.age); // ok
  }
  if ('color' in arg) {
    console.log(arg.age); // error
    console.log(arg.color); // ok
  }
}

getSometing(new Person());  // 20
getSometing(new Animal());  // pink

type Foo = {
  kind: 'foo'; // 字面量类型
  foo: number;
};
type Bar = {
  kind: 'bar'; // 字面量类型
  bar: number;
};
function doStuff(arg: Foo | Bar) {
  if (arg.kind === 'foo') {
    console.log(arg.foo); // ok
    console.log(arg.bar); // Error
  } else {
    console.log(arg.foo); // Error
    console.log(arg.bar); // ok
  }
}
```

### 高级类型

交叉类型，将多个类型合并为一个类型。 这让我们可以把现有的多种类型叠加到一起成为一种类型，它包含了所需的所有类型的特性。
```javascript
const x: Aa & Bb = {/* Aa和Bb的元素 */}
```

联合类型，希望属性为多种类型之一，如字符串或者数组。
```javascript
function xxx(command: string[] | string) {}
```

类型别名，给一个类型起个新名字，类型别名有时和接口很像，但是可以作用于原始值、联合类型、元组以及其它任何你需要手写的类型。
```javascript
type some = boolean | string    // 别名是联合类型
type Container<T> = { value: T }    // 别名是泛型
```
interface 只能用于定义对象类型，而 type 的声明方式除了对象之外还可以定义交叉、联合、原始类型等，类型声明的方式适用范围显然更加广泛。但是interface也有其特定的用处：
- interface 方式可以实现接口的 extends 和 implements
- interface 可以实现接口合并声明

```javascript
type Alias = { num: number }
interface Interface {
    num: number;
}
declare function aliased(arg: Alias): Alias;
declare function interfaced(arg: Interface): Interface;
```
interface创建了一个新的名字，可以在其它任何地方使用，type并不创建新名字，比如，错误信息就不会使用别名。

字面量类型（Literal Type）主要分为 真值字面量类型（boolean literal types）、数字字面量类型（numeric literal types）、枚举字面量类型（enum literal types）、大整数字面量类型（bigInt literal types）和字符串字面量类型（string literal types）。
```javascript
const a: 2333 = 2333 // ok
const ab : 0b10 = 2 // ok
const ao : 0o114 = 0b1001100 // ok
const ax : 0x514 = 0x514 // ok
const b : 0x1919n = 6425n // ok
const c : 'xiaomuzhu' = 'xiaomuzhu' // ok
const d : false = false // ok

const g: 'github' = 'pronhub' // 不能将类型“"pronhub"”分配给类型“"github"”
```
当字面量类型与联合类型结合的时候,用处就显现出来了,它可以模拟一个类似于枚举的效果。
```javascript
type Direction = 'North' | 'East' | 'South' | 'West';

function move(distance: number, direction: Direction) {
    // ...
}
```

类型字面量，跟 interface 也有点相似，在一定程度上类型字面量可以代替接口。
```javascript
type Foo = {
  baz: [
    number,
    'xiaomuzhu'
  ];
  toString(): string;
  readonly [Symbol.iterator]: 'github';
  0x1: 'foo';
  "bar": 12n;
};
```

可辨识联合类型，即利用类型守卫等方式，判断实体到底属于联合类型中的哪一种类型，这就要求每种类型需要唯一一个属性（或通俗性，但字面量类型不一样，参考上面的类型守卫示例）。

### 装饰器（看不太懂，后面专项学习）

在某些场景需要在不改变原有类和类属性的基础上扩展些功能，这也是装饰器出现的原因。

- 类装饰器
- 属性装饰器
- 方法装饰器
- 访问符装饰器
- 参数装饰器
- 装饰器工厂
- 装饰器顺序

[JS 装饰器，一篇就够](https://segmentfault.com/a/1190000014495089)