---
title: 【SpringCloud】Ribbon组成和负载均衡规则
tags:
  - SpringCloud
  - Ribbon
categories:
  - SpringCloud
---
### Ribbon饥饿加载
>	默认情况下Ribbon是懒加载的。当服务起动好之后，第一次请求是非常慢的，第二次之后就快很多。

#### 解决方式：开启饥饿加载
```yml
ribbon:
 eager-load:
  enabled: true #开启饥饿加载
  clients: server-1,server-2,server-3 #为哪些服务的名称开启饥饿加载,多个用逗号分隔
```

### Ribbon组成
| 接口  					   | 作用  						| 默认值  |
| ------------ 				| ------------ 					| ------------ |
| `IclientConfig`  			| 读取配置  					| `DefaultClientConfigImpl`  |
| `IRule` 		    		| 负载均衡规则，选择实例 		  | `ZoneAvoidanceRule`  |
| `IPing`  					| 筛选掉ping不通的实例  		 | `DumyPing`（该类什么不干，认为每个实例都可用，都能ping通）  |
| `ServerList<Server>`  		| 交给Ribbon的实例列表  		 | **Ribbon:**`ConfigurationBasedServerList`<br>**Spring Cloud Alibaba:**`NacosServerList` |
| `ServerListFilter<Server>`  | 过滤掉不符合条件的实例 		  | `ZonePreferenceServerListFilter`  |
| `ILoadBalancer` 			| Ribbon的入口  				| `ZoneAwareLoadBalancer`  |
| `ServerListUpdater`  		| 更新交给Ribbon的List的策略  	| `PollingServerListUpdater`  |

**这里的每一项都可以自定义**
`IclientConfig`Ribbon支持非常灵活的配置就是由该组件提供的
`IRule`为Ribbon提供规则，从而选择实例、该组件是最核心的组件
**举例：**
代码方式
```java
@Configuration
public class RibbonRuleConfig {
    @Bean
    public IRule ribbonRulr() {
        return new RandomRule();
    }
	@Bean
	public IPing iPing(){
		return new PingUrl();
	}
}
```
配置属性方式
```yml
<clientName>:
 ribbon:
  NFLoadBalancerClassName: #ILoadBalancer该接口实现类
  NFLoadBalancerRuleClassName: #IRule该接口实现类
  NFLoadBalancerPingClassName: #Iping该接口实现类
  NIWSServerListClassName: #ServerList该接口实现类
  NIWSServerListFilterClassName: #ServiceListFilter该接口实现类
```
**在这些属性中定义的类优先于使用`@RibbonClient(configuration=RibbonConfig.class)`Spring 定义的bean 以及由Spring Cloud Netflix提供的默认值。描述：配置文件中定义ribbon优先代码定义**

### Ribbon负载均衡的八种算法，其中~~`ResponseTimeWeightedRule`~~已废除
|  规则名称 | 特点  |
| ------------ | ------------ |
| `AvailabilityFilteringRule`  | 过滤掉一直连接失败的被标记为circuit tripped（电路跳闸）的后端Service，并过滤掉那些高并发的后端Server或者使用一个AvailabilityPredicate来包含过滤Server的逻辑，其实就是检查status的记录的各个Server的运行状态  |
| `BestAvailableRule`  | 选择一个最小的并发请求的Server，逐个考察Server，如果Server被tripped了，则跳过  |
| `RandomRule`  | 随机选择一个Server  |
| ~~`ResponseTimeWeightedRule`~~  | 已废弃，作用同WeightedResponseTimeRule  |
| `RetryRule`  | 对选定的负责均衡策略机上充值机制，在一个配置时间段内当选择Server不成功，则一直尝试使用subRule的方式选择一个可用的Server  |
| `RoundRobinRule`  | 轮询选择，轮询index，选择index对应位置Server  |
| `WeightedResponseTimeRule`  | 根据相应时间加权，相应时间越长，权重越小，被选中的可能性越低  |
| `ZoneAvoidanceRule`  | （默认是这个）负责判断Server所Zone的性能和Server的可用性选择Server，在没有Zone的环境下，类似于轮询（`RoundRobinRule`）  |


### 实现负载均衡<细粒度>配置-随机

#### 方式一：JAVA代码方式
首先定义RestTemplate，并且添加注解`@LoadBalanced`，这样RestTemplate就实现了负载均衡
```java
@LoadBalanced
@Bean
public RestTemplate restTemplate() {
//template.getMessageConverters().set(1, new StringHttpMessageConverter(StandardCharsets.UTF_8));//解决中文乱码
return new RestTemplate();
}
```

在SpringBootApplication主类下添加配置类。该类主要作用于为哪个服务做负载均衡。默认的是轮训
```java
@Configuration
@RibbonClient(name = "${服务名称}", configuration = GoodsRibbonRuleConfig.class)//configuration: 指向负载均衡规则的配置类
public class GoodsRibbonConfig {
}
```

添加Ribbon的配置类，注意该类必须配置在`@SpringBootApplication`主类以外的包下。不然的话所有的服务都会按照这个规则来实现。会被所有的RibbonClient共享。主要是主类的主上下文和Ribbon的子上下文起冲突了。父子上下文不能重叠。相关连接：https://blog.csdn.net/qq_32588349/article/details/52097943
```java
@Configuration
public class GoodsRibbonRuleConfig {
    @Bean
    public IRule ribbonRulr() {
        return new RandomRule();
    }
}
```
或者使用自定义注解排除该类

#### 方式一：配置属性方式
```yml
server-1: # 服务名称 Service-ID
  ribbon:
    # 属性配置方式【推荐】
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule #  配置文件配置负载均衡算法-我这里使用的是自定义的Ribbon的负载均衡算法，默认
```
**优先级：配置（不会影响其他服务）>(大于) 硬编码（类得写在SpringBoot启动类包外，不然会影响其他服务）**

#### 总结：
| 配置方式  | 有点  | 缺点  |
| ------------ | ------------ | ------------ |
| 代码配置  | 基于代码，更加灵活  | 有坑（父子上下文）<br> 线上修改得重新打包，发布  |
| 属性配置  | 易上手 配置更加直观 <br> 线上修改无需重新打包，发布 <br> **优先级更高**  | 极端场景下没有配置配置方式灵活  |

### 实现负载均衡<全局>配置-随机

#### 方式一：Ribbon的配置类定义在主类下
~~让ComponentScan上下文重叠（**强烈不建议使用**）~~
#### 方式二：
```java
@Configuration
@RibbonClients(defaultConfiguration = GoodsRibbonRuleConfig.class)//Ribbon负载均衡全局粒度配置（所有服务都按照这个配置）
public class RibbonConfig {
}
```
