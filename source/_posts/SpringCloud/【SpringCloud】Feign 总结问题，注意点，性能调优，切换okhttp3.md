---
title: 【SpringCloud】Feign 总结问题，注意点，性能调优，切换okhttp3
tags:
  - SpringCloud
  - Feign
  - okhttp
categories:
  - SpringCloud
---
### Feign常见问题总结
**FeignClient接口如使用`@PathVariable` ，必须指定value属性**
```java
//在一些早期版本中， @PathVariable("id") 中的 "id" ，也就是value属性，必须指定，不能省略。
@FeignClient("microservice-provider-user")
public interface UserFeignClient {
  @RequestMapping(value = "/simple/{id}", method = RequestMethod.GET)
  public User findById(@PathVariable("id") Long id);
  ...
}
```
**Java代码自定义Feign Client的注意点与坑**
```java
@FeignClient(name = "microservice-provider-user", configuration = UserFeignConfig.class)
public interface UserFeignClient {
  @GetMapping("/users/{id}")
  User findById(@PathVariable("id") Long id);
}

/**
 * 该Feign Client的配置类，注意：
 * 1. 该类可以独立出去；
 * 2. 该类上也可添加@Configuration声明是一个配置类；
 * 配置类上也可添加@Configuration注解，声明这是一个配置类；
 * 但此时千万别将该放置在主应用程序上下文@ComponentScan所扫描的包中，
 * 否则，该配置将会被所有Feign Client共享，无法实现细粒度配置！
 * 个人建议：像我一样，不加@Configuration注解
 *
 * @author zhouli
 */
class UserFeignConfig {
  @Bean
  public Logger.Level logger() {
    return Logger.Level.FULL;
  }
}
```
- 配置类上也可添加@Configuraiton 注解，声明这是一个配置类；但此时千万别将该放置在主应用程序上下文@ComponentScan 所扫描的包中，否则，该配置将会被所有Feign Client共享（相当于变成了通用配置，其实本质还是Spring父子上下文扫描包重叠导致的问题），无法实现细粒度配置！
- **个人建议：**像我一样，不加@Configuration注解，省得进坑。
- **最佳实践：**尽量用配置属性自定义Feign的配置！！！

**@FeignClient 注解属性**
```java
//@FeignClient(name = "microservice-provider-user")
//在早期的Spring Cloud版本中，无需提供name属性，从Brixton版开始，@FeignClient必须提供name属性，否则应用将无法正常启动！
//另外，name、url等属性支持占位符。例如：
@FeignClient(name = "${feign.name}", url = "${feign.url}")
```
**类级别的@RequestMapping会被Spring MVC加载**
```java
@RequestMapping("/users")
@FeignClient(name = "microservice-user")
public class TestFeignClient {
    // ...
}
```
类上的`@RequestMapping` 注解也会被Spring MVC加载。该问题现已经被解决，早期的版本有两种解决方案：
**方案1：**不在类上加@RequestMapping 注解；
**方案2：**添加如下代码：
```java
@Configuration
@ConditionalOnClass({ Feign.class })
public class FeignMappingDefaultConfiguration {
    @Bean
    public WebMvcRegistrations feignWebRegistrations() {
        return new WebMvcRegistrationsAdapter() {
            @Override
            public RequestMappingHandlerMapping getRequestMappingHandlerMapping() {
                return new FeignFilterRequestMappingHandlerMapping();
            }
        };
    }

    private static class FeignFilterRequestMappingHandlerMapping extends RequestMappingHandlerMapping {
        @Override
        protected boolean isHandler(Class<?> beanType) {
            return super.isHandler(beanType) && !beanType.isInterface();
        }
    }
}
```
**首次请求失败**
Ribbon的饥饿加载(eager-load)模式

**如需产生Hystrix Stream监控信息，需要做一些额外操作**
Feign本身已经整合了Hystrix，可直接使用`@FeignClient(value = "microservice-provider-user", fallback = XXX.class)` 来指定fallback类，fallback类继承`@FeignClient`所标注的接口即可。

但是假设如需使用Hystrix Stream进行监控，默认情况下，访问http://IP:PORT/actuator/hystrix.stream 是会返回404，这是因为Feign虽然整合了Hystrix，但并没有整合Hystrix的监控。如何添加监控支持呢？需要以下几步：

