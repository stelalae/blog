# TypeORM学习-日常使用

本篇记录日常中常见的问题，主要是CRUD。两个版本API，内容大部分一样：
- [repository-api](https://github.com/typeorm/typeorm/blob/master/docs/zh_CN/repository-api.md)
- [entity-manager-api](https://github.com/typeorm/typeorm/blob/master/docs/zh_CN/entity-manager-api.md)，仅包含EntityManager

* * *

[TOC]


## 常用API


hasId - 检查是否定义了给定实体的主列属性。
getId - 获取给定实体的主列属性值。复合主键返回的值将是一个具有主列名称和值的对象。
save - 保存给定实体或实体数组。不存在则插入，存在则更新，支持部分属性更新。Active Record模式下是事务的，在Data Mapper模式下不是事务的。
```TypeScript
await repository.save(user);
await repository.save([category1, category2, category3]);
```

remove - 删除给定的实体或实体数组。
```TypeScript
await repository.save(user);
await repository.save([category1, category2, category3]);
```

insert - 插入新实体或实体数组。
```TypeScript
await repository.insert({
  firstName: "Timber",
  lastName: "Timber"
});

await manager.insert(User, [
  {
    firstName: "Foo",
    lastName: "Bar"
  },
  {
    firstName: "Rizz",
    lastName: "Rak"
  }
]);
```

update - 通过给定的更新选项或实体 ID 部分更新实体。
```TypeScript
await repository.update({ firstName: "Timber" }, { firstName: "Rizzrak" });
// 执行 UPDATE user SET firstName = Rizzrak WHERE firstName = Timber

await repository.update(1, { firstName: "Rizzrak" });
// 执行 UPDATE user SET firstName = Rizzrak WHERE id = 1
```

delete -根据实体 id, ids 或给定的条件删除实体：
```TypeScript
await repository.delete(1);
await repository.delete([1, 2, 3]);
await repository.delete({ firstName: "Timber" });
```

count - 符合指定条件的实体数量。对分页很有用。
```TypeScript
const count = await repository.count({ firstName: "Timber" });
```

**find** - 查找指定条件的实体，[详细option](https://github.com/typeorm/typeorm/blob/master/docs/zh_CN/find-options.md)。
```TypeScript
const count = await repository.count({ firstName: "Timber" });

userRepository.find({
    select: ["firstName", "lastName"],
    relations: ["profile", "photos", "videos"],
    where: {
        firstName: "Timber",
        lastName: "Saw"
    },
    order: {
        name: "ASC",
        id: "DESC"
    },
    skip: 5,
    take: 10,
    cache: true
});
```

findAndCount - 查找指定条件的实体。还会计算与给定条件匹配的所有实体数量， 但是忽略分页设置 (skip 和 take 选项)。
```TypeScript
const [timbers, timbersCount] = await repository.findAndCount({ firstName: "Timber" });
```

findByIds - 按 ID 查找多个实体。
```TypeScript
const users = await repository.findByIds([1, 2, 3]);
```

findOne - 查找匹配某些 ID 或查找选项的第一个实体。
```TypeScript
const user = await repository.findOne(1);
const timber = await repository.findOne({ firstName: "Timber" });
```

findOneOrFail - 查找匹配某些 ID 或查找选项的第一个实体。 如果没有匹配，则 Rejects 一个 promise。
```TypeScript
const user = await repository.findOneOrFail(1);
const timber = await repository.findOneOrFail({ firstName: "Timber" });
```

query - 执行原始 SQL 查询。
```TypeScript
const rawData = await repository.query(`SELECT * FROM USERS`);
```

clear - 清除给定表中的所有数据(truncates/drops)。
```TypeScript
await repository.clear();
```

### 其他选项

**SaveOptions**选项可以传递save, insert 和 update参数。

- save：在事务里执行到save，会向数据库插一条数据，如果事务里异常，会回滚，删除数据库中插入的数据。
- persist：在事务里执行到persist，不会向数据库插数据，事务commit了才会插入数据。

**RemoveOptions**可以传递remove和delete参数。

## 缓存查询

QueryBuilder和Repository查询后的结果都可以缓存，需在`ormconfig.json`中启用`cache: true`。*首次启用缓存时，必须同步数据库架构（使用 CLI，migrations 或synchronize连接选项）。*

```TypeScript
// 全局配置
{
    type: "mysql",
    host: "localhost",
    username: "test",
    ...
    cache: {    // 或 cache: true
        duration: 30000 // 30秒
    }
}
```

```TypeScript
// 默认缓存生存期为1000 ms
const users = await connection
  .createQueryBuilder(User, "user")
  .where("user.isAdmin = :isAdmin", { isAdmin: true })
  .cache(true)  // cache(60000) - 1分钟
  .getMany();
  
// 等效于

const users = await connection.getRepository(User).find({
  where: { isAdmin: true },
  cache: true   // cache(60000) - 1分钟
});
```

默认情况下，使用一个名为query-result-cache的单独表，并在那里存储所有查询和结果，表名是可配置的。也可以通过设置"cache id"精确控制缓存。
```TypeScript
const users = await connection
  .createQueryBuilder(User, "user")
  .where("user.isAdmin = :isAdmin", { isAdmin: true })
  .cache("users_admins", 25000)
  .getMany();

const users = await connection.getRepository(User).find({
  where: { isAdmin: true },
  cache: {
    id: "users_admins",
    milisseconds: 25000
  }
});

// 手动清除缓存
await connection.queryResultCache.remove(["users_admins"]);
```

在`ormconfig.json`中`cache`里，也可将缓存类型更改为"redis"或者"ioredis"，options参考[node_redis specific options](https://github.com/NodeRedis/node_redis#options-object-properties)、[ioredis specific options](https://github.com/luin/ioredis/blob/master/API.md#new-redisport-host-options)。