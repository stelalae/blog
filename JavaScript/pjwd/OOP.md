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

