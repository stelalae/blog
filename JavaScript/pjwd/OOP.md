# 面向对象的程序设计

[TOC]

> ECMA-262 把对象定义为：“无序属性的集合，其属性可以包含基本值、对象或者函数。”严格来讲，
> 这就相当于说对象是一组没有特定顺序的值。对象的每个属性或方法都有一个名字，而每个名字都映射
> 到一个值。正因为这样（以及其他将要讨论的原因），我们可以把 ECMAScript 的对象想象成散列表：无
> 非就是一组名值对，其中值可以是数据或函数。
> 每个对象都是基于一个引用类型创建的，这个引用类型可以是第 5 章讨论的原生类型，也可以是开
> 发人员定义的类型。

## 理解对象

```javascript
var person = new Object(); 
person.name = "Nicholas"; 
person.age = 29; 
person.job = "Software Engineer"; 
person.sayName = function(){ 
	alert(this.name); 
};
// 等效于
var person = { 
  name: "Nicholas", 
  age: 29, 
  job: "Software Engineer", 
  sayName: function(){ 
  	alert(this.name); 
  } 
};
```

### 属性类型

ECMAScript 中有两种属性：数据属性和访问器属性。

#### 数据属性

![eJHbDh](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/07/14/ohtQGT.png)

> 笔者住，这里数据属性是指描述对象中一个成员的信息，即上面person对象的4个成员分别有各自的数据属性。

要修改属性默认的特性，必须使用 ECMAScript 5 的 Object.defineProperty()方法。这个方法接收三个参数：属性所在的对象、属性的名字和一个描述符对象。其中，描述符（descriptor）对象的属性必须是：configurable、enumerable、writable 和 value。设置其中的一或多个值，可以修改对应的特性值。

```javascript
var person = {}; 
Object.defineProperty(person, "name", { 
  writable: false, 
  value: "Nicholas" 
}); 
alert(person.name); //"Nicholas" 
person.name = "Greg"; // 非严格模式下该行代码不生效，严格模式下会抛出错误
alert(person.name); //"Nicholas"

Object.defineProperty(person, "name", { 
  configurable: false, 
  value: "Nicholas" 
});

// 抛出错误。设置为 false 之后就会有限制
Object.defineProperty(person, "name", { 
  configurable: true, 
  value: "Nicholas" 
});
```

在调用 Object.defineProperty()方法时，如果不指定，configurable、enumerable 和writable 特性的默认值都是 false。多数情况下都没有必要利用 Object.defineProperty()方法提供的这些高级功能。不过，理解这些概念对理解 JavaScript 对象却非常有用。

#### 访问器属性

![dzXbqG](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/07/14/dzXbqG.png)

不一定非要同时指定 getter 和 setter。只指定 getter 意味着属性是不能写，尝试写入属性会被忽略。在严格模式下，尝试写入只指定了 getter 函数的属性会抛出错误。类似地，只指定 setter 函数的属性也不能读，否则在非严格模式下会返回 undefined，而在严格模式下会抛出错误。

### 定义多个属性

Object.defineProperties() 可以通过描述符一次定义多个属性，接收两个对象参数：第一个对象是要添加和修改其属性的对象，第二个对象的属性与第一个对象中要添加或修改的属性一一对应。

```javascript
var book = {}; 
Object.defineProperties(book, { 
  _year: { 
  	value: 2004 
  }, 
  edition: { 
  	value: 1 
  }, 
  year: { 
    get: function(){
    	return this._year; 
    }, 
    set: function(newValue){ 
      if (newValue > 2004) { 
        this._year = newValue; 
        this.edition += newValue - 2004; 
      } 
    } 
  } 
});
```

### 读取属性的特性

Object.getOwnPropertyDescriptor() 可以取得给定属性的描述符，接收两个参数：属性所在的对象和要读取其描述符的属性名称。返回值是一个对象：

- 如果是访问器属性，这个对象的属性有 configurable、enumerable、get 和 set；
- 如果是数据属性，这个对象的属性有 configurable、enumerable、writable 和 value。

```javascript
// 结合上面的例子
var descriptor = Object.getOwnPropertyDescriptor(book, "_year"); 
alert(descriptor.value); //2004 
alert(descriptor.configurable); //false
alert(typeof descriptor.get);//"undefined" 

var descriptor = Object.getOwnPropertyDescriptor(book, "year"); 
alert(descriptor.value); //undefined 
alert(descriptor.enumerable); //false 
alert(typeof descriptor.get); //"function"
```

