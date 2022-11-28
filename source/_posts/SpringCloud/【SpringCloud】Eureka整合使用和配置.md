---
title: 【SpringCloud】Eureka整合使用和配置
tags:
  - SpringCloud
  - Eureka
  - 注册中心
categories:
  - SpringCloud
---
**遵循SpringBoot三板斧**
### 服务端
**第一步加依赖**
```pom
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```
**第二步加注解**
```java
//在启动类上加注解
@EnableEurekaServer
@SpringBootApplication
public class EurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}
```
**第三步写配置**
```yml
spring:
  application:
    name: eureka
# 详见EurekaServerConfigBean，需要注意与Client和Instance在client的jar包不同，Server是在server的jar包。
# eureka的各项配置可见EurekaXXXConfigBean。
eureka:
  datacenter: cloud           # 修改Eureka监控页面的System Status Data center
  environment: test            # 修改Eureka监控页面的System Status Environment
  instance:
    hostname: localhost
    prefer-ip-address: true
    leaseRenewalIntervalInSeconds:  5 # 心跳间隔，5秒
    leaseExpirationDurationInSeconds: 10  # 没有心跳的淘汰时间，10秒
    instance-id: ${spring.application.name}:${spring.cloud.client.ip-address}:${spring.application.instance_id:${server.port}} #SpringCloud 2.0 已经改成 ${spring.cloud.client.ip-address} 了，于是修改
  client:
    healthcheck:
      enabled: true
    # 默认情况下，eureka server同时也是eureka client，用于相互注册形成高可用eureka服务。
    # 单点时，如果registerWithEureka配置为true，则eureka server会报错Cannot execute request on any known server
    registerWithEureka: false # 是否注册到eureka服务，默认为true，当前已为eureka server，且单点eureka，故配置为false
    fetchRegistry: false # eureka之间如果网络不稳定，客户端一般也会缓存了注册列表，因此eureka服务可以不缓存，我觉得更能确保eureka之间的一致。
    serviceUrl:
      # registerWithEureka关闭后，defaultZone没有配置的必要。如果打开，即使配置为本机一样报错。
      # 也就是说defaultZone任何时候都没有配置为localhost的必要。这点上John的配置更好，永超和周立包括志朋的配置有点多余。
      # 但是周立说的对，这个属性默认配置是http://localhost:8761/eureka，也就是当你没有用户名密码安全认证时，本机调试时，客户端可以不配置，
      # 但对于server来说，这个默认没有什么作用。对于client来说，也只有调试的时候有点作用。
      # 但有一点很奇怪，既然默认了8761端口，为什么eureka server的默认端口要用8080而不是8761呢？
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/ #应用的主机名称
  #      defaultZone: http://${security.user.name}:${security.user.password}@localhost:${server.port}/eureka # 本配置应删除。
  server:
    # 自我保护机制，默认true。打开后，心跳失败在15分钟内低于85%(renewalPercentThreshold)的服务，也不进行剔除。
    # 关闭后，主页提示：RENEWALS ARE LESSER THAN THE THRESHOLD. THE SELF PRESERVATION MODE IS TURNED OFF.
    # THIS MAY NOT PROTECT INSTANCE EXPIRY IN CASE OF NETWORK/OTHER PROBLEMS.
    enableSelfPreservation: true # 本地调试时可fasle关闭。但生产建议打开，可防止因网络不稳定等原因导致误剔除服务。
    renewalPercentThreshold: 0.85 # 默认85%
    # 在服务器接收请求之前等待的初始时间，默认等待5min（John Carnell）
    waitTimeInMsWhenSyncEmpty: 5 # John说开发时最好注释此配置，服务注册需要3次心跳，每次10s，也就是30s才能显示在eureka。但是为什么我这里马上就显示呢？
    # eureka server刷新readCacheMap的时间，注意，client读取的是readCacheMap，这个时间决定了多久会把readWriteCacheMap的缓存更新到readCacheMap上
    # 默认30秒，eclipse提示默认0应该是错误的，源代码中responseCacheUpdateIntervalMs = 30 * 1000。
    response-cache-update-interval-ms: 3000 # 网上很多专家的博客错误写成responseCacheUpdateInvervalMs，请注意。这里配置为3秒。
    # eureka server缓存readWriteCacheMap失效时间，这个只有在这个时间过去后缓存才会失效，失效前不会更新，
    # 过期后从registry重新读取注册服务信息，registry是一个ConcurrentHashMap。
    # 由于启用了evict其实就用不太上改这个配置了，默认180s
    responseCacheAutoExpirationInSeconds: 180
    # 启用主动失效，并且每次主动失效检测间隔为3s。源码evictionIntervalTimerInMs = 60 * 1000，默认一分钟。
    # 需要注意的是该配置会打印INFO日志，增加info日志量，修改后从每60秒打印一次变成3秒打印一次。
    evictionIntervalTimerInMs: 3000 # 注意不要写成EvictionIntervalTimerInMs，yml大小写敏感。
```
如果是多实例高可用修改下列配置
```yml
eureka:
  client:
    registerWithEureka: true # 是否注册到eureka服务
    serviceUrl:
      defaultZone: http://peer2:1112/eureka/,http://peer3:1112/eureka/ #应用的主机名称
```
### 客户端
**第一步加依赖**
```pom
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```
**第二步加注解**
```java
//@EnableEurekaClient
@EnableDiscoveryClient
@SpringBootApplication
public class ClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(ClientApplication.class, args);
    }
}
```
**第三步写配置**
```yml
eureka:
  instance:
#    ip-address: #指定ip地址
    # 是否以IP注册到Eureka Server上，如果false则不是IP而是服务器名称
    # 但我设置了false，eureka主页仍显示192.168.100.16:client-microservice:8010
    preferIpAddress: true # 将IP注册到Eureka Server上，而如果不配置就是机器的主机名。默认false。应该始终设置为true。如果基于Docker等容器的部署，容器会生成一个随机的主机名，此时DNS不存在该名，无法解析 - John Carnell
    # 实例名。SpringCloud体系里的，服务实体向eureka注册时，注册名默认是“IP名:应用名:应用端口名”${spring.application.name}:${spring.cloud.client.ip-address}:${spring.application.instance_id:${random.int}}
    # 如果服务名，ip，端口都一致的话，eureka只显示一个服务
    instance-id: ${spring.cloud.client.hostname}:${spring.application.name}:${spring.cloud.client.ip-address}:${spring.application.instance_id:${random.int[1,9]}}-@project.version@
    # 服务续约的两个重要属性
    leaseRenewalIntervalInSeconds: 30 # 服务续约间隔时间。默认每隔30秒，客户端会向服务端发送心跳。见DiscoveryClient.initScheduledTasks
    leaseExpirationDurationInSeconds: 90 # 服务失效时间。缺省为90秒服务端接收不到客户端的心跳，则剔除该客户端服务实例。
    # 端点配置。若配置了context-path，actuator的监控端点会增加前缀，此时eureka也需要相应增加
    #status-page-url-path: ${server.servlet.context-path}/actuator/info
    #health-check-url-path: ${server.servlet.context-path}/actuator/health

    #    Eureka 的元数据
    metadata-map:
      zc-data: Current services are goods services  # 不会影响客户端
      zone: ABD               # Eureka可以理解的元数据，可以影响客户端
#    appname: AAAAA            # 填坑 Swagger：配置和spring.application.name 冲突
  client:
    # eureka服务的位置，如配置错误，则：Cannot execute request on any known server
    # 详见：com.netflix.discovery.endpoint.EndpointUtils
    service-url:
      defaultZone: http://localhost:8761/eureka/ #应用的主机名称
    # 是否启用eureka客户端。默认true
    enabled: true # 本地调试时，若不想启动eureka，可配置false即可，而不需要注释掉@EnableDiscoveryClient这么麻烦。感谢永超，从他的书知道这个属性。
    # 支持registerWithEureka(John、周立)和register-with-eureka(翟永超)两种写法，eclipse的STS默认使用后者。
    # 基本所有配置使用横杠或者驼峰都可以，鼠标放在上面，eclipse都可以显示详细注解和默认值（如果有）。
    registerWithEureka: true # 默认true，因此也可省略。
    fetchRegistry: true # 默认true，此处可不配置。
    # 缓存清单更新时间，默认30秒。见EurekaClientConfigBean，其中DefaultEurekaClientConfig可不看（前者spring实现，后者Netflix实现）
    registry-fetch-interval-seconds: 30 # 如果想eureka server剔除服务后尽快在client体现，我觉得可缩短此时间。
    # 周立在Camden SR4（对应eureka-client.jar1.2.6）中说有该属性，但我在SR6（对应1.2.4）和SR4中都找不到；
    # 又查找了Brixton SR7（对应1.1.7，其实不光eureka-client，整个spring-cloud-netflix都是这个版本），也是没有。
    # 这是因为该属性IDE确实不能提示，但写法是正确的。作用是修改eureka的健康检查方式（心跳），改为用actuator，详见HealthCheckHandler HealthIndicator。
    # 周立写的不是太详细，可详见这博客：https://blog.csdn.net/xiao_jun_0820/article/details/77991963
    # 若配置healthcheck，需引入actuator。
    healthcheck:
      enabled: true # 我建议配置为true。心跳机制有个问题，如当客户端的数据库连接出现问题导致不可用时，心跳机制不能反映，但actuator的health可以。
```
最后可以通过`DiscoveryClient`对象，在日志中打印出服务实例的相关内容。
```java
@Slf4j
@RestController
public class TestController {
	@Autowired
    private DiscoveryClient discoveryClient;

    @GetMapping("/getDiscoveryClient")
    public List<ServiceInstance> getDiscoveryClient() {
        return discoveryClient.getInstances("server-1");//获取客户端实例服务
    }

    @GetMapping("/getServices")
    public List<String> getServices() {
        return discoveryClient.getServices();
    }
}
```
