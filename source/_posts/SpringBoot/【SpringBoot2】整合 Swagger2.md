---
title: SpringBoot2 整合 Swagger2
tags:
  - SpringBoot
  - Swagger2
categories:
  - SpringBoot
---

# SpringBoot2 整合 Swagger2

**SpringBoot整合三板斧**

### 第一步、引入pom

```pom
<dependency>
  <groupId>com.spring4all</groupId>
  <artifactId>swagger-spring-boot-starter</artifactId>
  <version>1.9.0.RELEASE</version>
</dependency>
<dependency>
  <groupId>com.github.xiaoymin</groupId>
  <artifactId>swagger-bootstrap-ui</artifactId>
  <version>1.9.6</version>
</dependency>

<dependency>
  <groupId>io.swagger</groupId>
  <artifactId>swagger-annotations</artifactId>
  <version>1.5.22</version>
</dependency>
<dependency>
  <groupId>io.swagger</groupId>
  <artifactId>swagger-models</artifactId>
  <version>1.5.22</version>
</dependency>
```

**`swagger-spring-boot-starter`该项目主要利用Spring Boot的自动化配置特性来实现快速的将swagger2引入spring boot应用来生成API文档，简化原生使用swagger2的整合代码。**

**`swagger-bootstrap-ui`是`springfox-swagger`的增强UI实现，为Java开发者在使用Swagger的时候，能拥有一份简洁、强大的接口文档体验**

**`swagger-annotations`,`swagger-models`是因为`springfox-swagger2`包里有`swagger-models-1.5.20.jar`报错。所以替换成1.5.22版本**

```java
java.lang.NumberFormatException: For input string: ""
	at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
	at java.lang.Long.parseLong(Long.java:601)
	at java.lang.Long.valueOf(Long.java:803)
	at io.swagger.models.parameters.AbstractSerializableParameter.getExample(AbstractSerializableParameter.java:412)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at......
```

**看下1.5.20版本里AbstractSerializableParameter.java源码：**

```java
public Object getExample() {
    if (this.example == null) {
        return null;
    } else {
        try {
            if ("integer".equals(this.type)) {
                return Long.valueOf(this.example);
            }
        
            if ("number".equals(this.type)) {
                return Double.valueOf(this.example);
            }

            if ("boolean".equals(this.type) && ("true".equalsIgnoreCase(this.example) || "false".equalsIgnoreCase(this.defaultValue))) {
                return Boolean.valueOf(this.example);
            }
        } catch (NumberFormatException var2) {
            LOGGER.warn(String.format("Illegal DefaultValue %s for parameter type %s", this.defaultValue, this.type), var2);
        }

        return this.example;
    }
}
```

**这里只判断了this.example == null才返回null，其余会去进行转换，而空字符串也会进行转换，导致格式抛出格式化转换异常.再来看下1.5.22版本里AbstractSerializableParameter.java源码：**

```java
public Object getExample() {
    if (this.example != null && !this.example.isEmpty()) {
        try {
            if ("integer".equals(this.type)) {
                return Long.valueOf(this.example);
            }

            if ("number".equals(this.type)) {
                return Double.valueOf(this.example);
            }

            if ("boolean".equals(this.type) && ("true".equalsIgnoreCase(this.example) || "false".equalsIgnoreCase(this.defaultValue))) {
                return Boolean.valueOf(this.example);
            }
        } catch (NumberFormatException var2) {
            LOGGER.warn(String.format("Illegal DefaultValue %s for parameter type %s", this.defaultValue, this.type), var2);
        }

        return this.example;
    } else {
        return this.example;
    } 
}
```

**对example同时进行了null和空值的判断，官方也发现了自己的这个问题，我们进行相应的替换即可**

### 第二部、配置

**`swagger-spring-boot-starter`相关配置信息可参考如下地址:**

- 源码地址
  - GitHub：https://github.com/dyc87112/spring-boot-starter-swagger
  - 码云：https://gitee.com/didispace/spring-boot-starter-swagger