第一步：添加依赖，示例：
```pom
<!-- 整合hystrix，其实feign中自带了hystrix，引入该依赖主要是为了使用其中的hystrix-metrics-event-stream，用于dashboard -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```
第二步：在启动类上添加@EnableCircuitBreaker 注解，示例：
```java
@SpringBootApplication
@EnableFeignClients
@EnableDiscoveryClient
@EnableCircuitBreaker
public class MovieFeignHystrixApplication {
  public static void main(String[] args) {
    SpringApplication.run(MovieFeignHystrixApplication.class, args);
  }
}
```
第三步：在application.yml中添加如下内容，暴露hystrix.stream端点：
```yml
management:
  endpoints:
    web:
      exposure:
        include: 'hystrix.stream'
```
这样，访问任意Feign Client接口的API后，再访问http://IP:PORT/actuator/hystrix.stream ，就会展示一大堆Hystrix监控数据了。

原文链接：http://www.itmuch.com/spring-cloud-sum/feign-problems/

#### Feign 上传文件
**加依赖**
```pom
<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form</artifactId>
	<version>3.0.3</version>
</dependency>
<dependency>
	<groupId>io.github.openfeign.form</groupId>
	<artifactId>feign-form-spring</artifactId>
	<version>3.0.3</version>
</dependency>
```
**编写Feign Client**
```java
@FeignClient(name = "ms-content-sample", configuration = UploadFeignClient.MultipartSupportConfig.class)
public interface UploadFeignClient {
    @RequestMapping(value = "/upload", method = RequestMethod.POST,
            produces = {MediaType.APPLICATION_JSON_UTF8_VALUE},
            consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    @ResponseBody
    String handleFileUpload(@RequestPart(value = "file") MultipartFile file);

    class MultipartSupportConfig {
        @Bean
        public Encoder feignFormEncoder() {
            return new SpringFormEncoder();
        }
    }
}
```
如代码所示，在这个Feign Client中，我们引用了配置类`MultipartSupportConfig` ，在`MultipartSupportConfig` 中，我们实例化了`SpringFormEncoder` 。这样这个Feign Client就能够上传啦。
**注意点**
-
```java
//RequestMapping注解中的produeces 、consumes 不能少；
@RequestMapping(value = "/upload", method = RequestMethod.POST,
            produces = {MediaType.APPLICATION_JSON_UTF8_VALUE},
            consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
```
- 接口定义中的注解`@RequestPart(value = "file")` 不能写成`@RequestParam(value = "file"）` 。
- 最好将Hystrix的超时时间设长一点，例如5秒，否则可能文件还没上传完，Hystrix就超时了，从而导致客户端侧的报错。

原文链接：http://www.itmuch.com/spring-cloud-sum/spring-cloud-feign-upload/

#### Feign实现Form表单提交
**添加依赖：**
```pom
<dependency>
  <groupId>io.github.openfeign.form</groupId>
  <artifactId>feign-form</artifactId>
  <version>3.2.2</version>
</dependency>
<dependency>
  <groupId>io.github.openfeign.form</groupId>
  <artifactId>feign-form-spring</artifactId>
  <version>3.2.2</version>
</dependency>
```
**Feign Client示例：**
```java
@FeignClient(name = "xxx", url = "http://www.itmuch.com/", configuration = TestFeignClient.FormSupportConfig.class)
public interface TestFeignClient {
    @PostMapping(value = "/test",
            consumes = {MediaType.APPLICATION_FORM_URLENCODED_VALUE},
            produces = {MediaType.APPLICATION_JSON_UTF8_VALUE}
            )
    void post(Map<String, ?> queryParam);

    class FormSupportConfig {
        @Autowired
        private ObjectFactory<HttpMessageConverters> messageConverters;
        // new一个form编码器，实现支持form表单提交
        @Bean
        public Encoder feignFormEncoder() {
            return new SpringFormEncoder(new SpringEncoder(messageConverters));
        }
        // 开启Feign的日志
        @Bean
        public Logger.Level logger() {
            return Logger.Level.FULL;
        }
    }
}
```
**调用示例：**
```java
@GetMapping("/user/{id}")
public User findById(@PathVariable Long id) {
  HashMap<String, String> param = Maps.newHashMap();
  param.put("username","zhangsan");
  param.put("password","pwd");
  this.testFeignClient.post(param);
  return new User();
}
```
**日志：**
```code
...[TestFeignClient#post] ---> POST http://www.baidu.com/test HTTP/1.1
...[TestFeignClient#post] Accept: application/json;charset=UTF-8
...[TestFeignClient#post] Content-Type: application/x-www-form-urlencoded; charset=UTF-8
...[TestFeignClient#post] Content-Length: 30
...[TestFeignClient#post]
...[TestFeignClient#post] password=pwd&username=zhangsan
...[TestFeignClient#post] ---> END HTTP (30-byte body)
```
由日志可知，此时Feign已能使用Form表单方式提交数据。

原文链接：http://www.itmuch.com/spring-cloud-sum/feign-form-params/

