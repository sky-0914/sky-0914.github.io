---
title: Redis 分布式Session
tags:
  - Redis
categories:
  - 数据库
---
# Redis 分布式Session

**添加依赖**
```pom
    <!--    Spring Redis Session    -->
    <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-data-redis</artifactId>
    </dependency>
```

**配置文件**
```yaml
spring:
  redis:
    database: 8
    host: 114.116.69.230
    port: 16379
    password:
    timeout: 120000
  session:
    store-type: redis #指定redis实现spring session
    timeout: 60 # Session 过期时间，单位s
    redis:
      flush-mode: immediate # Sessions 刷新模式
      namespace: spring-session # Namespace for keys used to store sessions.
```

**编写代码**
```java
/**
 * Redis Session
 *
 * @author zc
 * @date 2020/9/15 19:12
 */
@RestController
@RequestMapping(value = "/redis")
public class SessionController {

    @GetMapping(value = "/first")
    public Map<String, Object> firstResp(HttpSession session, String name) {
        Map<String, Object> map = new HashMap<>(1);
        TestVO vo = new TestVO();
        vo.setName(name);
        session.setAttribute("account", vo);
        map.put("sessionId", session.getId());
        return map;
    }

    @GetMapping(value = "/sessions")
    public Object sessions(HttpSession session) {
        Map<String, Object> map = new HashMap<>(2);
        map.put("sessionId", session.getId());
        map.put("message", session.getAttribute("account"));
        return map;
    }
}

```
**拿Postman测试，启动两个服务，端口分别为8080，8081。在8080添加session，在8081获取session**
