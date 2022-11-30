---
title: 【SpringBoot】JPA在控制台打印带参sql语句
tags:
  - SpringBoot
  - JPA
  - 日志打印
categories:
  - SpringBoot
---
# 【SpringBoot】JPA在控制台打印带参sql语句

**第一步：引入依赖**

```xml
<dependency>
  <groupId>org.bgee.log4jdbc-log4j2</groupId>
  <artifactId>log4jdbc-log4j2-jdbc4.1</artifactId>
  <version>1.16</version>
</dependency>
```

**第二部：新建一个log4jdbc.log4j2.properties**

```properties
# If you use SLF4J. First, you need to tell log4jdbc-log4j2 that you want to use the SLF4J logger
log4jdbc.spylogdelegator.name=net.sf.log4jdbc.log.slf4j.Slf4jSpyLogDelegator
```

**第三步：修改配置**

```yaml
spring:
  application:
    name: H2DB
  datasource:
    #    driver-class-name: org.h2.Driver    #驱动
    #    url: jdbc:h2:mem:jpa                # h2 内存数据库 内存模式连接配置 库名: jpa，jdbc:h2:~/test
    driver-class-name: net.sf.log4jdbc.sql.jdbcapi.DriverSpy
    url: jdbc:log4jdbc:h2:mem:jpa
    # mysql url
    # 
```

> 将`driver-class-name`修改成`net.sf.log4jdbc.sql.jdbcapi.DriverSpy`
>
> 将`url`在`jdbc`后面添加个`log4jdbc`
>
> 比如将`url: jdbc:mysql://xxxxx:3306/dbname?xxxx`改成：`url: jdbc:log4jdbc:mysql://xxxxx:3306/dbname?xxxx`

**第四步：添加配置**

```yaml
# 关闭多余的日志打印
logging:
  level:
    jdbc.connection: off
    jdbc.resultset: off
    jdbc.resultsettable: INFO # 返回结果值
    jdbc.audit: off
    jdbc.sqltiming: INFO #sql
    jdbc.sqlonly: off
```

