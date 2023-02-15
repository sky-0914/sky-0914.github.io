---
title: 【SpringCloud】SpringCloud 集成 STC（腾讯北极星）
tags:
  - SpringCloud
  - 腾讯
  - 北极星
  - polaris
categories:
  - SpringCloud
---

## 服务注册与发现
[官方文档-注册中心](https://github.com/Tencent/spring-cloud-tencent/wiki/Spring-Cloud-Tencent-Discovery-%E4%BD%BF%E7%94%A8%E6%96%87%E6%A1%A3)

### 1. 第一步-引入依赖
#### 方式一： 只引入 spring-cloud-starter-tencent-polaris-discovery
```pom
<dependency>
    <groupId>com.tencent.cloud</groupId>
    <artifactId>spring-cloud-starter-tencent-polaris-discovery</artifactId>
</dependency>
```
### 方式二：通过 spring-cloud-starter-tencent-all 引入 sct 所有 starters
```pom
<dependency>
    <groupId>com.tencent.cloud</groupId>
    <artifactId>spring-cloud-starter-tencent-all</artifactId>
    <!-- 注意需要指定 type=pom-->
    <type>pom</type> 
</dependency>
```

**如果使用 Spring Cloud 2021 版本，还需要添加如下依赖：**
```pom
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

### 2.第二步-配置
在 bootstrap.yml 配置文件中加入以下配置内容
```yaml
spring:
  application:
    name: ${application.name}
  cloud:
    polaris:
      address: grpc://${修改为第一步部署的 Polaris 服务地址}:8091
      namespace: default
```

**完整的的配置列表如下**

| 配置项Key                                                    | 默认值                     | 是否必填 | 初始版本 | 配置项说明                                                   |
| ------------------------------------------------------------ | -------------------------- | -------- | -------- | ------------------------------------------------------------ |
| spring.cloud.polaris.address                                 | 无                         | 是       |          | Polaris 后端地址                                             |
| spring.cloud.polaris.namespace                               | default                    | 否       |          | 服务所在的命名空间名称                                       |
| spring.cloud.polaris.service                                 | ${spring.application.name} | 否       |          | 服务名称                                                     |
| spring.cloud.polaris.local-ip-address                        | 无                         | 否       |          | 注册的 IP 地址，默认情况下 Spring Boot 会自动获取 IP 地址无需指定 IP 地址，当需要自定义 IP 地址时才需要设置 |
| spring.cloud.polaris.discovery.enabled                       | true                       | 否       |          | 是否开启服务发现                                             |
| spring.cloud.polaris.discovery.register                      | true                       | 否       |          | 是否开启服务注册                                             |
| spring.cloud.polaris.discovery.instance-enabled              | true                       | 否       |          | 当前微服务实例是否可以被访问                                 |
| spring.cloud.polaris.discovery.health-check-url              | 无                         | 否       |          | 健康检查 url，向北极星服务端发送心跳前回调健康检查 url 判断当前服务是否健康，只有健康才会发送心跳 |
| spring.cloud.polaris.discovery.token                         | 无                         | 否       |          | 鉴权 Token                                                   |
| spring.cloud.polaris.discovery.version                       | null                       | 否       |          | 微服务版本                                                   |
| spring.cloud.polaris.discovery.protocol                      | null                       | 否       |          | 微服务协议类型                                               |
| spring.cloud.polaris.discovery.weight                        | 100                        | 否       |          | 微服务权重                                                   |
| spring.cloud.polaris.discovery.service-list-refresh-interval | 60000                      | 否       |          | 服务列表刷新间隔（毫秒）                                     |
| spring.cloud.polaris.discovery.heartbeat-interval            | 5000                       | 否       | 1.7.0    | 心跳间隔（毫秒）                                             |

### 3.第三步-服务调用

在 Spring Cloud 中可通过 `RestTemplate` 或者 `Feign` 发起服务调用。无论通过哪种方式，都需要引入 `spring-cloud-starter-tencent-polaris-discovery` 依赖，才能从北极星获取服务提供者的地址信息。

```
<dependency>
    <groupId>com.tencent.cloud</groupId>
    <artifactId>spring-cloud-starter-tencent-polaris-discovery</artifactId>
</dependency>
```

**RestTemplate**

只需要在实例化 `RestTemplate` 的地方加上 `@LoadBalanced` 注解即可。

```
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
	return new RestTemplate();
}
```

**Feign**

通过 `Feign` 框架调用，无需做任何适配工作，按照标准的 `Feign` 方式即可。

### 4.第四步-前往控制台查看

可以前往北极星服务端控制台查看实例信息，可以在控制台上对实例进行一系列管控操作，例如隔离实例，调整实例权重等

![img](https://user-images.githubusercontent.com/4991116/161026971-f57814fc-6e26-47ad-aa6d-16ff1e8d57ad.png)

## 配置中心

[官方文档-配置中心，如何在北极星控制台添加配置文件，拓展使用，北极星配置中心原理介绍](https://github.com/Tencent/spring-cloud-tencent/wiki/Spring-Cloud-Tencent-Config-%E4%BD%BF%E7%94%A8%E6%96%87%E6%A1%A3)

配置中心模块是 Spring Cloud Tencent 最核心的模块之一，实现了 Spring Cloud PropertySourceLocator SPI 接口(PolarisConfigFileLocator.java)。

在应用启动 Bootstrap 阶段，Spring Cloud 会调用 PolarisConfigFileLocator 从 Polaris 服务端获取配置文件并加载到 Spring 上下文里。通过 Spring Boot 标准的 @Value，@ConfigurationProperties 注解即可获取配置内容。

动态配置刷新能力，1.6.0 版本之前通过 Spring Cloud 标准的 @RefreshScope 机制实现。 1.6.0 版本之后，通过反射的方式实现。反射的方式不用重新构建 bean 对象，影响面更小、性能更优。
> 注意：
> 1. 由于 Spring Cloud PropertySourceLoader SPI 是在 Bootstrap 阶段调用，所以 Polaris Config 相关的配置内容（例如Polaris 服务地址）需要放在 bootstrap.yml 文件里，而不能放在 application.yml 文件里，否则会初始化失败。
> 2. 如果开发者原本的服务名（ spring.application.name ）配置在 application.yml 、application-*.yml 、application.properties 或 application-*.properties 内，使用 Spring Cloud Tencent Config 的功能时，需要将 spring.application.name 配置在 bootstrap.yml 内，或者在 bootstrap.yml 手动配置 spring.cloud.polaris.service 以指定文件分组名。

### 1. 第一步-引入依赖
#### 方式一：只引入 spring-cloud-starter-tencent-polaris-config
```pom
<dependency>
    <groupId>com.tencent.cloud</groupId>
    <artifactId>spring-cloud-starter-tencent-polaris-config</artifactId>
