---
title: SpringBoot2 线程池的定义和使用
tags:
  - SpringBoot
  - 线程池
categories:
  - SpringBoot
abbrlink: 8a11e15c
---

# SpringBoot2 线程池的定义和使用

**定义线程池**

```java
@Slf4j
@EnableAsync
@Configuration
public class AsyncExecutorConfig implements AsyncConfigurer {

    @Bean
    public ThreadPoolTaskExecutor asyncServiceExecutor() {
        //返回可用处理器的虚拟机的最大数量不小于1
        int cpu = Runtime.getRuntime().availableProcessors();
        log.info("start asyncServiceExecutor cpu : {}", cpu);
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        //配置核心线程数
        executor.setCorePoolSize(cpu);
        //配置最大线程数
        executor.setMaxPoolSize(cpu);
        //配置队列大小
        executor.setQueueCapacity(50);
        //用来设置线程池关闭的时候等待所有任务都完成再继续销毁其他的Bean
        executor.setWaitForTasksToCompleteOnShutdown(true);
        //设置线程池中任务的等待时间，如果超过这个时候还没有销毁就强制销毁，以确保应用最后能够被关闭，而不是阻塞住
        executor.setAwaitTerminationSeconds(60);
        //配置线程池中的线程的名称前缀
        executor.setThreadNamePrefix("async-service-");
        // rejection-policy：当pool已经达到max size的时候，如何处理新任务
        // CALLER_RUNS：不在新线程中执行任务，而是有调用者所在的线程来执行
        // 使用预定义的异常处理类
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        //执行初始化
        executor.initialize();
        return executor;
    }

    @Override
    public Executor getAsyncExecutor() {
        return asyncServiceExecutor();
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return (throwable, method, objects) -> {
            StringBuilder sb = new StringBuilder();
            for (Object param : objects) {
                sb.append(param).append(",");
            }
            log.error("Exception message - {}，Method name - {}，Parameter value - {}", throwable.getMessage(), method.getName(), sb.toString());
        };

    }
}
```

**如何使用**

```java
@Autowired    
private ThreadPoolTaskExecutor threadPoolTaskExecutor;

public void test(){
  CompletableFuture<Void> userFuture = CompletableFuture.runAsync(() ->  System.out.println(111), threadPoolTaskExecutor);
}
```

