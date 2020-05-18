# JavaScript的强语言之路—另类的JSON序列化与反序列化

> 2020年01月17日 更新
此篇文章代码Demo：https://github.com/stelalae/node_demo ，`yarn start:debug`看控制台输出效果。

JSON（JavaScript Object Notation）是一种轻量级，完全独立于语言的数据交换格式。目前被广泛应用在前后端的数据交互中。在JavaScript中的应用随处可见，灵活性、扩展性、可读性也是最强的！对应的`JSON.parse`与`JSON.stringify`就可以看做是对象的序列化和反序列化，将对象与字符串之间相互转换。

### 序列化与反序列化 

> 互联网的产生带来了机器间通讯的需求，而互联通讯的双方需要采用约定的协议，序列化和反序列化属于通讯协议的一部分。通讯协议往往采用分层模型，不同模型每层的功能定义以及颗粒度不同，例如：TCP/IP 协议是一个四层协议，而 OSI 模型却是七层协议模型。在 OSI 七层协议模型中展现层（Presentation Layer）的主要功能是把应用层的对象转换成一段连续的二进制串，或者反过来，把二进制串转换成应用层的对象 -- 这两个功能就是序列化和反序列化。一般而言，TCP/IP 协议的应用层对应与 OSI 七层协议模型的应用层，展示层和会话层，所以序列化协议属于 TCP/IP 协议应用层的一部分。本文对序列化协议的讲解主要基于 OSI 七层协议模型。

* 序列化： 将数据结构或对象转换成二进制串的过程。
* 反序列化：将在序列化过程中所生成的二进制串转换成数据结构或者对象的过程。

简单来说，序列化是将对象转换成字节流的过程，而反序列化的是将字节流恢复成对象的过程。两者的关系如下：
![168786827eb84b58](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/05/14/gPYOyT.jpg)

不同的计算机语言中，数据结构、对象以及二进制串的表示方式并不相同。如Java/JavaScript中使用的是对象（Object），来自类的实例化。而C是用struct去表示数据解构，或根据指针的偏移量在内存中读取数据。C++则是Java方式或C方式均可，因为C++比C强化了class的概念。

从计算机语言的发展历史来看，序列化协议从通用性、健壮性、可读性、扩展性、安全性、性能等方面，设计出下面几种常见的协议：

* 早期协议：COM、CORBA。
* XML&SOAP：XML（可扩展标记语言，Extensible Markup Language）本质上是一种描述语言，具有自我描述的属性。SOAP（Simple Object Access protocol） 是基于 XML 为序列化和反序列化协议的结构化消息传递协议，在前期互联网阶段被广泛应用，对应的解决方案叫做Web Service。
* JSON：源于JavaScript，赶上移动浪潮发展，现正在广泛使用在各种业务场景中。
* Thrift：是Facebook开源的轻量级RPC服务框架，在大数据量、分布式、跨语言、跨平台的服务端使用较多。
* Protobuf：来自Google，因改善了XML、JSON的诟病，空间开销小、高解析性能、语言支持度高等亮点，许多公司将其作为后端之间通信、对象持久化等方面的首选方案。在终端上，IM业务上也使用较多。
* Avro：属于 Apache Hadoop的子项目，提供两种序列化格式：JSON、Binary，即调试时用JSON，上线后用Binary。（个人觉得就是对JSON在传输时的改良版）

上述序列化协议的Benchmark请自行搜索与查询。在选型上也要根据具体业务场景，个人建议：JSON适合①有前端参与②小型项目， Protobuf适合①高性能②T级别数据。

### JSON的parse与stringify

首先来看看MDN的介绍：[JSON.stringify()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)、[JSON.parse](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse)。

- `JSON.stringify(value[, replacer [, space]])`：用来将一个 JavaScript 值（对象或者数组）转换为一个 JSON 字符串，如果指定了 replacer 是一个函数，则可以选择性地替换值，或者如果指定了 replacer 是一个数组，则可选择性地仅包含数组指定的属性。
- `JSON.parse(text[, reviver])`：用来解析JSON字符串，构造由字符串描述的JavaScript值或对象。提供可选的 reviver 函数用以在返回之前对所得到的对象执行变换(操作)。

