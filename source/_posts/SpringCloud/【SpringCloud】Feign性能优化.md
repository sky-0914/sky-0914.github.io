---
title: 【SpringCloud】Feign性能优化
tags:
  - SpringCloud
  - Feign
categories:
  - SpringCloud
---
#### 1、替换 tomcat
首先，把 tomcat 换成 undertow，这个性能在 Jmeter 的压测下，undertow 比 tomcat 高一倍
**第一步，pom 修改去除tomcat**
```pom
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
		<exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```
**第二步，配置**
```yml
server:
  undertow:
    max-http-post-size: 0
# 设置IO线程数, 它主要执行非阻塞的任务,它们会负责多个连接, 默认设置每个CPU核心一个线程,数量和CPU 内核数目一样即可
    io-threads: 4
# 阻塞任务线程池, 当执行类似servlet请求阻塞操作, undertow会从这个线程池中取得线程,它的值设置取决于系统的负载  io-threads*8
    worker-threads: 32
# 以下的配置会影响buffer,这些buffer会用于服务器连接的IO操作,有点类似netty的池化内存管理
# 每块buffer的空间大小,越小的空间被利用越充分
    buffer-size: 1024
# 每个区分配的buffer数量 , 所以pool的大小是buffer-size * buffers-per-region
#   buffers-per-region: 1024 # 这个参数不需要写了
# 是否分配的直接内存
    direct-buffers: true
```
#### 2、替换 HTTPClient
**第一步，加依赖**
```pom
<dependency>
	<groupId>io.github.openfeign</groupId>
	<artifactId>feign-httpclient</artifactId>
</dependency>
```
**第二部，在 application.yml或者 bootstrap.yml 里面配置**
```yml
# feign配置
feign:
  hystrix:
    # 在feign中开启hystrix功能，默认情况下feign不开启hystrix功能
    enabled: true
  ## 配置httpclient线程池
  httpclient:
    enabled: true
  okhttp:
    enabled: false
```
**第三步，配置 HTTPClient Bean**
```java
import java.util.Timer;
import java.util.TimerTask;
import java.util.concurrent.TimeUnit;

import org.apache.http.client.HttpClient;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.impl.client.DefaultConnectionKeepAliveStrategy;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class HttpPool {

    @Bean
    public HttpClient httpClient(){
        System.out.println("===== Apache httpclient 初始化连接池开始===" );
        // 生成默认请求配置
        RequestConfig.Builder requestConfigBuilder = RequestConfig.custom();
        // 超时时间
        requestConfigBuilder.setSocketTimeout(5 * 1000);
        // 连接时间
        requestConfigBuilder.setConnectTimeout(5 * 1000);
        RequestConfig defaultRequestConfig = requestConfigBuilder.build();
        // 连接池配置
        // 长连接保持30秒
        final PoolingHttpClientConnectionManager pollingConnectionManager = new PoolingHttpClientConnectionManager(30, TimeUnit.MILLISECONDS);
        // 总连接数
        pollingConnectionManager.setMaxTotal(1000);
        // 同路由的并发数
        pollingConnectionManager.setDefaultMaxPerRoute(100);

        // httpclient 配置
        HttpClientBuilder httpClientBuilder = HttpClientBuilder.create();
        // 保持长连接配置，需要在头添加Keep-Alive
        httpClientBuilder.setKeepAliveStrategy(new DefaultConnectionKeepAliveStrategy());
        httpClientBuilder.setConnectionManager(pollingConnectionManager);
        httpClientBuilder.setDefaultRequestConfig(defaultRequestConfig);
        HttpClient client = httpClientBuilder.build();

        // 启动定时器，定时回收过期的连接
        Timer timer = new Timer();
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                System.out.println("=====closeIdleConnections===");
                pollingConnectionManager.closeExpiredConnections();
                pollingConnectionManager.closeIdleConnections(5, TimeUnit.SECONDS);
            }
        }, 10 * 1000, 5 * 1000);
        System.out.println("===== Apache httpclient 初始化连接池完毕===");

        return client;
    }
}
```
#### 3、配置 Hystrix
**第一步，依赖**
```pom
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```
**第二步，配置**
```yml
# 配置hystrix的参数
hystrix:
  threadpool:
    # default: 默认参数，作用的所有的hystrix的客户端,如果需要对某个具体的接口，可以写接口+方法名称
    default:
      coreSize: 500
  command:
    default:
      fallback:
        # 是否开启回退方法
        enabled: true
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 30000 #缺省为1000
```
原文链接：[https://www.jianshu.com/p/fe1c4412de7f](https://www.jianshu.com/p/fe1c4412de7f "https://www.jianshu.com/p/fe1c4412de7f")
