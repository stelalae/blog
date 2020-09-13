# module

[ECMAScript 6 入门 - Module 的语法](http://es6.ruanyifeng.com/#docs/module)
[ECMAScript 6 入门 - Module 的加载实现](http://es6.ruanyifeng.com/#docs/module-loader)

[toc]

在 ES6 之前，社区制定了一些模块加载方案，最主要的有 CommonJS 和 AMD 两种。前者用于服务器，后者用于浏览器。ES6 在语言标准的层面上，实现了模块功能，而且实现得相当简单，完全可以取代 CommonJS 和 AMD 规范，成为浏览器和服务器通用的模块解决方案。ES6 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS 和 AMD 模块，都只能在运行时确定这些东西。比如，CommonJS 模块就是对象，输入时必须查找对象属性。
```JavaScript
// CommonJS模块
let { stat, exists, readFile } = require('fs');

// 等同于
let _fs = require('fs');
let stat = _fs.stat;
let exists = _fs.exists;
let readfile = _fs.readfile;
```
上面代码的实质是整体加载fs模块（即加载fs的所有方法），生成一个对象（_fs），然后再从这个对象上面读取 3 个方法。这种加载称为“运行时加载”，因为只有运行时才能得到这个对象，导致完全没办法在编译时做“静态优化”。

```JavaScript
// ES6模块
import { stat, exists, readFile } from 'fs';
```
ES6 模块不是对象，而是通过export命令显式指定输出的代码，再通过import命令输入。从fs模块加载 3 个方法，其他方法不加载。这种加载称为“编译时加载”或者静态加载，即 ES6 可以在编译时就完成模块加载，效率要比 CommonJS 模块的加载方式高。当然，这也导致了没法引用 ES6 模块本身，因为它不是对象。

### export

[export | MDN - Mozilla](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/export)，语法：
```JavaScript
// 导出单个特性
export let name1, name2, …, nameN; // also var, const
export let name1 = …, name2 = …, …, nameN; // also var, const
export function FunctionName(){...}
export class ClassName {...}

// 导出列表
export { name1, name2, …, nameN };

// 重命名导出
export { variable1 as name1, variable2 as name2, …, nameN };

// 默认导出
export default expression;
export default function (…) { … } // also class, function*
export default function name1(…) { … } // also class, function*
export { name1 as default, … };

// Aggregating modules
export * from …;
export { name1, name2, …, nameN } from …;
export { import1 as name1, import2 as name2, …, nameN } from …;
export { default } from …;
```
一个模块就是一个独立的文件。该文件内部的所有变量，外部无法获取。如果你希望外部能够读取模块内部的某个变量，就必须使用export关键字输出该变量。

export命令可以出现在模块的任何位置，只要处于模块顶层就可以，所以不能出现块级作用域内。优先考虑使用在文件尾部export，方便浏览输出了哪些变量，使用as关键字重命名，所以一个对象可以多次输出。

export命令规定的是对外的接口，必须与模块内部的变量建立一一对应关系。
```JavaScript
// 报错
export 1;

// 报错
var m = 1;
export m;

// 写法一
export var m = 1;

// 写法二
var m = 1;
export {m};

// 写法三
var n = 1;
export {n as m};
```

另外，export语句输出的接口，与其对应的值是动态绑定关系，即通过该接口，可以取到模块内部实时的值。
```JavaScript
// test.ts
export let foo = 'bar';
setTimeout(() => foo = 'baz', 500); // 输出变量foo，值为bar，500 毫秒之后变成baz。

// main.ts
import { foo } from './test';

console.log(foo);   // bar
setTimeout(() => console.log(foo), 100);    // baz
```

### import

[import | MDN - Mozilla](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/import)，语法：
```JavaScript
import defaultExport from "module-name";
import * as name from "module-name";
import { export } from "module-name";
import { export as alias } from "module-name";
import { export1 , export2 } from "module-name";
import { foo , bar } from "module-name/path/to/specific/un-exported/file";
import { export1 , export2 as alias2 , [...] } from "module-name";
import defaultExport, { export [ , [...] ] } from "module-name";
import defaultExport, * as name from "module-name";
import "module-name";
var promise = import("module-name");//这是一个处于第三阶段的提案。
```

import输入的变量（或整个模块）都是只读的，因为它的本质是输入接口，所以不允许在加载模块的脚本里面改写接口。但如果输入变量a是一个对象，改写其属性是允许的，并且其他模块也可以读到改写后的值。不过这种写法很难查错，建议凡是输入的变量都当作完全只读，不要轻易改变它的属性。可以使用as将输入的变量重命名，
```JavaScript
import A, { a } from './xxx.js'

a = {}; // Syntax Error : 'a' is read-only;
A.a = {};   // error
a.foo = 'hello'; // ok
A.a.foo = 1;    // ok
```
import命令具有提升效果，会提升到整个模块的头部，首先执行。原因是import命令是编译阶段执行的，所以不能使用表达式和变量。
```JavaScript
foo();  // ok

import { foo } from 'my_module';
import { 'f' + 'oo' } from 'my_module'; // error

let module = 'my_module';
import { foo } from module;  // error

if (x === 1) {
  import { foo } from 'module1';     // error
} else {
  import { foo } from 'module2';     // error
}
```

import语句是 Singleton 模式，所以多次重复执行同一句import语句，那么只会执行一次。下面foo和bar在两个语句中加载，但是它们对应的是同一个my_module实例。
```JavaScript
// 多次加载，但只会执行一次 
import 'lodash';
import 'lodash';

import { foo } from 'my_module';
import { bar } from 'my_module';

// 等同于
import { foo, bar } from 'my_module';
```

### 整体模块与默认变量

先说简单的import
```JavaScript
import { area, circumference } from './circle';

// 等效于
import * as circle from './circle';
const { area, circumference } = circle;
```

一个模块只能有一个export default的默认输出，所以import不加大括号输入的内容，即是模块的默认输出。本质上，export default就是输出一个叫做default的变量或方法，然后系统允许你为它取任意名字。
```JavaScript
// modules.js
function add(x, y) {
  return x * y;
}
export {add as default};
// 等同于
// export default add;

// app.js
import { default as foo } from 'modules';
// 等同于
// import foo from 'modules';

// test.js
export var a = 1;   // ok
export default var a = 1;   // error。将变量a的值赋给变量default，所以报错
export 42;   // ok

var a = 1;
export default a;   // ok
export default 42;   // ok

// 报错
```

### export与import的复合写法

```JavaScript
export { foo, bar } from 'my_module';

// 可以简单理解为
import { foo, bar } from 'my_module';
export { foo, bar };
```
注意，foo和bar实际上并没有被导入当前模块，只是相当于对外转发了这两个接口，导致当前模块不能直接使用foo和bar。

```JavaScript
// 接口改名
export { foo as myFoo } from 'my_module';

// 具名接口与默认接口相互改写
export { es6 as default } from './someModule';  // 这里的default会占用当前文件的default，所以当前模块不能再有 export default
export { default } from './someModule'; // （同上）
export { default as es6 } from './someModule';

// 整体输出，可以看作模块的继承。。但是会忽略my_module模块的default方法，所以需要将my_module的default改写
export * from 'my_module';  // 很重要！！！
export { default as defaultMyModule } from 'my_module';

// 下面三种没有符合写法
import * as someIdentifier from "someModule";
import someIdentifierDefault from "someModule";
import someIdentifierDefault, { namedIdentifier } from "someModule";
```

### 跨模块常量

前面介绍了export语句输出的接口，与其对应的值是动态绑定关系，即通过该接口，可以取到模块内部实时的值。即使const声明的常量只在当前代码块有效，也可以通过export设置跨模块的常量（即跨多个文件），然后该值被多个模块共享。
```JavaScript
// constants/db.js
export const db = {
  url: 'http://my.couchdbserver.local:5984',
  admin_username: 'admin',
  admin_password: 'admin password'
};

// constants/user.js
export const users = ['root', 'admin', 'staff', 'ceo', 'chief', 'moderator'];

// constants/index.js
export {db} from './db';
export {users} from './users';

// script.js
import {db, users} from './constants/index';
```

### 其他

[ES2020提案](https://github.com/tc39/proposal-dynamic-import) 引入import()函数，支持动态加载模块。


### 浏览器加载

#### 传统方法

默认情况下，浏览器是同步加载 JavaScript 脚本，即渲染引擎遇到<script>标签就会停下来，等到执行完脚本，再继续向下渲染。如果是外部脚本，还必须加入脚本下载的时间。
浏览器允许脚本异步加载，下面就是两种异步加载的语法。渲染引擎遇到这一行命令，就会开始下载外部脚本，但不会等它下载和执行，而是直接执行后面的命令。
```JavaScript
<script src="path/to/myModule.js" defer></script>
<script src="path/to/myModule.js" async></script>
```
defer与async的区别是：defer要等到整个页面在内存中正常渲染结束（DOM 结构完全生成，以及其他脚本执行完成），才会执行；async一旦下载完，渲染引擎就会中断渲染，执行这个脚本以后，再继续渲染。一句话，defer是“渲染完再执行”，async是“下载完就执行”。另外，如果有多个defer脚本，会按照它们在页面出现的顺序加载，而多个async脚本是不能保证加载顺序的。

#### 加载规则

浏览器加载 ES6 模块，也使用<script>标签，但是要加入type="module"属性。ES6 模块是异步加载，不会造成堵塞浏览器，即等到整个页面渲染完，再执行模块脚本，等同于打开了<script>标签的defer属性。多个模块会按照在页面出现的顺序依次执行。
```JavaScript
<script type="module" src="./foo.js"></script>
<!-- 等同于 -->
<script type="module" src="./foo.js" defer></script>
```
使用async属性后，则只要加载完成，渲染引擎就会中断渲染立即执行，执行完成后再恢复渲染，不保证执行顺序。

ES6 模块也允许内嵌在网页中，语法行为与加载外部脚本完全一致。
```JavaScript
<script type="module">
  import utils from "./utils.js";

  // other code
</script>
```
对于外部的模块脚本，有几点需要注意。
* 代码是在模块作用域之中运行，而不是在全局作用域运行。模块内部的顶层变量，外部不可见。
* 模块脚本自动采用严格模式，不管有没有声明use strict。
* 模块之中，可以使用import命令加载其他模块（.js后缀不可省略，需要提供绝对 URL 或相对 URL），也可以使用export命令输出对外接口。
* 模块之中，顶层的this关键字返回undefined，而不是指向window。也就是说，在模块顶层使用this关键字，是无意义的。
* 同一个模块如果加载多次，将只执行一次。
```JavaScript
import utils from 'https://example.com/js/utils.js';

const x = 1;

console.log(x === window.x); //false
console.log(this === undefined); // true

// 侦测当前代码是否在 ES6 模块之中
const isNotModuleScript = this !== undefined;
```

### ES6 模块与 CommonJS 模块的差异

讨论 Node.js 加载 ES6 模块之前，必须了解 ES6 模块与 CommonJS 模块完全不同。它们有两个重大差异。
* CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
* CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。

第二个差异是因为 CommonJS 加载的是一个对象（即module.exports属性），该对象只有在脚本运行完才会生成。而 ES6 模块不是对象，它的对外接口只是一种静态定义，在代码静态解析阶段就会生成。

CommonJS 模块输出的是值的拷贝，也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。除非写成一个函数，才能得到内部变动后的值。
```JavaScript
// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  get counter() {
    return counter
  },
  incCounter: incCounter,
};
```
ES6 模块的运行机制与 CommonJS 不一样。JS 引擎对脚本静态分析的时候，遇到模块加载命令import，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值。换句话说，ES6 的import有点像 Unix 系统的“符号连接”，原始值变了，import加载的值也会跟着变。因此，ES6 模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块。

由于 ES6 输入的模块变量，只是一个“符号连接”，所以这个变量是只读的，对它进行重新赋值会报错。


### Node.js 加载

Node.js 对 ES6 模块的处理比较麻烦，因为它有自己的 CommonJS 模块格式，与 ES6 模块格式是不兼容的。目前的解决方案是，将两者分开，ES6 模块和 CommonJS 采用各自的加载方案。从 v13.2 版本开始，Node.js 已经默认打开了 ES6 模块支持。

Node.js 要求 ES6 模块采用.mjs后缀文件名，默认启用严格模式。如果不希望将后缀名改成.mjs，可以在项目的package.json文件中，指定type字段为module。一旦设置了以后，该目录里面的 JS 脚本，就被解释用 ES6 模块。
```JavaScript
"type": "module",
```
**如果没有type字段，或者type字段为commonjs，则.js脚本会被解释成 CommonJS 模块。ES6 模块与 CommonJS 模块尽量不要混用。** require命令不能加载.mjs文件，会报错，只有import命令才可以加载.mjs文件。反过来，.mjs文件里面也不能使用require命令，必须使用import。

#### main 字段

package.json文件有两个字段可以指定模块的入口文件：main和exports。比较简单的模块，可以只使用main字段，指定模块加载的入口文件。
```JavaScript
// ./node_modules/es-module-package/package.json
{
  "type": "module",
  "main": "./src/index.js"
}

// ./my-app.mjs
import { something } from 'es-module-package';
// 实际加载的是 ./node_modules/es-module-package/src/index.js
```
运行该脚本以后，Node.js 就会到./node_modules目录下面，寻找es-module-package模块，然后根据该模块package.json的main字段去执行入口文件。这时，如果用 CommonJS 模块的require()命令去加载es-module-package模块会报错，因为 CommonJS 模块不能处理export命令。

#### exports 字段

exports字段的优先级高于main字段。它有多种用法。

（1）子目录别名：package.json文件的exports字段可以指定脚本或子目录的别名。
```JavaScript
// ./node_modules/es-module-package/package.json
{
  "exports": {
    "./submodule": "./src/submodule.js",
    "./features/": "./src/features/"
  }
}

import submodule from 'es-module-package/submodule';
// 加载 ./node_modules/es-module-package/src/submodule.js

import feature from 'es-module-package/features/x.js';
// 加载 ./node_modules/es-module-package/src/features/x.js
```

如果没有指定别名，就不能用“模块+脚本名”这种形式加载脚本。
```JavaScript
// 报错
import submodule from 'es-module-package/private-module.js';

// 不报错
import submodule from './node_modules/es-module-package/private-module.js';
```

（2）main 的别名：exports字段的别名如果是.，就代表模块的主入口，优先级高于main字段，并且可以直接简写成exports字段的值。
```JavaScript
{
  "exports": {
    ".": "./main.js"
  }
}

// 等同于
{
  "exports": "./main.js"
}
```
兼容不支持ES6旧版本的 Node.js。
```JavaScript
{
  "main": "./main-legacy.cjs",
  "exports": {
    ".": "./main-modern.cjs"
  }
}
```

（3）条件加载：利用.这个别名，可以为 ES6 模块和 CommonJS 指定不同的入口。目前，这个功能需要在 Node.js 运行的时候，打开--experimental-conditional-exports标志。只有别名.的情况下可以简写exports。
```JavaScript
{
  "type": "module",
  "exports": {
    ".": {
      "require": "./main.cjs",
      "default": "./main.js"
    },
    "./feature": "./lib/feature.js"
  }
}
```

#### ES6 模块加载 CommonJS 模块

```JavaScript
// ./node_modules/pkg/index.cjs
exports.name = 'value';

// ./node_modules/pkg/wrapper.mjs
import cjsModule from './index.cjs';
export const name = cjsModule.name;
```
注意，import命令加载 CommonJS 模块，只能整体加载，不能只加载单一的输出项（解构式加载）。


#### CommonJS 模块加载 ES6 模块

CommonJS 的require命令不能加载 ES6 模块，会报错，只能使用import()这个方法加载。
```JavaScript
(async () => {
  await import('./my-app.mjs');
})();
```

#### 其他

Node.js 的内置模块可以整体加载，也可以加载指定的输出项。

ES6 模块的加载路径必须给出脚本的完整路径，不能省略脚本的后缀名。import命令和package.json文件的main字段如果省略脚本的后缀名，会报错。为了与浏览器的import加载规则相同，Node.js 的.mjs文件支持 URL 路径。
```JavaScript
// ES6 模块中将报错
import { something } from './index';

import './foo.mjs?query=1'; // 加载 ./foo 传入参数 ?query=1
```
脚本路径带有参数?query=1，Node 会按 URL 规则解读。同一个脚本只要参数不同，就会被加载多次，并且保存成不同的缓存。由于这个原因，只要文件名中含有:、%、#、?等特殊字符，最好对这些字符进行转义。

目前，Node.js 的import命令只支持加载本地模块（file:协议）和data:协议，不支持加载远程模块。另外，脚本路径只支持相对路径，不支持绝对路径（即以/或//开头的路径）。Node 的import命令是异步加载，这一点与浏览器的处理方法相同。

ES6 模块应该是通用的，同一个模块不用修改，就可以用在浏览器环境和服务器环境。为了达到这个目标，Node 规定 ES6 模块之中不能使用 CommonJS 模块的特有的一些内部变量。

首先，就是this关键字。ES6 模块之中，顶层的this指向undefined；CommonJS 模块的顶层this指向当前模块，这是两者的一个重大差异。其次，以下这些顶层变量在 ES6 模块之中都是不存在的。
* arguments
* require
* module
* exports
* \_\_filename
* \_\_dirname

### 循环加载

“循环加载”（circular dependency）指的是，a脚本的执行依赖b脚本，而b脚本的执行又依赖a脚本。“循环加载”表示存在强耦合，如果处理不好，还可能导致递归加载，使得程序无法执行，因此应该避免出现。CommonJS 和 ES6，处理“循环加载”的方法是不一样的，返回的结果也不一样。

CommonJS 的一个模块，就是一个脚本文件。require命令第一次加载该脚本，就会执行整个脚本，然后在内存生成一个对象。
```JavaScript
{
  id: '...',
  exports: { ... },
  loaded: true,
  ...
}
```
上面代码就是 Node 内部加载模块后生成的一个对象。该对象的id属性是模块名，exports属性是模块输出的各个接口，loaded属性是一个布尔值，表示该模块的脚本是否执行完毕。其他还有很多属性，这里都省略了。

以后需要用到这个模块的时候，就会到exports属性上面取值。即使再次执行require命令，也不会再次执行该模块，而是到缓存之中取值。也就是说，CommonJS 模块无论加载多少次，都只会在第一次加载时运行一次，以后再加载，就返回第一次运行的结果，除非手动清除系统缓存。

**CommonJS 模块的重要特性是加载时执行，即脚本代码在require的时候，就会全部执行。一旦出现某个模块被"循环加载"，就只输出已经执行的部分，还未执行的部分不会输出。** 

（跳过 CommonJS 模块的循环加载）

ES6 处理“循环加载”与 CommonJS 有本质的不同。**ES6 模块是动态引用，如果使用import从一个模块加载变量（即import foo from 'foo'），那些变量不会被缓存，而是成为一个指向被加载模块的引用，需要开发者自己保证，真正取值的时候能够取到值。**

*因ES6是编译时完成模块加载，所以当发现需要加载新模块时，会暂停后面的定义导出，此时其他模块来引用当前模块还未导出的定义就会报错。*