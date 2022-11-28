---
title: 【SpringCloud】Feign组成和配置
tags:
  - SpringCloud
  - Feign
categories:
  - SpringCloud
---
### Feign的组成
| 接口  | 作用  | 默认值  |
| ------------ | ------------ | ------------ |
| `Feign.Builder`  | Feign的入口  | `Feign.Builder`  |
| `Client`  | Feign底层用什么去请求  | **和Ribbon配合时：**`LoadBalancerFeignClient`<br> **不和Ribbon配合时：**`Fgien.Client.Default` |
| `Contract`  | 契约，注解支持  | `SpringMVCContract`  |
| `Encoder`  | 解码器，用于将独享转换成HTTP请求消息体  | `SpringEncoder`  |
| `Decoder`  | 编码器，将相应消息体转成对象  | `ResponseEntityDecoder`  |
| `Logger`  | 日志管理器  | `Slf4jLogger`  |
| `RequestInterceptor`  | 用于为每个请求添加通用逻辑（拦截器，例子：比如想给每个请求都带上heared）  | 无  |

### Feign的日记级别
| 日志级别  | 打印内容  |
| ------------ | ------------ |
| NONE（默认）  | 不记录任何日志  |
| BASIC  |仅记录请求方法，URL，响应状态代码以及执行时间（适合生产环境）   |
| HEADERS  | 记录BASIC级别的基础上，记录请求和响应的header  |
| FULL  | 记录请求和弦ineader，body和元数据  |

### 首先如何整合Feign
**遵循SpringBoot的三板斧**
**第一步：加依赖**
```pom
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```
**第二步：写注解**
```java
@EnableFeignClients //在启动类上加
```
**第三步：写配置**
```yml
#没有配置
```
****
### 如何给Feign添加日志级别

#### 细粒度
**方式一：代码实现**
第一步：添加Feign配置类，可以添加在主类下，但是不用添加`@Configuration`。如果添加了`@Configuration`而且又放在了主类之下，那么就会所有Feign客户端实例共享，同Ribbon配置类一样父子上下文加载冲突；如果一定添加`@Configuration`，就放在主类加载之外的包。建议还是不用加`@Configuration`。
```java
public class FeignConfig {
    @Bean
    public Logger.Level Logger() {
        return Logger.Level.FULL;
    }
}
```
第二步：给`@FeignClient`添加配置类
```java
//@FeignClient configuration = GoodsFeignConfig.class 细粒度配置，指定配置类
@FeignClient(name = "goods", configuration = FeignConfig.class)
```
第四步：写配置
```yml
logging:
  level:
    com.xxx.xxx.FeignAPI: DEBUG #需要将FeignClient接口全路径写上# 开启日志 格式为logging.level.+Feign客户端路径
```
**方式二：配置属性实现**
```yml
feign:
  client:
    config:
      #想要调用的微服务名称
      server-1:
        loggerLevel: FULL
```
#### 全局配置
**方式一：代码实现**
~~添加了`@Configuration`放在了主类之下，那么就会所有Feign客户端实例共享，同Ribbon配置类一样父子上下文加载冲突；让父子上下文ComponentScan重叠（强烈不建议）~~
**唯一正确方式**
```java
//在启动类上为@EnableFeignClients注解添加defaultConfiguration配置
@EnableFeignClients(defaultConfiguration = FeignConfig.class)
```
**方式二：配置属性实现**
```yml
feign:
  client:
    config:
      #将调用的微服务名称改成default就配置成全局的了
      default:
        loggerLevel: FULL
```

### Feign支持的配置项
#### 代码方式

| 配置项  | 作用  |
| ------------ | ------------ |
| `Logger.Level`  | 指定日志级别  |
| `Retryer`  | 指定重试策略  |
| `ErrorDecoder`  | 指定错误解码器  |
| `Request.Options`  | 超时时间  |
| `Collection<RequestInterceptor>`  | 拦截器  |
| `SetterFactory`  | 用于设置Hystrix的配置属性，Fgien整合Hystrix才会用  |

#### 配置属性
```yml
feign:
  client:
    config:
      feignName:
        connectTimeout: 5000  # 相当于Request.Optionsn 连接超时时间
        readTimeout: 5000     # 相当于Request.Options 读取超时时间
        loggerLevel: full     # 配置Feign的日志级别，相当于代码配置方式中的Logger
        errorDecoder: com.example.SimpleErrorDecoder  # Feign的错误解码器，相当于代码配置方式中的ErrorDecoder
        retryer: com.example.SimpleRetryer  # 配置重试，相当于代码配置方式中的Retryer
        requestInterceptors: # 配置拦截器，相当于代码配置方式中的RequestInterceptor
          - com.example.FooRequestInterceptor
          - com.example.BarRequestInterceptor
		# 是否对404错误解码
        decode404: false
		encode: com.example.SimpleEncoder
		decoder: com.example.SimpleDecoder
		contract: com.example.SimpleContract
```

**Feign还支持对请求和响应进行GZIP压缩，以提高通信效率，配置方式如下：**
```yml
# 配置请求GZIP压缩
feign.compression.request.enabled=true
# 配置响应GZIP压缩
feign.compression.response.enabled=true
# 配置压缩支持的MIME TYPE
feign.compression.request.mime-types=text/xml,application/xml,application/json
# 配置压缩数据大小的下限
feign.compression.request.min-request-size=2048
```

### Ribbon配置 VS Feign配置
| 粒度  | Ribbon  | Feign  |
| ------------ | ------------ | ------------ |
| 代码局部  |` @RibbonClient(configuration=RibbonConfig.class)`，`RibbonConfig`类必须加`@Configuration`,且必须放在父上下文无法扫到的包下  | `@FeignClient(configuration=FeignConfig.class)`，`FeignConfig`类的`@Configuration`可以不加（可选），如果有，必须放在父上下文无法扫到的包下  |
| 代码全局  | `@RibbonClients(defaultConfigurtion=RibbonConfig.class)`  | `@EnableFeignClients(defaultConfiguration = FeignConfig.class)`<br>...  |
| 配置属性局部  | <clientName(服务名称)>.ribbon.NFLoadBalancerClassName<br>...  | feign.client.config.<clientName(服务名称)>.loggerLevel <br>...  |
| 配置属性全局  | 无 | feign.client.config.default.loggerLevel  |

### Feign 代码方式 VS 配置属性方式
| 配置方式  | 有点  | 缺点  |
| ------------ | ------------ | ------------ |
| 代码配置  | 基于代码，更加灵活  | 如果Feign的配置类加了`@Configuration`注解，需注意父子上下文，线上修改需要重打包，发布  |
| 属性配置  | 易上手<br> 配置更加直观<br> 线上修改无需重新打包，发布<br> **优先级更高**  | 极端场景下没有代码配置更加灵活  |

**优先级：细粒度属性配置 > 细粒度代码配置 > 全局属性配置 > 全局代码配置**









