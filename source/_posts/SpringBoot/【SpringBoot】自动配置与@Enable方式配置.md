---
title: SpringBoot自动配置与@Enable方式配置
tags:
  - SpringBoot
  - 自动配置
categories:
  - SpringBoot
---
# SpringBoot自动配置与@Enable方式配置

## SpringBoot自动配置：

在`resources`目录下新建`META-INF`目录，创建`spring.factories`文件。

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=xx.xx.xx.TestAutoConfig
```

```java
/**
 * @author zc
 * @date 2020/12/22 00:53
 */
@Slf4j
@Configuration
public class TestAutoConfig {

    @Data
    public class Test {

    }

    @Bean
    public Test user() {
        log.error("=======");
        return new Test();
    }
}
```

此种方式就可以实现SpringBoot的自动配置加载。

## 通过`@Enable`注解方式实现自动配置

```java
/**
 * 启用注解：@EnableTest
 *
 * @author ZC
 * @date 2020/6/8-22:05
 */
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({TestAutoConfig.class})
public @interface EnableTest {
}
```

**不要在`spring.factories`文件里配置加载类，不然SpringBoot就自动加载配置了**

此时只需要在启动类上加上`@EnableTest`就可以实现加载配置类了