你可能已注意到，JSON的stringify比parse多了一段[描述](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify#%E6%8F%8F%E8%BF%B0)：
> - 转换值如果有toJSON()方法，该方法定义什么值将被序列化。
> - 非数组对象的属性不能保证以特定的顺序出现在序列化后的字符串中。
> - 布尔值、数字、字符串的包装对象在序列化过程中会自动转换成对应的原始值。
> - undefined、任意的函数以及 symbol 值，在序列化过程中会被忽略（出现在非数组对象的属性值中时）或者被转换成 null（出现在数组中时）。函数、undefined被单独转换时，会返回undefined，如JSON.stringify(function(){}) or JSON.stringify(undefined).
> - 对包含循环引用的对象（对象之间相互引用，形成无限循环）执行此方法，会抛出错误。
> - 所有以 symbol 为属性键的属性都会被完全忽略掉，即便 replacer 参数中强制指定包含了它们。
> - Date日期调用了toJSON()将其转换为了string字符串（同Date.toISOString()），因此会被当做字符串处理。
> - NaN和Infinity格式的数值及null都会被当做null。
> - 其他类型的对象，包括Map/Set/weakMap/weakSet，仅会序列化可枚举的属性。

这段描述说明了，在JSON.stringify序列化时哪些数据会被保留、转换、忽略。所以我的前端项目里一定会在接口请求层里对undefined和null进行屏蔽：
```JavaScript
const body = JSON.stringify(params, (k, v) => {
    if (v !== null && v !== undefined) {
      return v;
    }
});
```

同样在axios请求响应后，对response data里的undefined和null进行屏蔽：
```JavaScript
import axios from 'axios';
import { ResponseData } from '../defines';

// axios.defaults.timeout = 10000;

const parseJSON = (response: any) => {
  // 先对Object进行序列化，再有条件的反序列化
  const dataString = JSON.stringify(response);
  const dataObj = JSON.parse(dataString, (k: any, v: any) => {
    if (v === null) {
      return undefined;
    }
    return v;
  });
  return dataObj;
};

const parseResponse = (response: any) => {
  if (response.status >= 200 && response.status < 300) {
    return response.data;
  }
  return {};
};

export const request = async (options: any): Promise<ResponseData> => {
  try {
    const resp = await axios(options);
    const data = await parseResponse(resp);
    return parseJSON(data);
  } catch (err) {
    return Promise.resolve({
      code: 999,
      msg: '网络超时',
    });
  }
};
```

### 完善解构赋值

上面的JSON.stringify、JSON.parse是多余和浪费性能的？？？下面我们用代码来说话。
```JavaScript
// types.ts
export class OrderBase {
  creatTime: string;
  payTime: string;
  // 其他省略
}

export class OrderDetail {
  orderNo: string;
  userNo: string;
  businessCode: string;
  imgList: string[];
  base: OrderBase;
}
```
鉴于目前ts越来越火，在中大型项目中使用越来越多，所以我这里也引入ts，且不会提示ts错误。

```JavaScript
// order.ts
function test() {
  const orderinfo: OrderDetail = JSON.parse('{}');
  const { imgList, base } = orderinfo;

  let imgurl = [];
  if (imgList && imgList.length > 0) {
    imgurl = imgList.map(item => (item += '?xxx'));
  }
  const { payTime } = base || {};

  let showpay = false;
  if (payTime && payTime.length > 0) {
    showpay = true;
  }
}
test();
```
在现代JavaScript项目中ES6写法已经广泛使用，如果你熟悉和习惯ES6写法，特别是[解构赋值](http://es6.ruanyifeng.com/#docs/destructuring)，刚才的代码一定会让你感到不舒服的！
```JavaScript
function test() {
  const orderinfo: OrderDetail = JSON.parse('{}');
  const { imgList = [], base: { payTime = '' } = {} } = orderinfo;
  const imgurl = imgList.map(item => (item += '?xxx'));
  const showpay = payTime !== '';
}
test();
```
没错，这才是标准的ES6写法，解构赋值的默认值、嵌套解构等经常使用时，必须在解析接口响应处等源头，利用反序列化时将值为null的属性从对象中移除掉。

 JavaScript 项目中最常见的十大错误。
![16ddd25080469b29](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/05/14/SonX90.jpg)

我觉得解构赋值和解构默认值的出现，至少让前端少了10%的代码量！！！ts的出现，加上es6、es2017、es2020等发布，正确使用新特性，让我们逐步从低级bug中解脱出来。

不管接口规范定义得再好，也不要相信接口数据会100%按照你的要求来。——*前后端日常扯皮之接口部分*

### 重构解构赋值

上面例子简单的说了解构赋值及默认值的好处，就是简化代码逻辑，少写bug。但你也可能意识到，如果很多地方都要用解构，是不是都要写一遍默认值？！好像是的。

所以理想情况是，只在一处设置对象的默认值后，其他地方如嵌套解构、JSON的序列化和反序列化等，就可以直接使用默认值了！下面开始记录改造过程。

TypeScript项目里肯定要是从类型定义开始下手，即要在对象声明时定义默认值。因ts有类型推导，所以有了默认值的部分属性会被显示移除类型声明。*注意，因添加了默认值，所以只能使用class，且声明文件不会被注解，不能用interface。*
```JavaScript
export class OrderBase {
  creatTime: string = '';
  payTime: string = '';
}

export class OrderDetail {
  orderNo: string = '';
  userNo = '';  // 根据默认值推导出userNo的类型为string
  imgList: string[] = [];
  base: OrderBase;
}
```

那现在测试效果，发现userNo居然是undefined，意思是类型声明里的默认值未生效。
```JavaScript
function test() {
  const orderinfo: OrderDetail = JSON.parse('{}');
  const { base: { payTime = '123' } = {}, userNo } = orderinfo;
  console.log(payTime, userNo); // 打印：123 undefined
}
test();
```

因为是ts项目，实际运行代码是编译之后的代码，而不是编写的代码。
```JavaScript
// tsconfig.json中"target": "es2017"时的效果
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
function test() {
    const orderinfo = JSON.parse('{}');
    const { base: { payTime = '123' } = {}, userNo } = orderinfo;
    console.log(payTime, userNo);
}
test();


// tsconfig.json中"target": "es5"时的效果
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
function test() {
    var orderinfo = JSON.parse('{}');
    var _a = orderinfo.base, _b = (_a === void 0 ? {} : _a).payTime, payTime = _b === void 0 ? '123' : _b, userNo = orderinfo.userNo;
    console.log(payTime, userNo);
}
test();
```

经过编译之后，test里的代码好像与声明文件没关系了，更别说使用定义的默认值。可以思考想一想这是为什么！
![fa9c36fbf520cf0e0e66ebf71844275d](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/05/14/HWjkQg.jpg)


原因是：
1. class当做类型去声明变量时，用法和interface一样效果，编译后对应代码会被注解。
2. 添加了类型声明的变量，数据类型并未发生变化，而不是变成类的一个实例。

**正确做法时，让变量由一个JSON对象变成一个类实例。被类实例化后，就可以使用类的一切特性和自定义函数。**

#### 属性回填

```JavaScript
const a = new OrderDetail();
console.log(a.userNo); // 打印：''
a.userNo = 'abc';
console.log(a.userNo); // 打印：'abc'
```

> 太阔怕了！

#### Object.assign

```JavaScript
const obj = {
  userNo: 'a1',
  base: { creatTime: '2020-01-13 18:32:58' },
};
const a = Object.assign(new OrderDetail(), obj);
console.log(a.userNo); // 打印：'a1'
a.userNo = 'abc';
console.log(a.userNo); // 打印：'abc'
console.log(a.base.creatTime); // 打印：'2020-01-13 18:32:58'
console.log(a.base.payTime); // 打印：undefined
```
相比属性回填，大家应该更能接受通过assign去实例化。**另外，assign只能一层深拷贝，所以不要想着原型链上的操作。**

#### immutable record

[Record](https://immutable-js.github.io/immutable-js/docs/#/Record)是一种仅记录JS对象中已注册过的成员的数据结构，且支持默认值。
> A record is similar to a JS object, but enforces a specific set of allowed string keys, and has default values.

```JavaScript
const { Record } = require('immutable')
const ABRecord = Record({ a: 1, b: 2 })
const myRecord = ABRecord({ b: 3,  x: 10 })     // ts中此处会提示错误

myRecord.size // 2
myRecord.get('a') // 1
myRecord.get('b') // 3
const myRecordWithoutB = myRecord.remove('b')   // remove后，b会被重置为2，而不是undefined
myRecordWithoutB.get('b') // 2
myRecordWithoutB.size // 2
myRecord.get('x') // undefined
```

**immutable record仅会记录初始化时已注册的key，且能永远保证你key-value存在。** 和Object.assign相比，Record的数据结构更强势，自带的方法会为你的数据管理设计提供大放异彩的可能。

```JavaScript
// 方式一：有类型声明
type PersonProps = {name: string, age: number};
const defaultValues: PersonProps = {name: 'Aristotle', age: 2400};
const PersonRecord = Record(defaultValues);
class Person extends PersonRecord<PersonProps> {
  getName(): string {
    return this.get('name')
  }

  setName(name: string): this {
    return this.set('name', name);
  }
}

// 方式二：无类型声明
class ABRecord extends Record({ a: 1, b: 2 }) {
  getAB() {
    return this.a + this.b;
  }
}

var myRecord = new ABRecord({b: 3})
myRecord.getAB() // 4
```

对比上面两种方式，区别在于是否是集成了 TypeScript。结合 Record 的文档，会发现 Record 其实是工厂模式的一种应用。先通过默认值方式注册到工厂，然后内部桥接了一些自定义函数。（本人未研究 Record 源码，若有不对请指出）

[Record](https://immutable-js.github.io/immutable-js/docs/#/Record)里也有专门说明 **Choosing Records vs plain JavaScript objects**：
* Runtime immutability：运行时不可变性，这符合immutable的一贯原则，数据的只读性。
* Value equality：值相等，即Record提供equals来判断两个对象是否严格相等。
* API methods：额外的API方法，如getIn、equals、hashCode、toJS-深层、toObject-浅层、toJSON-浅层等。
* Default values：默认值。为每个键提供默认值，即时被remove之后。
* Serialization：序列化。即提供toJS-深层、toObject-浅层、toJSON-浅层，和JSON对象之间的序列化和反序列化。

所以我在React项目中一直使用immutable，为数据管理和页面Render设计了很多优化方案。

### 数据对象的类实例化

前面提到的重构解构赋值，目的就是将JavaScript中的JSON Object变为Class Object，如标题所说的『另类』就是指**将原有的 JSON 字符串与JavaScript 值（对象或者数组）互转，变成 JSON 字符串或JavaScript 值（对象或者数组）与Class互转**。然后对class进行扩展，比如封装、继承、重载等，让JavaScript的强语言之路走得更远！

在MVC模型里，相比utils函数、String/Number扩展函数等方式，使用class管理更容易减少项目的内部耦合度，也更方便扩展和维护。比如性别：
```JavaScript
export enum SexType {
  none = 0,
  man = 1,
  woman = 2,
}

export class UserProfile {
  userNo: string = '';
  userName: string = '';
  sex: SexType = 0;

  static sexMap = {
    [SexType.none]: '保密',
    [SexType.man]: '男',
    [SexType.woman]: '女',
  };

  public get sexT(): string {
    return UserProfile.sexMap[this.sex] || '保密';
  }
  
  // 其他自定义函数
}

const myinfo = Object.assign(new UserProfile(), {
  userNo: '1',
  userName: 'test',
  sex: 1,
});

console.log(myinfo.sex, myinfo.sexT); // 打印：1   '男'
myinfo.userName = '1'; // ok - userName是可修改的
```

但是，如果引入immutable-record方式呢？

```JavaScript

// index.ts - 声明文件
import { Record } from 'immutable';

export enum SexType {
  none = 0,
  man = 1,
  woman = 2,
}

interface IRecordInfo {
  createTime: string;
  updateTime?: string;
}

const IRecordInfoDefault: IRecordInfo = {
  createTime: '',
  updateTime: '',
};

export class RecordInfo extends Record(IRecordInfoDefault) {} // 建议使用class，即时没有扩展属性
// export const RecordInfo = Record(IRecordInfoDefault); // 不建议变量方式，否则不能把 RecordInfo 当做类型去声明变量

interface IUserProfile extends IRecordInfo {
  userNo: string;
  userName: string;
  sex: SexType;
}

const UserProfileDefault: IUserProfile = {
  userNo: '',
  userName: '',
  sex: SexType.none,
  ...IRecordInfoDefault,
};

export class UserProfile extends Record(UserProfileDefault) {
  sexMap = {
    [SexType.none]: '保密',
    [SexType.man]: '男',
    [SexType.woman]: '女',
  };

  public get sexT(): string {
    return this.sexMap[this.sex] || '保密';
  }

  // 其他自定义函数
}

export interface ITsTest {
  id: string;
}

const ITsTestDefault: ITsTest = {
  id: '',
};

export const TsTest = Record(ITsTestDefault);

const myinfo = new UserProfile({ userNo: '1', userName: 'test', sex: 1 });
console.log(myinfo.sex, myinfo.sexT); // 打印：1   '男'

myinfo.userName = '1'; // error - userName是只读的，不可修改
const myinfonew = myinfo.set('userName', '1'); // ok - 返回新的 immutable 对象
```

你应该注意到，引入immutable record后，在类型声明上可能会增加代码量，但我觉得这是值得的！原因有下：
* 不影响声明类型，可读性强。
* 从源头严格控制数据模型，防止无关数据。
* 从 JSON 对象到 Class 对象，降低设计上的耦合度。

React是以数据驱动页面的模式，即MVC中通过改变M，自动去更新V。所以引入immutable record能提升M，减少V、C中的逻辑。对比自己日常的React代码，能看出哪些优化空间吗？！
```JavaScript

// index.ts - 业务文件
import React from 'react';

import { RecordInfo, UserProfile, TsTest } from '../types';

interface DivBaseProps {
  data: RecordInfo;
}

const DivBase = (props: DivBaseProps) => {
  const { data } = props;
  console.log(data, data.hashCode()); // 如前面声明类型时，建议用class方式
  return (
    <div>
      <p>{data.createTime}</p>
      <p>{data.updateTime}</p>
    </div>
  );
};

interface HomeState {
  detail: UserProfile;
  extra: RecordInfo;
}

export default class Home extends React.PureComponent<any, HomeState> {
  constructor(props: any) {
    super(props);
    this.state = {
      detail: new UserProfile(), // ok
      extra: new RecordInfo(), // ok
      // detail: new RecordInfo(), // error - 缺失数据
      // extra: new UserProfile(), // ok - 因 UserProfile 从 RecordInfo继承而来
      // extra: new TsTest(), // error - 缺失数据
    };
  }

  componentDidMount() {
    setTimeout(() => {
      this.setState({ detail: new UserProfile({ sex: 1 }) });
    }, 1000);
  }

  render() {
    const { detail } = this.state;
    return <DivBase data={new RecordInfo(detail)}></DivBase>; // ok
    // return <DivBase data={{ ...detail }}></DivBase>; // error
  }
}
```

### 总结

本文先介绍JavaScript自带的JSON序列化和反序列化的基本使用，配合ES6中解构赋值及默认值，优化了在对空值判断的处理方式。然后利用类实例化，对JSON对象进一步反序列化，同时利用class进行高内聚低耦合的代码管理。

因为TypeScript提供编译基础，现在JavaScript的编码方式逐步向强语言靠拢，当然Running time还是未改变，期待TypeScript可输出WebAssembly！

参考资料：  
1.[序列化和反序列化](https://www.infoq.cn/article/serialization-and-deserialization)  
2.[序列化与反序列化](https://juejin.im/post/5c47c8aae51d4551363ff2f9)  
3.[探索如何使用 JSON.stringify() 去序列化一个 Error](https://juejin.im/post/5d81ee1151882570315f56ad)  
4.[Record](https://immutable-js.github.io/immutable-js/docs/#/Record)  