## 创建对象

### 工厂模式

```javascript
function Person(name, age, job){ 
  this.name = name; 
  this.age = age; 
  this.job = job; 
  this.sayName = function(){ 
    alert(this.name); 
  }; 
} 
var person1 = new Person("Nicholas", 29, "Software Engineer"); 
var person2 = new Person("Greg", 27, "Doctor");
```

### 构造函数模式

构造函数始终都应该以一个大写字母开头，而非构造函数则应该以一个小写字母开头。

> 这个做法借鉴自其他 OO 语言，主要是为了区别于 ECMAScript 中的其他函数；因为构造函数本身也是函数，只不过可以用来创建对象而已。

使用 new 操作符会经历以下 4个步骤：

1. 创建一个新对象；
2. 将构造函数的作用域赋给新对象（因此 this 就指向了这个新对象）；
3. 执行构造函数中的代码（为这个新对象添加属性）；
4. 返回新对象。

自定义的构造函数可以将它的实例标识为一种特定的类型，而这正是构造函数模式胜过工厂模式的地方。

#### 将构造函数当作函数

构造函数与其他函数的唯一区别：就在于调用它们的方式不同。构造函数也是不存在特殊语法的函数，任何函数只要通过 new 操作符来调用，那它就可以作为构造函数；如果不通过 new 操作符来调用，那它跟普通函数也不会有什么两样。

```javascript
// 当作构造函数使用
var person = new Person("Nicholas", 29, "Software Engineer"); 
person.sayName(); //"Nicholas" 

// 作为普通函数调用
Person("Greg", 27, "Doctor"); // 当在全局作用域中调用一个函数时，this 对象总是指向 Global 对象
window.sayName(); //"Greg" 

// 在另一个对象的作用域中调用
var o = new Object(); 
Person.call(o, "Kristen", 25, "Nurse"); 
o.sayName(); //"Kristen"
```

#### 构造函数的问题

使用构造函数的主要问题，就是每个方法都要在每个实例上重新创建一遍，即以这种方式创建函数会导致不同的作用域链和标识符解析。如下面代码，虽创建 Function 新实例的机制仍然是相同的，但不同实例上的同名函数是不相等的。

```javascript
function Person(name, age, job){ 
	this.name = name; 
	this.age = age; 
	this.job = job; 
	this.sayName = new Function("alert(this.name)"); // 与声明函数在逻辑上是等价的
}
alert(person1.sayName == person2.sayName); //false，不是同一个 Function 的实例
```

可以将sayName改成全局函数，但这样就有全局污染。

```javascript
function Person(name, age, job){ 
  this.name = name; 
  this.age = age; 
  this.job = job; 
  this.sayName = sayName; 
} 
function sayName(){ 
	alert(this.name); 
}
```

### 原型模式

每个函数都有一个 prototype（原型）属性，这个属性是一个指向一个对象的指针，而这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法。

> 按照字面意思来理解，prototype 就是通过调用构造函数而创建的那个对象实例的原型对象。使用原型对象的好处是可以让所有对象实例共享它所包含的属性和方法。换句话说，不必在构造函数中定义对象实例的信息，而是可以将这些信息直接添加到原型对象中。

#### 理解原型模型

无论什么时候，只要创建了一个新函数，就会根据一组特定的规则为该函数创建一个 prototype 属性，这个属性指向函数的原型对象。在默认情况下，所有原型对象都会自动获得一个 constructor（构造函数）属性，这个属性包含一个指向 prototype 属性所在函数的指针。

> 就拿前面的例子来说，Person.prototype.constructor 指向 Person。而通过这个构造函数，我们还可继续为原型对象添加其他属性和方法。

创建了自定义的构造函数之后，其原型对象默认只会取得 constructor 属性；至于其他方法，则都是从 Object 继承而来的。当调用构造函数创建一个新实例后，该实例的内部将包含一个指针（内部属性），指向构造函数的原型对象。

![I33hYx](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/07/15/I33hYx.png)

![050oZ8](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/07/15/050oZ8.png)

