

# MySQL日常记录

## 安装与配置



1. [Centos上安装Mysql-server](https://typecodes.com/linux/yuminstallmysql5710.html)，[重设root密码](https://my.oschina.net/zhailibao2010/blog/529887)；
2. [在CentOS7上使用yum安装MySQL 5.7](https://blog.frognew.com/2017/05/yum-install-mysql-5.7.html)；
3. [MySQL创建用户与授权](http://www.jianshu.com/p/d7b9c468f20d)；
4. [支持Emoji符号存储](https://segmentfault.com/a/1190000006851140)；



/etc/my.cnf里加一个『validate_password=off』关闭密码验证，重启生效；



## 常用命令



### 创建用户

```mysql
/** 创建用户 */
CREATE USER 'user'@'%' IDENTIFIED BY 'password';

/** 创建数据库 */
CREATE DATABASE newdb DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;

/** 授权用户访问数据库 */
GRANT all ON newdb.* TO 'user'@'%';

/** 刷新权限 */
flush privileges
```




## 概念

hibernate的save方法和persist方法的区别

- save：在事务里执行到save，会向数据库插一条数据，如果事务里异常，会回滚，删除数据库中插入的数据。

- persist：在事务里执行到persist，不会向数据库插数据，事务commit了才会插入数据。

