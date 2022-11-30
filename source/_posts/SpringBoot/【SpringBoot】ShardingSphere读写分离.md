---
title: 【SpringBoot】ShardingSphere读写分离
tags:
  - SpringBoot
  - JPA
  - ShardingSphere
categories:
  - SpringBoot
---
# ShardingSphere读写分离
**第一步：添加依赖**
```pom
    <dependency>
        <groupId>org.apache.shardingsphere</groupId>
        <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
        <version>{last.version}</version>
    </dependency>
```
**第一步：配置信息**
```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: update
      naming:
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
    show-sql: true
  shardingsphere:
    datasource:
      names: master,slave0 # 配置真实数据源
      master:
        type: com.zaxxer.hikari.HikariDataSource # 连接池配置
        driver-class-name: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://114.116.69.230:13036/sharding_db_1?useUnicode=true&characterEncoding=utf8&autoReconnect=true&useSSL=false&allowMultiQueries=true&useAffectedRows=true&serverTimezone=Asia/Shanghai
        username: fyzq_scm_test
        password: r7XNMpwfHffT44Lz
        hikari:
          minimum-idle: 5           # 最小连接数
          maximum-pool-size: 15     # 最大连接数
          auto-commit: true         # 此属性控制从池返回的连接的默认自动提交行为,默认值：true
          idle-timeout: 30000       # 连接允许在池中闲置的最长时间，默认600000（10分钟），单位ms
          pool-name: UserHikariCP   # 连接池名称
          max-lifetime: 1800000     # 此属性控制池中连接的最长生命周期，值0表示无限生命周期，默认1800000即30分钟，单位ms
          connection-timeout: 30000 # 数据库连接超时时间,默认30秒，即30000，单位ms
          connection-test-query: SELECT 1
      slave0:
        type: com.zaxxer.hikari.HikariDataSource # 连接池配置
        driver-class-name: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://114.116.69.230:13036/sharding_db_2?useUnicode=true&characterEncoding=utf8&autoReconnect=true&useSSL=false&allowMultiQueries=true&useAffectedRows=true&serverTimezone=Asia/Shanghai
        username: fyzq_scm_test
        password: r7XNMpwfHffT44Lz
        hikari:
          minimum-idle: 5           # 最小连接数
          maximum-pool-size: 15     # 最大连接数
          auto-commit: true         # 此属性控制从池返回的连接的默认自动提交行为,默认值：true
          idle-timeout: 30000       # 连接允许在池中闲置的最长时间，默认600000（10分钟），单位ms
          pool-name: UserHikariCP   # 连接池名称
          max-lifetime: 1800000     # 此属性控制池中连接的最长生命周期，值0表示无限生命周期，默认1800000即30分钟，单位ms
          connection-timeout: 30000 # 数据库连接超时时间,默认30秒，即30000，单位ms
          connection-test-query: SELECT 1
    masterslave: #读写配置
      name: ms
      # 从库的读取规则为round_robin（轮询策略），除了轮询策略，还有支持random（随机策略）
      load-balance-algorithm-type: round_robin
      master-data-source-name: master
      slave-data-source-names: slave0
    props: #打印sql
      sql:
        show: true
```

**编写代码:**
>实体类
```java
/**
 * @author zc
 * @date 2020/9/17 17:22
 */
@Data
@Entity
@Table(name = "account")
public class Account implements Serializable {

    private static final long serialVersionUID = -1132915818521200568L;
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    private String username;

    private String password;

    private String name;
    private int age;
    private String no;
    private String phone;
}
```
>DAO
```java
/**
 * @author zc
 * @date 2020/9/17 17:25
 */
public interface AccountRepository extends JpaRepository<Account, Integer> {
}

```
>Controller
```java
/**
 * @author zc
 * @date 2020/9/17 17:27
 */
@RestController
@RequestMapping("/account")
public class AccountController {

    @Autowired
    AccountRepository accountRepository;

    @PostMapping("/save")
    public void save(@RequestBody Account account) {
        accountRepository.save(account);
    }

    @GetMapping("/getAll")
    public List<Account> getAll() {
        return accountRepository.findAll();
    }

    @PostMapping("/")
    @Transactional(rollbackFor = Exception.class)
    public Account getAccount(@RequestBody Account account) throws Exception {
        List<Account> all = accountRepository.findAll();
        System.out.println("查询所有:" + all);
        accountRepository.save(account);
        if (account.getAge() < 0) {
            System.out.println("异常信息");
            throw new Exception("年龄小于0");
        } else {
            return account;
        }
    }
}
```