当读取某个对象的某个属性时，都会执行一次搜索，目标是具有给定名字的属性。规则如下：

- 搜索首先从对象实例本身开始。如果在实例中找到了具有给定名字的属性，则返回该属性的值；
- 如果没有找到，则继续搜索指针指向的原型对象，在原型对象中查找具有给定名字的属性；
- 如果在原型对象中找到了这个属性，则返回该属性的值。

虽然可以通过对象实例访问保存在原型中的值，但却不能通过对象实例重写原型中的值。当在实例中添加了一个与实例原型同名的属性时，该属性将会屏蔽原型中的那个属性。

> - 将这个新增的同名属性设置为 null，而不会恢复其指向原型的连接。
>
> - 使用 delete 操作符则可以完全删除实例属性，从而让我们能够重新访问原型中的属性。

hasOwnProperty() 可以检测属性是存在于实例还是原型中，在给定属性存在于对象实例中时返回 true。

![HWj1mx](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/07/15/HWj1mx.png)

> ECMAScript 5 的 Object.getOwnPropertyDescriptor() 只能用于实例属性，取得原型属性的描述符则使用在原型对象上的 Object.getOwnPropertyDescriptor()。

#### 原型与 in 操作符

有两种方式使用 in 操作符：单独使用和在 for-in 循环中使用。在单独使用时，in 操作符会在通过对象能够访问给定属性时返回 true，无论该属性存在于实例中还是原型中。

```javascript
var person1 = new Person(); 
var person2 = new Person(); 

alert(person1.hasOwnProperty("name")); //false 
alert("name" in person1); //true 

person1.name = "Greg"; 
alert(person1.name); //"Greg" ——来自实例
alert(person1.hasOwnProperty("name")); //true 
alert("name" in person1); //true 

alert(person2.name); //"Nicholas" ——来自原型
alert(person2.hasOwnProperty("name")); //false 
alert("name" in person2); //true 

delete person1.name; 
alert(person1.name); //"Nicholas" ——来自原型
alert(person1.hasOwnProperty("name")); //false 
alert("name" in person1); //true
```

for-in 循环能返回所有能够通过对象访问的、可枚举的（enumerated）属性，其中包括实例和原型中的属性。

> 屏蔽了原型中不可枚举属性（即将[[Enumerable]]标记为 false 的属性）的实例属性也会在 for-in 循环中返回，因为根据规定，所有开发人员定义的属性都是可枚举的。

Object.keys() 可取得对象上所有可枚举的实例属性。接收一个对象作为参数，返回一个包含所有可枚举属性的字符串数组。

```javascript
function Person(){
}
Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engin";
Person.prototype.sayName = function() {
  alert(this.name);
};

var keys = Object.keys(Person.prototype);  // ['name', 'age', 'job', 'sayName']

var p1 = new Person(); 
p1.name = "Rob";
p1.age = 31; 
var keys2 = Object.keys(p1); // ['name', 'age']
for (var k in p1) { 
  console.log(k)； // ['name', 'age', 'job', 'sayName']
}
```

Object.getOwnPropertyNames() 得到所有实例属性，无论它是否可枚举。

```javascript
var keys = Object.getOwnPropertyNames(Person.prototype); 
alert(keys); //"constructor,name,age,job,sayName"，其中constructor是不可枚举的
```

> Object.keys()和 Object.getOwnPropertyNames()方法都可以用来替代 for-in 循环。

#### 更简单的原型语法

使用字面量方式来创建prototype，但这时的 constructor 属性的指向发生了变化。

> 每创建一个函数就会同时创建它的 prototype 对象，这个对象也会自动获得 constructor 属性。而这里本质上完全重写了默认的 prototype 对象，因此 constructor 属性也就变成了新对象的 constructor 属性（指向 Object 构造函数），不再指向 Person 函数。此时尽管 instanceof操作符还能返回正确的结果，但通过 constructor 已经无法确定对象的类型了。

```javascript
function Person(){ 
}
Person.prototype = { 
  name : "Nic",
  age : 29,
  job: "Software Engineer,
  sayName : function () {
    alert(this.name);
  } 
};

var friend = new Person(); 
alert(friend instanceof Object); //true 
alert(friend instanceof Person); //true 
alert(friend.constructor == Person); //false 
alert(friend.constructor == Object); //true

// 通过手动方式补救
Person.prototype = { 
	constructor : Person, 
  ...... // 其他的同上
};
Object.defineProperty(Person.prototype, "constructor", { 
  enumerable: false, 
  value: Person 
});
```

