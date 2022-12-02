---
title: 【SpringBoot】整合JPA、H2DB
tags:
  - SpringBoot
  - JPA
  - H2DB
categories:
  - SpringBoot
---
# 【SpringBoot】整合JPA、H2DB

JPA作为ORM框架一直是我非常喜欢的框架之一。
H2DB一直都是我作用测试用的一款内存数据库。其实也是可以存储在本地文件中的。

好了，废话不多说，咱们开始整合起来吧~

> 遵循SpringBoot整合策略的三板斧

**第一步：添加依赖**

```pom
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
  <groupId>com.h2database</groupId>
  <artifactId>h2</artifactId>
  <scope>runtime</scope>
</dependency>
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <optional>true</optional>
</dependency>
```

**第二部：添加配置**

> 如果想将H2DB数据持久化，那么`spring.datasource.url=jdbc:h2:file:${文件路径}`这样配置。
>
> 这样下次启动后，原先保存操作的数据，都还在。

```yaml
server:
  port: 8001
spring:
  application:
    name: H2DB
  datasource:
    driver-class-name: org.h2.Driver    #驱动
    url: jdbc:h2:mem:jpa                #h2内存数据库 内存模式连接配置 库名: jpa，jdbc:h2:~/test
#    url: jdbc:h2:file:/Users/zc/Documents/work spaces/happyloves/happyloves/h2db/src/main/resources/db   #将数据库持久化
    username: shuma                     #默认不填写的用户名是 sa
    password: shuma123                  #默认没有密码
  h2:
    console:
      enabled: true             #开启console 访问 默认false
      settings:
        trace: true             #开启h2 console 跟踪 方便调试  默认 false
        web-allow-others: true  #允许console 远程访问 默认false
      path: /h2db         #h2 访问路径上下文，启动后访问：http://{ip}:{port}/h2db。可以打开H2DB数据库的控制面板
  jpa:
    database: H2
    hibernate:
      #      create 启动时删数据库中的表，然后创建，退出时不删除数据表
      #      create-drop 启动时删数据库中的表，然后创建，退出时删除数据表 如果表不存在报错
      #      update 如果启动时表格式不一致则更新表，原有数据保留
      #      validate 项目启动表结构进行校验 如果不一致则报错
      ddl-auto: update # 常用的都是update，不会影响大原有的数据结构和数据
      naming:
        #  Hibernate到5.1废除了
        #  strategy: org.hibernate.cfg.ImprovedNamingStrategy
        #在进行领域映射时,首字母小写，大写字母变为下划线加小写
        physical-strategy: org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy
        #不做修改，直接映射org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
        implicit-strategy: org.hibernate.boot.model.naming.ImplicitNamingStrategyComponentPathImpl
    show-sql: true
    open-in-view: true
```

**第三步：写代码加注解**

> 创建数据库映射的实体类

```java
@Data//lombok插件的注解
@Entity//JPA实体类注解
@Table(name = "account")//JPA表映射的注解
public class Account {
    @Id//主键ID注解
    @GeneratedValue(strategy = GenerationType.IDENTITY)//主键ID唯一，一版数据库是Mysql时，主键ID自增时这样设置
    private Integer id;

    private String name;
    private String username;
    private String password;
}
```

> 创建实体类的JpaRepository接口

```java
//JpaRepository<实体类, 主键ID类型>
public interface AccountJPA extends JpaRepository<Account, Integer> {
}
```

> 接下来编写Controller API接口

```java
/**
 * @author zc
 * @date 2020/8/19 17:04
 * 采用RESTFUL接口风格
 */
@RestController
@RequestMapping("/account")
public class AccountController {
    @Autowired
    private AccountJPA accountJPA;


    /**
     * 保存账号信息
     *
     * @param account 请求值
     */
    @PostMapping("/")
    public void save(@RequestBody Account account) {
        accountJPA.save(account);
    }

    /**
     * 返回所有账号信息
     *
     * @return 返回值
     */
    @GetMapping("/")
    public List<Account> getAll() {
        return accountJPA.findAll();
    }
}
```

> 好了，启动项目工程，接下来我们来测试一下吧；可以通过Postman工具进行测试，还有Postwoman，这里我是通过IDEA的http测试工具：

```http
### 保存账号信息
POST http://localhost:8001/account/
Content-Type: application/json

{
  "name":"李四",
  "username":"bbb",
  "password":"bbb"
}


### 返回接口
POST http://localhost:8001/account/

HTTP/1.1 200 
Content-Length: 0
Date: Wed, 19 Aug 2020 09:34:47 GMT
Keep-Alive: timeout=60
Connection: keep-alive

<Response body is empty>

Response code: 200; Time: 39ms; Content length: 0 bytes
```

```http
### 返回所有账号信息
GET http://localhost:8001/account/

### 返回结果
GET http://localhost:8001/account/

HTTP/1.1 200 
Content-Type: application/json
Transfer-Encoding: chunked
Date: Wed, 19 Aug 2020 09:35:41 GMT
Keep-Alive: timeout=60
Connection: keep-alive

[
  {
    "id": 1,
    "name": "张三",
    "username": "aaa",
    "password": "aaa"
  },
  {
    "id": 2,
    "name": "李四",
    "username": "bbb",
    "password": "bbb"
  },
  {
    "id": 3,
    "name": "王五",
    "username": "ccc",
    "password": "ccc"
  }
]

Response code: 200; Time: 35ms; Content length: 166 bytes
```

