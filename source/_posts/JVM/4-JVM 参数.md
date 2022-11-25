---
title: 4-JVM 参数
tags:
  - JVM
categories:
  - JVM
---
# JVM 参数

1. 标准参数：不会随着jdk版本的变化而变化。比如：java -version、java -help

2. 非标准参数：随着JDK版本的变化而变化。

   -X参数【用的较少】非标准参数，也就是在JDK各个版本中可能会变动

   ```shell
   # 修改执行方式
   # compiled 编译执行方式，第一次使用就编译成本地代码
   java -Xcomp -version
   # interpreted 解释执行方式
   java -Xini -version
   # mixed 默认的混合执行方式，混合模式，JVM自己来决定
   java -Xmixed -version
   ```

   -XX参数【用的最多：JVM调优额Debug】非标准化参数，相对不稳定。

   boolean类型

   ```shell
   格式：-XX:[+-]<name>      		+或-表示启用或者禁用name属性
   比如：
   -XX:+UseConcMarkSweepGC  		表示启用CMS类型的垃圾回收器
   -XX:+UseG1GC       				表示启用G1类型的垃圾回收器
   # 设置JVM使用垃圾收集器 +：使用，-：未使用
   -XX:+/-UseG1GC
   
   
   ```

   非boolean类型

   ```shell
   格式：-XX<name>=<value>表示name属性的值是value
   比如：
   -XX:MaxGCPauseMillis=500
   # 设置初始堆内存
   -XX:initialHeapSize=100M # 简写方式 ===>>> -Xms100M
   # 设置最大堆内存
   -XX:MaxHeapSize=100M # 简写方式 ===>>> -Xmx100M
   ```

打印出JAVA进程中所有JVM的参数：

+ 执行命令 `java -XX:+PrintFlagsFinal -version` 即可打印出JVM中各参数。由于参数较多，建议将参数放入文件中查看：	`java -XX:+PrintFlagsFinal -version > xxx.txt`
+ 使用  `jinfo -flag ${参数名} ${PID} `也可查看对应JVM参数值

```shell
-XX:+PrintFlagsFinal
```

**如何设置JVM参数：**

*以下方式是JAVA进程还未启动*

1. 在开发工具中，比如IDEA中的启动配置项中 VM options
2. SpringBoot jar包 启动的命令 java -jar -Xms100M test.jar
3. 在java语言的中间件中如Tomcat中的 bin 目录下的 catalina.sh/bat 中的JAVA_OPTIONS=""

*以下方式是当JAVA进程已经启动后该如何设置*

4. 实时修改某个JVM参数的值 **jinfo修改（参数只有被标记为manageable的flags可以被实时修改)**

**常用参数：**

|                             参数                             |                             含义                             |                             说明                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|                     XX:CICompilerCount=3                     |                        最大并行编译数                        | 如果设置大于1，虽然编译速度会提高，但是同样影响系统稳定性，会增加JVM崩溃的可能 |
|                   -XX:InitialHeapSize=100M                   |                         初始化堆大小                         |                         简写-Xms100M                         |
|                     -XX:MaxHeapSize=100M                     |                          最大堆大小                          |                        简写-Xm x 100M                        |
|                       -XX:NewSize=20M                        |                       设置年轻代的大小                       |                                                              |
|                      -XX:MaxNewSize=50M                      |                        年轻代最大大小                        |                                                              |
|                       -XX:OldSize=50M                        |                        设置老年代大小                        |                                                              |
|                    -XX:MetaspaceSize=50M                     |                        设置方法区大小                        |                                                              |
|                   -XX:MaxMetaspaceSize=50M                   |                        方法区最大大小                        |                                                              |
|                      -XX:+UseParallelGC                      |                      使用UseParallelGC                       |                      新生代，吞吐量优先                      |
|                    -XX:+UseParallelOldGC                     |                     使用UseParallelOldGC                     |                      老年代，吞吐量优先                      |
|                   -XX:+UseConcMarkSweepGC                    |                           使用CMS                            |                     老年代，停顿时间优先                     |
|                         -XX:+UseG1GC                         |                           使用G1GC                           |                 新生代，老年代，停顿时间优先                 |
|                         -XX:NewRatio                         |                        新老生代的比值                        | 比如-XX:Ratio=4，则表示新生代:老年代=1:4，也就是新生代占整个堆内存的1/5 |
|                      -XX:SurvivorRatio                       |                    两个S区和Eden区的比值                     | 比如-XX:SurvivorRatio=8，也就是(S0+S1):Eden=2:8，也就是一个S占整个新生代的1/10 |
|               -XX:+HeapDumpOnOutOfMemoryError                |                      启动堆内存溢出打印                      |      当JVM堆内存发生溢出时，也就是OOM，自动生成dump文件      |
|                 -XX:HeapDumpPath=heap.hprof                  |                    指定堆内存溢出打印目录                    |             表示在当前目录生成一个heap.hprof文件             |
| XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStampsXloggc:$CATALINA_HOME/logs/gc.log |                         打印出GC日志                         |           可以使用不同的垃圾收集器，对比查看GC情况           |
|                           -Xss128k                           |                    设置每个线程的堆栈大小                    |                    经验值是3000-5000最佳                     |
|                  -XX:MaxTenuringThreshold=6                  |                    提升年老代的最大临界值                    |                         默认值为 15                          |
|              -XX:InitiatingHeapOccupancyPercent              |                启动并发GC周期时堆内存使用占比                | G1之类的垃圾收集器用它来触发并发GC周期,基于整个堆的使用率,而不只是某一代内存的使用比. 值为 0 则表示”一直执行GC循环”. 默认值为 45. |
|                    -XX:G1HeapWastePercent                    |                    允许的浪费堆空间的占比                    | 默认是10%，如果并发标记可回收的空间小于10%,则不会触发MixedGC。 |
|                  -XX:MaxGCPauseMillis=200ms                  |                        G1最大停顿时间                        | 暂停时间不能太小，太小的话就会导致出现G1跟不上垃圾产生的速度。最终退化成Full GC。所以对这个参数的调优是一个持续的过程，逐步调整到最佳状态 |
|                     -XX:ConcGCThreads=n                      |                 并发垃圾收集器使用的线程数量                 |               默认值随JVM运行的平台不同而不同                |
|             -XX:G1MixedGCLiveThresholdPercent=65             |        混合垃圾回收周期中要包括的旧区域设置占用率阈值        |                       默认占用率为 65%                       |
|                  -XX:G1MixedGCCountTarget=8                  | 设置标记周期完成后，对存活数据上限为G1MixedGCLIveThresholdPercent 的旧区域执行混合垃圾回收的目标次数 | 默认8次混合垃圾回收，混合回收的目标是要控制在此目标次数以内  |
|             XX:G1OldCSetRegionThresholdPercent=1             |           描述Mixed GC时，Old Region被加入到CSet中           |        默认情况下，G1只把10%的Old Region加入到CSet中         |