</dependency>
```
#### 方式二：通过 spring-cloud-starter-tencent-all 引入 sct 所有 starters
```pom
<dependency>
    <groupId>com.tencent.cloud</groupId>
    <artifactId>spring-cloud-starter-tencent-all</artifactId>
    <!-- 注意需要指定 type=pom-->
    <type>pom</type> 
</dependency>
```
**如果使用 Spring Cloud 2021 版本，还需要添加如下依赖：**
```pom
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```
### 2.第二步-配置
> 在您项目的 bootstrap.yml 配置文件中加入以下配置内容，注意一定要放在 bootstrap.yml 里。因为应用启动强依赖配置，所以 Spring Cloud Config 是在 Bootstrap 阶段加载配置文件，所以需要把北极星相关的配置放在 bootstrap.yml 里。</br> 
> bootstrap.yml 核心配置内容为配置北极星服务端地址以及注入的配置文件信息。

#### 1. 配置 Polaris 配置中心地址
如果您使用的北极星配置中心和注册中心是同一套北极星集群，则只需配置 spring.cloud.polaris.address 即可。

如果您部署了两套北极星集群，分别用于注册中心和配置中心，则 spring.cloud.polaris.address 用于指定注册中心集群的地址，spring.cloud.polaris.config.address 用于指定配置中心的地址。

如下所示：
```yml
spring:
  application:
    name: ${application.name}
  cloud:
    polaris:
      address: grpc://${修改为第一步部署的 Polaris 服务地址}:8091 # 必填
      namespace: default # 全局 namespace 参数
      config:
        address: grpc://${独立的配置中心}:8093 # 选填，只有在配置中心和注册中心是两个不同的地址时才需要配置
        auto-refresh: true # 选填，当配置发布后，动态刷新 Spring 上下文，默认值为 true
