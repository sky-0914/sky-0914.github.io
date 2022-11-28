---
title: Mysql5.7及以上版本 ONLY_FULL_GROUP_BY报错
tags:
  - Mysql
  - 数据库
categories:
  - 数据库
---

> 近期在开发过程中，因为项目开发环境连接的mysql数据库是阿里云的数据库，而阿里云的数据库版本是5.6的。而测试环境的mysql是自己安装的5.7。因此在开发过程中有小伙伴不注意写了有关group by的sql语句。在开发环境中运行是正常的，而到了测试环境中就发现了异常。

原因分析：MySQL5.7版本默认设置了 mysql sql_mode = only_full_group_by 属性，导致报错。

> 其中ONLY_FULL_GROUP_BY就是造成这个错误的罪魁祸首了,对于group by聚合操作,如果在select中的列没有在group by中出现,那么这个SQL是不合法的,因为列不在group by从句中,所以设置了sql_mode=only_full_group_by的数据库,在使用group by时就会报错。
>
> 测试环境下载安装的是最新版的mysql5.7.x版本，默认是开启了 only_full_group_by 模式的，但开启这个模式后，原先的 group by 语句就报错，然后又把它移除了。
>
> 一旦开启 only_full_group_by ，感觉，group by 将变成和 distinct 一样，只能获取受到其影响的字段信息，无法和其他未受其影响的字段共存，这样，group by 的功能将变得十分狭窄了
>
> only_full_group_by 模式开启比较好。因为在 mysql 中有一个函数： any_value(field) 允许，非分组字段的出现（和关闭 only_full_group_by 模式有相同效果）。

**1、查看sql_mode**

```mysql
SELECT @@sql_mode;
```

查询出来的值为：

```mysql
ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

**2、去掉ONLY_FULL_GROUP_BY，重新设置值。**

```mysq
SET @@global.sql_mode ='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
```

**3、上面是改变了全局sql_mode，对于新建的数据库有效。对于已存在的数据库，则需要在对应的数据下执行**

```mysq
SET sql_mode ='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
```

### 以上方法mysql数据库重启后依然无效，下列方式重启后依然生效

**找到MySQL的配置文件,在linux系统上/etc/my.cnf文件,查询sql_mode字段,我并没有在配置文件中找到这个关键字,所以我手动添加进去:**

```
sql_mode = STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

**需要注意的一点是一定要添加在[mysqld]配置内,这样添加完后重启mysql才会生效,退出数据库:exit,重启命令:**

```shell
service mysqld restart
```

**刷新页面报错信息消失成功解决,再次连接上数据库查看sql_mode配置select @@sql_mode:**

```mysql
STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

**成功**

