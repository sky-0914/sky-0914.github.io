---
title: 【SpringBoot】整合ElasticSearch
tags:
  - SpringBoot
  - ElasticSearch
categories:
  - SpringBoot
---
# ElasticSearch

## SpringBoot 整合 ElasticSearch

### 第一步：添加依赖
> 此处需要注意下，ElasticSearch版本与`spring-boot-starter-data-elasticsearch`依赖包的版本兼容问题
```pom
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```
当然也可以根据当前ElasticSearch版本自定义选择依赖`jar`包的版本
```pom
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>${elasticsearch-version}</version>
</dependency>
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-client</artifactId>
    <version>${elasticsearch-version}</version>
</dependency>
<dependency>
    <groupId>org.elasticsearch</groupId>
    <artifactId>elasticsearch</artifactId>
    <version>${elasticsearch-version}</version>
</dependency>
```

### 第二部：编写配置
```yaml
spring:
  data:
    elasticsearch:
      repositories:
        enabled: true
      client:
        reactive:
          use-ssl: false
          endpoints: 127.0.0.1:9200
```

### 示例代码

Entity
```java
/**
 * @author zc
 * @date 2020/10/23 12:16
 */
@Data
@Document(indexName = "student")
public class Student implements Serializable {
    private static final long serialVersionUID = 4830083526353606264L;
    @Id
    private Long id;
    private String name;
    private int age;
    private String sex;
}
```

Repository:类似JPA 
```java
/**
 * @author zc
 * @date 2020/10/23 14:00
 */
public interface StudentRepository extends ElasticsearchRepository<Student, Long> {
}
```
Controller
```java
/**
 * @author zc
 * @date 2020/10/23 14:00
 */
@RestController
@RequestMapping("/student")
public class StudentController {

    @Autowired
    private StudentRepository repository;

    @PostMapping("/")
    public void save(@RequestBody Student student) {
        repository.save(student);
    }

    @GetMapping("/")
    public Iterable<Student> getAll() {
        return repository.findAll();
    }

}
```
**接下来用PostMan测试下**