注意，手动补救的 constructor 属性会导致它的[[Enumerable]]特性被设置为 true，所以需要额外设置以完全兼容默认的配置。

#### 原型的动态性

尽管可以随时为原型添加属性和方法，并且修改能够立即在所有对象实例中反映出来，但如果是重写整个原型对象，那么情况就不一样了。我们知道，调用构造函数时会为实例添加一个指向最初原型的[[Prototype]]指针，而把原型修改为另外一个对象就等于切断了构造函数与最初原型之间的联系。*请记住：实例中的指针仅指向原型，而不指向构造函数。*

所以动态性是指原型可以修改，修改后会影响到所有实例的调用。

> 笔者住，类似于C++的动态，但不完全一致。

```javascript
function Person(){ 
} 
var friend = new Person(); 
Person.prototype = { 
  constructor: Person, 
  name : "Nicholas", 
  age : 29, 
  job : "Software Engineer", 
  sayName : function () { 
  	alert(this.name); 
  } 
}; 
friend.sayName(); //error
```

![C83J5q](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/07/15/C83J5q.png)

#### 原生对象的原型

原型模式的重要性不仅体现在创建自定义类型方面，就连所有原生的引用类型，都是采用这种模式创建的。所有原生引用类型（Object、Array、String，等等）都在其构造函数的原型上定义了方法。例如，在 Array.prototype 中可以找到 sort()方法，而在 String.prototype 中可以找到substring()方法。

所以通过修改原生对象的原型，不仅可以取得所有默认方法的引用，而且也可以定义新方法。可以像修改自定义对象的原型一样修改原生对象的原型，因此可以随时添加方法。

```javascript
String.prototype.startsWith = function (text) { 
	return this.indexOf(text) == 0; 
}; 
var msg = "Hello world!"; 
alert(msg.startsWith("Hello")); //true
```

> 尽管可以这样做，但我们不推荐在产品化的程序中修改原生对象的原型。如果因某个实现中缺少某个方法，就在原生对象的原型中添加这个方法，那么当在另一个支持该方法的实现中运行代码时，就可能会导致命名冲突。而且，这样做也可能会意外地重写原生方法。

#### 原型对象的问题

原型中所有属性是被很多实例共享的，这种共享对于函数非常合适。但其最大问题也是由其共享的本性所导致的。

- 对于那些包含基本值的属性倒也说得过去，通过在实例上添加一个同名属性，可以隐藏原型中的对应属性。
- 对于包含引用类型值的属性来说，问题就比较突出了。

### 组合使用构造函数模式和原型模式

创建自定义类型的最常见方式，就是组合使用构造函数模式与原型模式。

- 构造函数模式用于定义实例属性；

- 原型模式用于定义方法和共享的属性。

结果每个实例都会有自己的一份实例属性的副本，但同时又共享着对方法的引用，最大限度地节省了内存。另外，这种混成模式还支持向构造函数传递参数；可谓是集两种模式之长。

```javascript
function Person(name, age, job){ 
  this.name = name; 
  this.age = age; 
  this.job = job; 
  this.friends = ["Shelby", "Court"]; 
} 
Person.prototype = { 
  constructor : Person, 
  sayName : function(){ 
  	alert(this.name); 
	} 
} 
var person1 = new Person("Nicholas", 29, "Software Engineer"); 
var person2 = new Person("Greg", 27, "Doctor"); 
person1.friends.push("Van"); 
alert(person1.friends); //"Shelby,Count,Van" 
alert(person2.friends); //"Shelby,Count" 
alert(person1.friends === person2.friends); //false 
alert(person1.sayName === person2.sayName); //true
```

> 这种混成模式，是目前在 ECMAScript 中使用最广泛、认同度最高的一种创建自定义类型的方法。可以说，这是用来定义引用类型的一种默认模式。

### 动态原型模式

（跳过）