- 使用样例：https://github.com/dyc87112/swagger-starter-demo
- 博客：[http://blog.didispace.com](http://blog.didispace.com/)
- 社区：[http://www.spring4all.com](http://www.spring4all.com/)

**`swagger-bootstrap-ui`相关配置信息可参考如下地址:**

官方地址：https://doc.xiaominfo.com/guide/

> `swagger-bootstrap-ui`目前已改名了`knife4j-spring-boot-starter`
>
> 项目正式更名为**knife4j**,取名knife4j是希望她能像一把匕首一样小巧,轻量,并且功能强悍,更名也是希望把她做成一个为Swagger接口文档服务的通用性解决方案,不仅仅只是专注于前端Ui前端.
>
> swagger-bootstrap-ui的所有特性都会集中在`knife4j-spring-ui`包中,并且后续也会满足开发者更多的个性化需求.

```yaml
swagger:
  version: 1.0v # 版本号
  authorization: # 全局参数
    name: Authorization # 鉴权策略ID，对应 SecurityReferences ID
    type: ApiKey # 鉴权策略，可选 ApiKey | BasicAuth | None，默认ApiKey
    key-name: X-Token # 鉴权传递的Header参数
  #    auth-regex: ^.*$ # 需要开启鉴权URL的正则, 默认^.*$匹配所有URL
  ui-config: # 排序规则
    operations-sorter: method # 按方法定义顺序排序
    tags-sorter: alpha # 按字母表排序
  docket: # 分组配置
    common:
      base-package: com.xxxx.a
      description: API接口文档
      title: xxx接口
      contact:
        name: xxx
        url: https://cn.bing.com/
    hq:
      base-package: com.xxxx.b
      description: API接口文档
      title: xxx接口
      contact:
        name: xxx
        url: https://zc.happyloves.cn:4443/wordpress/
    shop:
      base-package: com.xxxx.c
      description: API接口文档
      title: xxx接口
      contact:
        name: xxx
        url: https://zc.happyloves.cn
```

### 第三步、注解

```java
@EnableSwagger2Doc // 启用Swagger2
@EnableSwaggerBootstrapUI //启用swagger-bootstrap-ui
@SpringBootApplication
public class WebApplication {
    public static void main(String[] args) {
        SpringApplication.run(WebApplication.class, args);
    }
}
```

**编写代码**

```java
@Api(value = "DemoOne-DemoOne服务~~~~~~~~", tags = {"1-DemoOne-DemoOne服务"})
@Slf4j
@Validated
@RestController
@RequestMapping("/common/DemoOne")
public class DemoOneController {
    private final DemoOneService service;

    @Autowired
    public DemoOneController(DemoOneService service) {
        this.service = service;
    }

    //=====================================================================================DELETE=====================================================================================
    @ApiOperation(value = "根据主键ID删除", notes = "根据主键ID删除~~~~~~~~~~~~~")
    @DeleteMapping("/{id}")
    public ApiMessage deleteById(@PathVariable @Min(1) int id) throws Exception {
        return service.deleteById(id);
    }

    //=====================================================================================GET========================================================================================

    @ApiOperation(value = "获取所有数据", notes = "获取所有数据~~~~~~~~~~~~~")
    @GetMapping("/")
    public ApiMessage<List<DemoOneResponse>> getAllList() {
        return service.getAllList();
    }

    @ApiOperation(value = "根据主键ID获取数据", notes = "根据主键ID获取数据~~~~~~~~~~~~~")
    @ApiImplicitParams(value = {
            @ApiImplicitParam(name = "id", required = true, value = "主键ID", paramType = "path", dataType = "string"),
    })
    @GetMapping("/{id}/{name}")
    public ApiMessage<DemoOneResponse> getById(@PathVariable @Min(1) int id, @PathVariable @AssertFalse boolean name) {
        return service.getById(id);
    }

    //=====================================================================================POST=======================================================================================
    @ApiOperation(value = "新增DemoOne数据", notes = "新增DemoOne数据~~~~~~~~~~~~~")
    @PostMapping("/")
    public ApiMessage<DemoOneResponse> save(@RequestBody @Valid DemoOneRequest parameter) {
        return service.addDemoOne(parameter);
    }

    //=====================================================================================PUT========================================================================================
    @ApiOperation(value = "更新DemoOne数据", notes = "更新DemoOne数据~~~~~~~~~~~~~")
    @PutMapping("/")
    public ApiMessage<DemoOneResponse> update(@RequestBody @Valid DemoOneRequest parameter) {
        return service.update(parameter);
    }
```



#### 大功告成！！！启动访问如下地址：

**Swagger2地址：**

http://${ip地址}:${端口}/swagger-ui.html

**swagger-bootstrap-ui地址：**

http://${ip地址}:${端口}/doc.html
