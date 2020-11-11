[TOC]

### 什么是ORM

对象关系映射（Object Relational Mapping，简称ORM）是通过使用描述对象和数据库之间映射的元数据，将面向对象语言程序中的对象自动持久化到关系数据库中。简单说，ORM 就是通过实例对象的语法，完成关系型数据库的操作的技术，本质上就是将数据从一种形式转换到另外一种形式。。。摘自[百度百科](https://baike.baidu.com/item/%E5%AF%B9%E8%B1%A1%E5%85%B3%E7%B3%BB%E6%98%A0%E5%B0%84/311152)

### 为什么用ORM

在程序开发中，数据库保存的表，字段与程序中的实体类之间是没有关联的，在实现持久化时就比较不方便。那么，到底如何实现持久化呢？一种简单的方案是采用硬编码方式，为每一种可能的数据库访问操作提供单独的方法。这种方案存在以下不足：

1. 持久化层缺乏弹性。一旦出现业务需求的变更，就必须修改持久化层的接口。

2. 持久化层同时与域模型与关系数据库模型绑定，不管域模型还是关系数据库模型发生变化，毒药修改持久化曾的相关程序代码，增加了软件的维护难度。

ORM提供了实现持久化层的另一种模式，它采用映射元数据来描述对象关系的映射，使得ORM中间件能在任何一个应用的业务逻辑层和数据库层之间充当桥梁。

ORM的方法论基于三个核心原则：

- 简单：以最基本的形式建模数据

- 传达性：数据库结构被任何人都能理解的语言文档化

- 精确性：基于数据模型创建正确标准化了的结构

### 优/缺点

ORM 使用对象，封装了数据库操作，因此可以不碰 SQL 语言。开发者只使用面向对象编程，与数据对象直接交互，不用关心底层数据库。

总结起来，有下面这些优点：

- 数据模型都在一个地方定义，更容易更新和维护，也利于重用代码。

- ORM 有现成的工具，很多功能都可以自动完成，比如数据消毒、预处理、事务等等。

- 它迫使你使用 MVC 架构，ORM 就是天然的 Model，最终使代码更清晰。

- 基于 ORM 的业务代码比较简单，代码量少，语义性好，容易理解。

- 你不必编写性能不佳的 SQL。

- 开发效率更高

- 数据访问更抽象、轻便

- 支持面向对象封装

也有很突出的缺点：

- ORM 库不是轻量级工具，需要花很多精力学习和设置。

- 对于复杂的查询，ORM 要么是无法表达，要么是性能不如原生的 SQL。

- ORM 抽象掉了数据库层，开发者无法了解底层的数据库操作，也无法定制一些特殊的 SQL。

- 降低程序的执行效率

- 思维固定化

### 命名规定

许多语言都有自己的 ORM 库，最典型、最规范的实现公认是 Ruby 语言的 [Active Record](https://guides.rubyonrails.org/active_record_basics.html)。Active Record 对于对象和数据库表的映射，有一些命名限制。

（1）一个类对应一张表。类名是单数，且首字母大写；表名是复数，且全部是小写。比如，表books对应类Book。

（2）如果名字是不规则复数，则类名依照英语习惯命名，比如，表mice对应类Mouse，表people对应类Person。

（3）如果名字包含多个单词，那么类名使用首字母全部大写的骆驼拼写法，而表名使用下划线分隔的小写单词。比如，表book_clubs对应类BookClub，表line_items对应类LineItem。

（4）每个表都必须有一个主键字段，通常是叫做id的整数字段。外键字段名约定为单数的表名 + 下划线 + id，比如item_id表示该字段对应items表的id字段。

### 个人心得

1、相比之前手动创建MySQL的db、table等，ORM带来更愉快的开发体验。但是门槛变得更高，需要单独学习Repository、Entity等。

2、在变更table解构时，如果做好兼容，需要仔细考虑。

3、担心性能问题，和复杂 SQL 语句的实现。

参考资料：

- [什么是ORM？为什么用ORM？浅析ORM的使用及利弊](https://segmentfault.com/a/1190000011642533)

- [ORM 实例教程](http://www.ruanyifeng.com/blog/2019/02/orm-tutorial.html)