### Feign GET请求如何构造多参数
假设需请求的URL包含多个参数，例如http://microservice-provider-user/get?id=1&username=张三 ，该如何使用Feign构造呢？
我们知道，Spring Cloud为Feign添加了Spring MVC的注解支持，那么我们不妨按照Spring MVC的写法尝试一下：
```java
@FeignClient("microservice-provider-user")
public interface UserFeignClient {
  @RequestMapping(value = "/get", method = RequestMethod.GET)
  public User get0(User user);
}
```
然而，这种写法并不正确，控制台会输出类似如下的异常。
```code
feign.FeignException: status 405 reading UserFeignClient#get0(User); content:
{"timestamp":1482676142940,"status":405,"error":"Method Not Allowed","exception":"org.springframework.web.HttpRequestMethodNotSupportedException","message":"Request method 'POST' not supported","path":"/get"}
```
由异常可知，尽管我们指定了GET方法，Feign依然会使用POST方法发送请求。于是导致了异常。正确写法如下

**方法一[推荐]**
**注意:使用该方法无法使用Fegin的继承模式**
```java
@FeignClient("microservice-provider-user")
public interface UserFeignClient {
  @GetMapping("/get")
  public User get0(@SpringQueryMap User user);
}
```
**方法二[推荐]**
```java
@FeignClient(name = "microservice-provider-user")
public interface UserFeignClient {
  @RequestMapping(value = "/get", method = RequestMethod.GET)
  public User get1(@RequestParam("id") Long id, @RequestParam("username") String username);
}
```
这是最为直观的方式，URL有几个参数，Feign接口中的方法就有几个参数。使用@RequestParam注解指定请求的参数是什么。

**方法三[不推荐]**
多参数的URL也可使用Map来构建。当目标URL参数非常多的时候，可使用这种方式简化Feign接口的编写。
```java
@FeignClient(name = "microservice-provider-user")
public interface UserFeignClient {
  @RequestMapping(value = "/get", method = RequestMethod.GET)
  public User get2(@RequestParam Map<String, Object> map);
}
```
在调用时，可使用类似以下的代码。
```java
public User get(String username, String password) {
  HashMap<String, Object> map = Maps.newHashMap();
  map.put("id", "1");
  map.put("username", "张三");
  return this.userFeignClient.get2(map);
}
```
**注意：**这种方式不建议使用。主要是因为可读性不好，而且如果参数为空的时候会有一些问题，例如`map.put("username", null);` 会导致服务调用方（消费者服务）接收到的username是"" ，而不是null。

原文链接：http://www.itmuch.com/spring-cloud-sum/feign-multiple-params-2/

### 切换为 Okhttp3 提升 QPS 性能优化
**加依赖引入okhttp3**
```pom
<dependency>
	<groupId>io.github.openfeign</groupId>
	<artifactId>feign-okhttp</artifactId>
	<version>${version}</version>
</dependency>
```
**写配置**
```yml
feign:
  # feign启用hystrix，才能熔断、降级
  # hystrix:
  # enabled: true
  # 启用 okhttp 关闭默认 httpclient
  httpclient:
    enabled: false #关闭httpclient
	# 配置连接池
    max-connections: 200 #feign的最大连接数
    max-connections-per-route: 50 #fegin单个路径的最大连接数
  okhttp:
    enabled: true
  # 请求与响应的压缩以提高通信效率
  compression:
    request:
      enabled: true
      min-request-size: 2048
      mime-types: text/xml,application/xml,application/json
    response:
      enabled: true
```
**参数配置**
```java
/**
 * 配置 okhttp 与连接池
 * ConnectionPool 默认创建5个线程，保持5分钟长连接
 */
@Configuration
@ConditionalOnClass(Feign.class)
@AutoConfigureBefore(FeignAutoConfiguration.class) //SpringBoot自动配置
public class OkHttpConfig {

    // 默认老外留给你彩蛋中文乱码，加上它就 OK
    @Bean
    public Encoder encoder() {
        return new FormEncoder();
    }

    @Bean
    public okhttp3.OkHttpClient okHttpClient() {
        return new okhttp3.OkHttpClient.Builder()
                //设置连接超时
                .connectTimeout(10, TimeUnit.SECONDS)
                //设置读超时
                .readTimeout(10, TimeUnit.SECONDS)
                //设置写超时
                .writeTimeout(10, TimeUnit.SECONDS)
                //是否自动重连
                .retryOnConnectionFailure(true)
                .connectionPool(new ConnectionPool(10, 5L, TimeUnit.MINUTES))
                .build();
    }
}
```
原文链接：https://mp.weixin.qq.com/s/PAjXS9d6Sxa04pw1Lw2HXQ

