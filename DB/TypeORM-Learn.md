# TypeORM学习

[TOC]

## Active Record 与 Data Mapper

[官网介绍](https://github.com/typeorm/typeorm/blob/master/docs/zh_CN/active-record-data-mapper.md)

### Active Record

使用 Active Record 方法，可以在模型Entity 本身内定义所有查询方法，并使用模型方法保存、删除和加载对象。简单来说，Active Record 模式是一种在模型中访问数据库的方法。 

```TypeScript

// 实体类
import { BaseEntity, Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class User extends BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  isActive: boolean;

  // 在User类中创建静态方法等函数
  static findByName(firstName: string, lastName: string) {
    return this.createQueryBuilder("user")
      .where("user.firstName = :firstName", { firstName })
      .andWhere("user.lastName = :lastName", { lastName })
      .getMany();
  }
}


// 示例如何保存AR实体
const user = new User();
user.firstName = "Timber";
user.lastName = "Saw";
user.isActive = true;
await user.save();

// 示例如何删除AR实体
await user.remove();

// 示例如何加载AR实体
const users = await User.find({ skip: 2, take: 5 });
const newUsers = await User.find({ isActive: true });
const timber = await User.findOne({ firstName: "Timber", lastName: "Saw" });

const timber2 = await User.findByName("Timber", "Saw");
```

所有实体都必须扩展BaseEntity类，它提供了与实体一起使用的方法。BaseEntity具有标准Repository的大部分方法。大多数情况下，你不需要将Repository或EntityManager与 active record 实体一起使用。

### Data Mapper

使用 Data Mapper 方法，你可以在名为"repositories"的单独类中定义所有查询方法，并使用存储库保存、删除和加载对象。简单来说，数据映射器是一种在存储库而不是模型中访问数据库的方法。个人感觉，和后面介绍的 EntityManager 比较类似。

```TypeScript

// 实体类
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  isActive: boolean;
}

// 基本使用
const userRepository = connection.getRepository(User);

// 示例如何保存DM实体
const user = new User();
user.firstName = "Timber";
user.lastName = "Saw";
user.isActive = true;
await userRepository.save(user);

// 示例如何删除DM实体
await userRepository.remove(user);

// 示例如何加载DM实体
const users = await userRepository.find({ skip: 2, take: 5 });
const newUsers = await userRepository.find({ isActive: true });
const timber = await userRepository.findOne({ firstName: "Timber", lastName: "Saw" });
```

```TypeScript

// 定义数据映射方法
import { EntityRepository, Repository } from "typeorm";
import { User } from "../entity/User";

@EntityRepository()
export class UserRepository extends Repository<User> {
  findByName(firstName: string, lastName: string) {
    return this.createQueryBuilder("user")
      .where("user.firstName = :firstName", { firstName })
      .andWhere("user.lastName = :lastName", { lastName })
      .getMany();
  }
}

// 使用
const userRepository = connection.getCustomRepository(UserRepository);
const timber = await userRepository.findByName("Timber", "Saw");
```

**在Nestjs中，推荐使用Data Mapper模式**。


## Entity

columns类型：
- PrimaryColumn：默认 int型。
- PrimaryGeneratedColumn：默认 int型。
- @PrimaryGeneratedColumn("uuid")：默认 string。

### Entity Inheritance 继承式实体

具体表继承，使用实体继承模式减少代码中的重复。 最简单和最有效的是具体的表继承。

```TypeScript

// 基类
export abstract class Content {
    @PrimaryGeneratedColumn()
    id: number;
 
    @Column()
    title: string;
}

// 子类1
@Entity()
export class Photo extends Content {
    
    @Column()
    size: string;
    
}

// 子类2
@Entity()
export class Question extends Content {
    
    @Column()
    answersCount: number;
    
}

// 子类3
@Entity()
export class Post extends Content {
    
    @Column()
    viewCount: number;
    
}
```

另外 单表继承（装饰器：TableInheritance，ChildEntity），即多个有公用Column的实体数据存在一个Table中。与所有Column放在一个实体中的区别是：使用Entity时，各个子Entity管理有差异的Column，减少代码的耦合。

### Entity Embedded 嵌入式实体

嵌入列是一个列，它接受具有自己列的类，并将这些列合并到当前实体的数据库表中。个人觉得和继承差别不大，不过可以控制生成实际DB Table中列的顺序。

```TypeScript

// 被嵌套类
import { Column} from "typeorm";

export class Name {
    
    @Column()
    first: string;
    
    @Column()
    last: string;
    
}

// 嵌套类
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";
import {Name} from "./Name";

@Entity()
export class User {
    
    @PrimaryGeneratedColumn()
    id: string;
    
    @Column(type => Name)
    name: Name;
    
    @Column()
    isActive: boolean;
    
}

@Entity()
export class Student {
    
    @PrimaryGeneratedColumn()
    id: string;
    
    @Column(type => Name)
    name: Name;
    
    @Column()
    faculty: string;
    
}
```


## Repository

Repository用来表示存储库（数据层，DAO），可以看做Entity的集合，即Table与DB的关系。

从repository 角度看是，Repository包含4种：
- EntityManager：自带的，最基本的。提供给所有的Entity使用。
- TreeRepository：自带的。提供给几个能组成Tree的Entity集合使用，意思是父子Entity之间必须某一Column进行关联。
- MongoRepository：自带的。
- custom-repository：自定义存储库。通常为单个实体创建自定义存储库，并包含其特定查询。即在Repository的基础上，通过继承或扩展，定制对DB的CRUD。

### Entity Manager

EntityManager是最基本的Repository管理器，你可以管理（insert, update, delete, load 等）任何实体，就像放一个实体存储库的集合的地方。你可以通过getManager（）或Connection访问实体管理器。
```TypeScript
import { getManager } from "typeorm";
import { User } from "./entity/User";

const entityManager = getManager(); // 你也可以通过 getConnection().manager 获取
const user = await entityManager.findOne(User, 1);
user.name = "Umed";
await entityManager.save(user);
```

#### API列表

两个版本，内容大部分一样：
- [repository-api](https://github.com/typeorm/typeorm/blob/master/docs/zh_CN/repository-api.md)
- [entity-manager-api](https://github.com/typeorm/typeorm/blob/master/docs/zh_CN/entity-manager-api.md)，仅包含EntityManager


### 自定义存储库

[详细说明](https://github.com/typeorm/typeorm/blob/master/docs/zh_CN/custom-repository.md)

#### 扩展了标准存储库的定制存储库

#### 扩展了标准AbstractRepository的自定义存储库

#### 没有扩展的自定义存储库

#### 在事务中使用自定义存储库或为什么自定义存储库不能是服务


## 日志

[官方文档](https://github.com/typeorm/typeorm/blob/master/docs/zh_CN/logging.md)，在连接配置中增加`logging: true`，或记录所有日志`logging: 'all'`，或增加日志过滤器`logging: ['query', 'error']`。支持过滤器类型：
- query：记录所有查询。
- error：记录所有失败的查询和错误。
- schema：记录架构构建过程。
- warn：记录内部 orm 警告。
- info：记录内部 orm 信息性消息。
- log：记录内部 orm 日志消息。

另外设置`maxQueryExecutionTime: 1000`来记录所有运行超过1秒的查询。

自带4种不同类型的记录器，通过`logger: "file"`启用：
advanced-console - 默认记录器，它将使用颜色和 sql 语法高亮显示所有记录到控制台中的消息（使用[chalk](https://github.com/chalk/chalk)）。
simple-console - 简单的控制台记录器，与高级记录器完全相同，但它不使用任何颜色突出显示。 如果你不喜欢/或者使用彩色日志有问题，可以使用此记录器。
file - 这个记录器将所有日志写入项目根文件夹中的ormlogs.log（靠近package.json和ormconfig.json）。
debug - 此记录器使用[debug package](https://github.com/visionmedia/debug)打开日志记录设置你的 env 变量DEBUG = typeorm：*（注意记录选项对此记录器没有影响）。

也可自定义记录器`logger: new MyCustomLogger()`：
```TypeScript
import { Logger } from "typeorm";

export class MyCustomLogger implements Logger {
  // 实现logger类的所有方法
}
```