```
#### 2. 注入配置文件
我们推荐的最佳实践是在北极星管控端创建一个名为当前应用名（ ${spring.application.name}） 的配置分组，Spring Cloud Tencent Config 会自动注入当前应用名分组下的
> + application-${activeProfile}.properties 
> + application-${activeProfile}.yml 
> + application.properties 
> + application.yml 
> + bootstrap-${activeProfile}.properties 
> + bootstrap-${activeProfile}.yml 
> + bootstrap.properties 
> + bootstrap.yml
> 
> 优先级从上到下依次降低
> 
> **注意：是 yml 后缀，而不是 yaml**

自动注入以上配置文件符合 Spring Boot 的规范，能够满足绝大部分应用场景了。

只有当您需要注入额外自定义的配置文件时，才需要在 bootstrap.yml 里配置 spring.cloud.polaris.config.groups，如下所示：

```yml
spring:
  cloud:
    polaris:
      config:
        groups:
          - name: ${spring.application.name} # 选填，注入自定义配置的配置分组
            files: [ "config/application.properties", "config/bootstrap.yml" ] # 注入自定义配置文件列表，当 key 冲突时，排在前面的配置文件优先级高于后面
```

**完整的配置参数**

| 配置项Key                                                    | 起始版本 | 默认值                  | 是否必填 | 配置项说明                                                   |
| ------------------------------------------------------------ | -------- | ----------------------- | -------- | ------------------------------------------------------------ |
| spring.cloud.polaris.config.enabled                          |          | true                    | 否       | 是否开启配置模块                                             |
| spring.cloud.polaris.config.address                          |          | 无                      | 否       | 北极星服务端地址，可不填。当配置中心和注册中心为不同的地址时候才需要填写 |
| spring.cloud.polaris.config.port                             |          | 8093                    | 否       | 北极星配置中心的端口号，默认为 8093                          |
| spring.cloud.polaris.config.auto-refresh                     |          | true                    | 否       | 是否动态更新配置                                             |
| spring.cloud.polaris.config.groups                           |          | 无                      | 否       | 从北极星服务端获取自定义的配置文件                           |
| spring.cloud.polaris.config.connect-remote-server            |          | true                    | 否       | 是否连接到远程北极星配置中心服务端，当没有网络环境或者无北极星服务端时可关闭 |
| spring.cloud.polaris.config.refresh-type                     | 1.7.0    | refresh_context         | 否       | 动态刷新配置的实现方式。 `refresh_context` 为 Spring Cloud 默认的方式, `reflect` 为 SCT 实现的通过反射的机制实现的，优点是无需重建 Bean |
| spring.cloud.polaris.config.shutdown-if-connect-to-config-server-failed | 1.7.0    | true                    | 否       | 应用启动时检查配置中心的后端连接，如果连接配置中心失败，是否需要终止应用启动（true 为终止启动） |
| spring.cloud.polaris.config.data-source                      | 1.8.0    | polaris                 | 否       | 动态配置数据源类型，当前支持 polaris 和 local 两种模式，默认为 polaris。 - polaris: 从北极星服务端获取配置文件。 - local: 从本地磁盘读取配置文件 |
| spring.cloud.polaris.config.local-file-root-path             | 1.8.0    | ./polaris/backup/config | 否       | 当 `spring.cloud.polaris.config.data-source=local` 时，可设置本地磁盘读取配置文件根目录 |

### 3.第三步-代码里使用配置

#### 1. 通过 `@Value` 注入

```java
@Value("${name:张三}")
private int name;
```

#### 2. 通过 `@ConfigurationProperties` 注入

```java
@Data
@Component
@ConfigurationProperties(prefix = "teacher")
public class MyProperties {
	private String name;
	private int age;
}
```

### 4.第四步-控制台发布配置

### 创建并发布配置文件

北极星配置中心的控制台，配置文件名可以通过 / 来按树状目录结构展示，树状结构可以清晰的管理配置文件。例如一个应用下分不同的模块，每个模块都有独立的一组配置文件，则可以创建 ` module1/file1.properties`, `module1/file2.yaml`, `module2/file3.yaml`。

> 注意：配置文件名强烈建议带文件后缀，例如 `.properties` `.yaml` `.yml` `.json` 等。因为客户端会通过文件名后缀来解析文件内容，如果客户端发现不认识的后缀名则默认当做 `.properties` 文件处理。

配置好的实例如下图所示：

![image](https://user-images.githubusercontent.com/4991116/170200088-b4a4a1c0-a550-4400-8f17-923565f7b4ec.png)

### 北极星配置中心原理介绍

![image](https://user-images.githubusercontent.com/4991116/171319787-278ac349-da99-4cc1-a264-6e91c150fbe0.png)

**客户端端视角**

1. 应用启动时，同步从服务端拉取一次配置，获取最新的配置内容
2. 把第一步拉取到的所有的配置文件生成 `List<File->Version>` 的数据 ，并向服务端发送订阅配置请求，请求内容为 `List<File->Version>`
3. 当收到配置文件的推送消息时，向服务端拉取最新的配置文件

**订阅配置服务端视角**

1. 先检查客户端 `List<File->Version>` 的请求里是否存在 File 版本号落后，如果存在，则立马响应 `File -> NewVersion` 内容给客户端
2. 如果客户端配置文件版本号都是最新的，则在内存里维护 `File -> List<Client>` 的数据结构并 Hold 请求 30s。如果 30s 内有配置文件发布，则立马响应请求，返回 `File -> NewVersion` 给所有客户端

**发布推送配置简化流程**

1. 用户在界面点击发布按钮，服务端更新数据库里配置发布表的数据。配置发布表的核心字段：`file, version, content, mtime`
2. 每个北极星服务端实例，都会定时1s扫描配置发布表，根据 mtime 捞出最近 1s 内变更过的数据
3. 北极星服务端实例扫描到最新变更的数据之后

- 重新加载内存缓存
- 向内存里的消息发布管道里写入一条消息

1. 推送协程从消息发布管道里获取到消息，并消费消息。通过 `File -> List<Client>` 信息，获取所有订阅配置文件的客户端信息，并响应客户端 Hold 的请求。

## 服务限流

[官方文档](https://github.com/Tencent/spring-cloud-tencent/wiki/Spring-Cloud-Tencent-Rate-Limit-%E4%BD%BF%E7%94%A8%E6%96%87%E6%A1%A3)

### 1.第一步-引入依赖

#### 方式一：只引入`spring-cloud-starter-tencent-polaris-ratelimit`

```pom
<dependency>
    <groupId>com.tencent.cloud</groupId>
    <artifactId>spring-cloud-starter-tencent-polaris-ratelimit</artifactId>