```javascript
function Person(name, age, job){ 
  //属性
  this.name = name; 
  this.age = age; 
  this.job = job;
  //方法
  if (typeof this.sayName != "function"){ 
      Person.prototype.sayName = function(){ 
        alert(this.name); 
      }; 
    } 
} 
var friend = new Person("Nicholas", 29, "Software Engineer"); 
friend.sayName();
```

### 寄生构造函数模式

基本思想是创建一个函数，该函数的作用仅仅是封装创建对象的代码，然后再返回新创建的对象；但从表面上看，这个函数又很像是典型的构造函数。

```javascript
function SpecialArray(){ 
  //创建数组
  var values = new Array(); 
  //添加值
  values.push.apply(values, arguments); 
  //添加方法
  values.toPipedString = function(){ 
  	return this.join("|"); 
  }; 
  //返回数组
  return values; 
} 
var colors = new SpecialArray("red", "blue", "green"); 
alert(colors.toPipedString()); //"red|blue|green"
```

构造函数在不返回值的情况下，默认会返回新对象实例。而通过在构造函数的末尾添加一个 return 语句，可以重写调用构造函数时返回的值。

> 首先，返回的对象与构造函数或者与构造函数的原型属性之间没有关系，即构造函数返回的对象与在构造函数外部创建的对象没有什么不同。为此，不能依赖 instanceof 操作符来确定对象类型。由于存在上述问题，我们建议在可以使用其他模式的情况下，不要使用这种模式。

### 稳妥构造函数模式

所谓稳妥对象，指的是没有公共属性，而且其方法也不引用 this 的对象。稳妥对象最适合在一些安全的环境中（这些环境中会禁止使用 this 和 new），或者在防止数据被其他应用程序（如 Mashup程序）改动时使用。稳妥构造函数遵循与寄生构造函数类似的模式，但有两点不同：一是新创建对象的实例方法不引用 this；二是不使用 new 操作符调用构造函数。

```javascript
function Person(name, age, job){ 
  //创建要返回的对象
  var o = new Object();
  
  //可以在这里定义私有变量和函数
  
  //添加方法
  o.sayName = function(){ 
    alert(name); 
  }; 
  //返回对象
  return o; 
}

var friend = Person("Nicholas", 29, "Software Engineer"); 
friend.sayName(); //"Nicholas"
```

## 继承

许多 OO 语言都支持两种继承方式：接口继承和实现继承。接口继承只继承方法签名，而实现继承则继承实际的方法。由于函数没有签名，ECMAScript 中无法实现接口继承， 只支持实现继承，而且其实现继承主要是依靠原型链来实现的。

### 原型链

其基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。简单回顾一下构造函数、原型和实例的关系：每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，而实例都包含一个指向原型对象的内部指针。

```javascript
function SuperType(){ 
	this.property = true; 
}
SuperType.prototype.getSuperValue = function(){ 
	return this.property; 
}; 
function SubType(){ 
	this.subproperty = false; 
} 
//继承了 SuperType 
SubType.prototype = new SuperType(); 
  SubType.prototype.getSubValue = function (){ 
  return this.subproperty; 
}; 
var instance = new SubType(); 
alert(instance.getSuperValue()); //true
```

![ofXRJI](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/07/16/ofXRJI.png)

![0QW4ve](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/07/16/0QW4ve.png)

#### 默认的原型

所有引用类型默认都继承了 Object，而这个继承也是通过原型链实现的。图 6-5 为我们展示了该例子中完整的原型链。

> 所有函数的默认原型都是 Object 的实例，因此默认原型都会包含一个内部指针，指向 Object.prototype。这也正是所有自定义类型都会继承 toString()、valueOf()等默认方法的根本原因。所以，我们说上面例子展示的原型链中还应该包括另外一个继承层次。

![zoGzCp](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/07/16/zoGzCp.png)

#### 确定原型和实例的关系

第一种方式是使用 instanceof 操作符，只要用这个操作符来测试实例与原型链中出现过的构造函数，结果就会返回 true。

```javascript
alert(instance instanceof Object); //true 
alert(instance instanceof SuperType); //true 
alert(instance instanceof SubType); //true
```

第二种方式是使用 isPrototypeOf()方法。同样，只要是原型链中出现过的原型，都可以说是该原型链所派生的实例的原型，因此 isPrototypeOf()方法也会返回 true，