</dependency>
```

#### **方式二：通过 `spring-cloud-starter-tencent-all` 引入 sct 所有 starters**

```pom
<dependency>
    <groupId>com.tencent.cloud</groupId>
    <artifactId>spring-cloud-starter-tencent-all</artifactId>
    <!-- 注意需要指定 type=pom-->
    <type>pom</type> 
</dependency>
```

如果使用 `Spring Cloud 2021` 版本，还需要添加如下依赖：

```pom
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

### 2.第二步-配置

在您的 `bootstrap.yml` 配置文件中加入以下配置内容

```yml
spring:
  application:
    name: ${application.name}
  cloud:
    polaris:
      address: grpc://${修改为第一步部署的 Polaris 服务地址}:8091
      namespace: default
```

**配置列表**

| 配置项Key                                                | 默认值                | 是否必填 | 配置项说明                                                   |
| -------------------------------------------------------- | --------------------- | -------- | ------------------------------------------------------------ |
| spring.cloud.polaris.ratelimit.enabled                   | true                  | 否       | 是否开启服务限流                                             |
| spring.cloud.polaris.ratelimit.rejectRequestTips         |                       | 否       | 自定义拒绝请求响应的文本内容                                 |
| spring.cloud.polaris.ratelimit.rejectRequestTipsFilePath |                       | 否       | 自定义拒绝请求响应内容的文件地址，常用于返回 html 文件内容等。目前仅支持 classpath 下的文件，放在 resources 目录。 |
| spring.cloud.polaris.ratelimit.rejectHttpCode            | 429(Too Many Request) | 否       | 自定义拒绝请求响应的 Http 状态码                             |
| spring.cloud.polaris.ratelimit.maxQueuingTime            | 1000                  | 否       | 匀速排队限流最大排队时间                                     |