```javascript
alert(Object.prototype.isPrototypeOf(instance)); //true 
alert(SuperType.prototype.isPrototypeOf(instance)); //true 
alert(SubType.prototype.isPrototypeOf(instance)); //true
```

子类型有时候需要重写超类型中的某个方法，或者需要添加超类型中不存在的某个方法。但不管怎样，给原型添加方法的代码一定要放在替换原型的语句之后。

原型链也存在一些问题。

- 最主要的问题来自包含引用类型值的原型。

  > 前面介绍过包含引用类型值的原型属性会被所有实例共享。而这也正是为什么要在构造函数中，而不是在原型对象中定义属性的原因。在通过原型来实现继承时，原型实际上会变成另一个类型的实例。于是，原先的实例属性也就顺理成章地变成了现在的原型属性了。

- 在创建子类型的实例时，不能向超类型的构造函数中传递参数。

  > 实际上，应该说是没有办法在不影响所有对象实例的情况下，给超类型的构造函数传递参数。有鉴于此，再加上前面刚刚讨论过的由于原型中包含引用类型值所带来的问题，实践中很少会单独使用原型链。

### 借用构造函数

（跳过）

### 组合继承

指的是将原型链和借用构造函数的技术组合到一块，从而发挥二者之长的一种继承模式。其背后的思路是使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承。这样，既通过在原型上定义方法实现了函数复用，又能够保证每个实例都有它自己的属性。

> 组合继承避免了原型链和借用构造函数的缺陷，融合了它们的优点，成为 JavaScript 中最常用的继
> 承模式。而且，instanceof 和 isPrototypeOf()也能够用于识别基于组合继承创建的对象。

```javascript
function SuperType(name){ 
  this.name = name; 
  this.colors = ["red", "blue", "green"]; 
} 
SuperType.prototype.sayName = function(){ 
	alert(this.name);
}; 
function SubType(name, age){ 
  //继承属性
  SuperType.call(this, name); 
  this.age = age; 
} 
//继承方法
SubType.prototype = new SuperType(); 
SubType.prototype.constructor = SubType; 
SubType.prototype.sayAge = function(){ 
	alert(this.age); 
}; 
var instance1 = new SubType("Nicholas", 29); 
instance1.colors.push("black"); 
alert(instance1.colors); //"red,blue,green,black" 
instance1.sayName(); //"Nicholas"; 
instance1.sayAge(); //29 

var instance2 = new SubType("Greg", 27); 
alert(instance2.colors); //"red,blue,green" 
instance2.sayName(); //"Greg"; 
instance2.sayAge(); //27
```

### 原型式继承

新增 Object.create()方法规范化了原型式继承。这个方法接收两个参数：一个用作新对象原型的对象和（可选的）一个为新对象定义额外属性的对象。在传入一个参数的情况下，Object.create()与 object()方法的行为相同。

```javascript
var person = { 
  name: "Nicholas", 
  friends: ["Shelby", "Court", "Van"] 
}; 
var anotherPerson = Object.create(person); 
anotherPerson.name = "Greg"; 
anotherPerson.friends.push("Rob"); 
var yetAnotherPerson = Object.create(person); 
yetAnotherPerson.name = "Linda"; 
yetAnotherPerson.friends.push("Barbie"); 
alert(person.friends); //"Shelby,Court,Van,Rob,Barbie"
```

Object.create()方法的第二个参数与Object.defineProperties()方法的第二个参数格式相同：每个属性都是通过自己的描述符定义的。以这种方式指定的任何属性都会覆盖原型对象上的同名属性。

```javascript
var person = { 
  name: "Nicholas", 
  friends: ["Shelby", "Court", "Van"] 
};
var anotherPerson = Object.create(person, { 
  name: { 
  	value: "Greg" 
  } 
}); 
alert(anotherPerson.name); //"Greg"
```

### 寄生式继承

（跳过）

### 寄生组合式继承

通过借用构造函数来继承属性，通过原型链的混成形式来继承方法。其背后的基本思路是：不必为了指定子类型的原型而调用超类型的构造函数，我们所需要的无非就是超类型原型的一个副本而已。本质上，就是使用寄生式继承来继承超类型的原型，然后再将结果指定给子类型的原型。

![qzazmf](https://cdn.jsdelivr.net/gh/stelalae/oss@master/files/2020/07/16/qzazmf.png)