### 4.第四步-控制台配置限流规则

可以在北极星控制台动态配置限流规则。

## 服务熔断

[官方文档](https://github.com/Tencent/spring-cloud-tencent/wiki/Spring-Cloud-Tencent-Circuitbreaker-%E4%BD%BF%E7%94%A8%E6%96%87%E6%A1%A3)

故障实例熔断是常见的一种容错保护机制。故障实例熔断能实现主调方迅速自动屏蔽错误率高或故障的服务实例，并启动定时任务对熔断实例进行探活。在达到恢复条件后对其进行半开恢复。在半开恢复后，释放少量请求去进行真实业务探测。并根据真实业务探测结果去判断是否完全恢复正常。

如下图所示当被调端 `Instance3` 出现异常时，主调端会自动的把请求只打到 `Instance1`，`Instance2`，通过这种方式保证主调端能够正常的调用服务。

![image](https://user-images.githubusercontent.com/4991116/161234560-ad9fb472-29da-41ff-95fc-3c805cd86357.png)

**熔断策略**

- 故障比例熔断：当服务实例在上一个时间窗（默认1分钟）内，通过的请求量达到或超过最小请求阈值（默认10个），且错误率达到或超过故障比率阈值（默认`50%`），实例会进入隔离状态。故障比率的阈值范围是 `[0.0, 1.0]`，代表 `0% - 100%`。
- 连续故障熔断：当实例在上一个时间窗（默认1分钟）内，连续失败的请求数达到或者超过连续故障阈值（默认10个），实例会进入隔离状态。
- 熔断隔离时间：默认隔离`30s`，支持可配置。

### 1.第一步-引入服务注册与发现和熔断依赖

#### 1. 主调方和被调方都引入服务注册与发现依赖

因为熔断能力依赖服务发现能力，所以需要主调方和被调方都引入服务注册与发现依赖。

##### 1.1 引入 Spring Cloud Tencent Discovery Starter

**方式一：只引入`spring-cloud-starter-tencent-polaris-discovery`**

```pom
<dependency>
    <groupId>com.tencent.cloud</groupId>
    <artifactId>spring-cloud-starter-tencent-polaris-discovery</artifactId>
</dependency>
```

**方式二：通过 `spring-cloud-starter-tencent-all` 引入 sct 所有 starters**

```pom
<dependency>
    <groupId>com.tencent.cloud</groupId>
    <artifactId>spring-cloud-starter-tencent-all</artifactId>
    <!-- 注意需要指定 type=pom-->
    <type>pom</type> 
</dependency>
```

如果使用 `Spring Cloud 2021` 版本，还需要添加如下依赖：

```pom
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

#### 2. 主调方引入熔断依赖

**方式一：只引入 `spring-cloud-starter-tencent-polaris-circuitbreaker`**

```pom
<dependency>
    <groupId>com.tencent.cloud</groupId>
    <artifactId>spring-cloud-starter-tencent-polaris-circuitbreaker</artifactId>
</dependency>
```

**方式二：通过 `spring-cloud-starter-tencent-all` 引入 sct 所有 starters**

```pom
<dependency>
    <groupId>com.tencent.cloud</groupId>
    <artifactId>spring-cloud-starter-tencent-all</artifactId>
    <!-- 注意需要指定 type=pom-->
    <type>pom</type> 
</dependency>
```

### 2.第二步：增加相关配置文件

在您的 `bootstrap.yml` 配置文件中加入以下配置内容

```yml
spring:
  application:
    name: ${application.name}
  cloud:
    polaris:
      address: grpc://${修改为第一步部署的 Polaris 服务地址}:8091
      namespace: default
    loadbalancer:
      configurations: polaris # 2020.0.x SDK需要添加这个配置
```

**配置列表**

| 配置项Key                                   | 默认值 | 是否必填 | 配置项说明       |
| ------------------------------------------- | ------ | -------- | ---------------- |
| spring.cloud.polaris.circuitbreaker.enabled | true   | 否       | 是否开启故障熔断 |

### 3.第三步-控制台配置熔断

控制台提供了动态下发熔断配置的能力，如下图所示可以根据自身业务的实际场景调整参数。

![image](https://user-images.githubusercontent.com/4991116/161363924-49d8988b-cf25-485c-a5c4-075f1075f7d9.png)
