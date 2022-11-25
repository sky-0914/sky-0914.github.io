---
title: 5-JVM常用的命令
tags:
  - JVM
categories:
  - JVM
---
# JVM常用的命令

## jps ：基础工具

查看JAVA进程PID。

`jps` 命令用来查看所有 Java 进程，每一行就是一个 Java 进程信息。

`jps` 仅查找当前用户的 Java 进程，而不是当前系统中的所有进程，要显示其他用户的还只能用 `ps` 命令。

**jps 常用参数**

- `jps -l` 如果是以 class 方式运行，会显示进程的主类 `main.class` 的全名，如果是以 jar 包方式运行的，就会输出 jar 包的完整路径名

第一列的数字就是进程的 `pid`

- `jps -v` 输出传递给 JVM 的参数，`v` 表示虚拟机，`jps -vl` 比较常见的组合；
- `jps -V` 大写 `v`，表示通过文件传递给 JVM 的参数

```java
Copy# michael @ Michael-MBP in ~ [16:37:59]
$ jps -v |grep Mybatis
8005 MybatisDemoApplication -agentlib:jdwp=transport=dt_socket,address=127.0.0.1:53364,suspend=y,server=n -XX:TieredStopAtLevel=1 -Xverify:none -Dspring.output.ansi.enabled=always -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=53363 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=127.0.0.1 -Dspring.liveBeansView.mbeanDomain -Dspring.application.admin.enabled=true -javaagent:/Users/michael/Library/Caches/IntelliJIdea2018.2/captureAgent/debugger-agent.jar=file:/private/var/folders/m1/ydypchs901lffc5sms07mrp40000gn/T/capture.props -Dfile.encoding=UTF-8
```

- `jps -m` 输出传递给 `main.class` 方法的参数，实用的一个命令，`jps -ml` 比较实用的组合，会显示包名/类名/参数
- `jps -q` 只输出进程的 pid

jps 是比较常用的 Java 命令。使用 jps 可以查看当前有哪些 Java 进程处于运行状态。如果运行了一个 web 应用（使用tomcat、jboss、jetty等启动）的时候，就可以使用 jps 查看启动情况。

有的时候我想知道这个应用的日志会输出到哪里，或者启动的时候使用了哪些javaagent，那么我可以使用 `jps -v` 查看进程的jvm参数情况。

---



##	jinfo：命令可以用来查看 Java 进程运行的 JVM 参数、

参考原文：https://blog.csdn.net/yx0628/article/details/80958488

`jinfo -flag initilHeapSize ${PID} `查看某个**JAVA进程**中，JVM的参数值是什么。

`jinfo -flag ${PID} ` 如果不加JVM参数的话，默认查看JVM中所有被修改过的值。

```shell
[root@admin ~]# jinfo --help
Usage:
    jinfo [option] <pid>
        (to connect to running process)
    jinfo [option] <executable <core>
        (to connect to a core file)
    jinfo [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

where <option> is one of:
    -flag <name>         to print the value of the named VM flag
    -flag [+|-]<name>    to enable or disable the named VM flag
    -flag <name>=<value> to set the named VM flag to the given value
    -flags               to print VM flags
    -sysprops            to print Java system properties
    <no option>          to print both of the above
    -h | -help           to print this help message
```

我们先用 jps 命令查到 PID ，然后可以通过 jinfo 来查看对应进程的参数信息：

```shell
[root@admin ~]# jps
43520 Test
35900 Jps
```

查看 JVM 参数：

```shell
[root@admin ~]# jinfo -flags 43520
Attaching to process ID 43520, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 24.45-b08

-Dfile.encoding=GBK
```

查看系统参数：

```shell
[root@admin ~]# jinfo -sysflags 43520
```

虚拟机的参数可以通过这个命令查看：

```shell
java -XX:+PrintFlagsFinal -version | grep manageable
```

除了通过启动脚本可以设置参数，PrintGC 默认是打开的，因此我们只需要打开 PrintGCDetails 参数。

```shell
jinfo -flag +PrintGC 43520
jinfo -flag +PrintGCDetails 43520
```

关闭 GC 日志的话同理：

```shell
jinfo -flag -PrintGC 43520
jinfo -flag -PrintGCDetails 43520
```

查看是否开启 GC 日志的打印：

```shell
jinfo -flag PrintGC 43520
jinfo -flag PrintGCDetails 43520
```

```shell
[root@admin ~]# jinfo -flag PrintGC 43520
-XX:-PrintGC

[root@admin ~]# jinfo -flag PrintGCDetails 43520
-XX:-PrintGCDetails
```

常用 JVM 参数：

```shell
-Xms：初始堆大小，默认为物理内存的1/64(<1GB)；默认(MinHeapFreeRatio参数可以调整)空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制
-Xmx：最大堆大小，默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制
-Xmn：新生代的内存空间大小，注意：此处的大小是（eden+ 2 survivor space)。与jmap -heap中显示的New gen是不同的。整个堆大小=新生代大小 + 老生代大小 + 永久代大小。在保证堆大小不变的情况下，增大新生代后,将会减小老生代大小。此值对系统性能影响较大,Sun官方推荐配置为整个堆的3/8。
-XX:SurvivorRatio：新生代中Eden区域与Survivor区域的容量比值，默认值为8。两个Survivor区与一个Eden区的比值为2:8,一个Survivor区占整个年轻代的1/10。
-Xss：每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1M,以前每个线程堆栈大小为256K。应根据应用的线程所需内存大小进行适当调整。在相同物理内存下,减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。一般小的应用， 如果栈不是很深， 应该是128k够用的，大的应用建议使用256k。这个选项对性能影响比较大，需要严格的测试。和threadstacksize选项解释很类似,官方文档似乎没有解释,在论坛中有这样一句话:"-Xss is translated in a VM flag named ThreadStackSize”一般设置这个值就可以了。
-XX:PermSize：设置永久代(perm gen)初始值。默认值为物理内存的1/64。
-XX:MaxPermSize：设置持久代最大值。物理内存的1/4。
```

---



##	jstat：主要是对java应用程序的资源和性能进行实时的命令行监控，包括了对heap size和垃圾回收状况的监控

原文参考：https://blog.csdn.net/cockroach02/article/details/82670500

查看JVM中相关性能的信息。

jstat（Java Virtual Machine Statistics Monitoring Tool）是从JDK1.5自带的一个轻量级小工具。它位于java/bin目录下，主要利用JVM内建的指令对Java虚拟机的资源和性能进行实时的监控。

> 类装载信息：`jstat -class ${PID}`
>
> 举例：`jstat -class ${PID} ${几秒内打印一次} ${打印10次} = jstat -class ${PID} 1000 10`。打印出过去10秒类加载的数据。

> GC相关的情况：`jstat -gc ${PID}`
>
> 举例：`jstat -gc ${PID} ${几秒内打印一次} ${打印10次} = jstat -gc ${PID} 1000 10`。打印出过去10秒GC的数据。

**参数说明：**

```shell
C:\Users\Administrator>jstat -help
Usage: jstat -help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

Definitions:
  <option>      An option reported by the -options option
  <vmid>        Virtual Machine Identifier. A vmid takes the following form:
                     <lvmid>[@<hostname>[:<port>]]
                Where <lvmid> is the local vm identifier for the target
                Java virtual machine, typically a process id; <hostname> is
                the name of the host running the target Java virtual machine;
                and <port> is the port number for the rmiregistry on the
                target host. See the jvmstat documentation for a more complete
                description of the Virtual Machine Identifier.
  <lines>       Number of samples between header lines.
  <interval>    Sampling interval. The following forms are allowed:
                    <n>["ms"|"s"]
                Where <n> is an integer and the suffix specifies the units as
                milliseconds("ms") or seconds("s"). The default units are "ms".
  <count>       Number of samples to take before terminating.
  -J<flag>      Pass <flag> directly to the runtime system.
```

由以上可知，jstat的命令的格式如下：

```shell
jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
```

我们可以通过jstat -options查看可以支持的具体参数

```shell
C:\Users\Administrator>jstat -options
-class
-compiler
-gc
-gccapacity
-gccause
-gcmetacapacity
-gcnew
-gcnewcapacity
-gcold
-gcoldcapacity
-gcutil
-printcompilation
```

**Option参数说明：**

|       参数        |                             说明                             |
| :---------------: | :----------------------------------------------------------: |
|      -class       | 类加载的行为统计 Displays statistics about the behavior of the class loader. |
|     -compiler     | HotSpt JIT编译器行为统计 Displays statistics about the behavior of the Java HotSpot VM Just-in-Time compiler. |
|        -gc        | 垃圾回收堆的行为统计。Displays statistics about the behavior of the garbage collected heap. |
|    -gccapacity    | 各个垃圾回收代容量(young,old,meta)和他们相应的空间统计。isplays statistics about the capacities of the generations and their corresponding spaces. |
|     -gccause      | 垃圾收集统计概述（同-gcutil）,附加最近两次垃圾回收事件的原因。Displays a summary about garbage collection statistics (same as -gcutil), with the cause of the last and current (when applicable) garbage collection events. |
|  -gcmetacapacity  | 统计元空间使用情况 Displays statistics about the sizes of the metaspace. |
|      -gcnew       | 显示新生代统计信息 Displays statistics of the behavior of the new generation. |
|  -gcnewcapacity   | 统计新生代及内存使用情况 Displays statistics about the sizes of the new generations and its corresponding spaces. |
|      -gcold       | 统计老年代和元空间使用情况 Displays statistics about the behavior of the old generation and metaspace statistics. |
|  -gcoldcapacity   | 统计老年代内存使用情况 Displays statistics about the sizes of the old generation. |
|      -gcutil      | 显示各个各代内存使用百分比 Displays a summary about garbage collection statistics. |
| -printcompilation | Hotspot方法编译统计情况 Displays Java HotSpot VM compilation method statistics. |

**使用示例：**

**jstat -class ： 类加载的行为统计**

```shell
C:\Users\Administrator>jstat -class 2284
Loaded  Bytes  Unloaded  Bytes     Time
 30116 75021.8       26    51.4      86.72
```

- Loaded ：加载class的数量
- Bytes ： 加载class的大小（单位KB）
- Unloaded ：卸载class的数量
- Bytes： 卸载class的大小（单位KB）
- Time ： 加载和卸载class所耗费的时间

**jstat -compiler ：HotSpt JIT编译器行为统计**

```shell
C:\Users\Administrator>jstat -compiler 2284
Compiled Failed Invalid   Time   FailedType FailedMethod
   21247      8       0   189.38          1 com/fr/third/alibaba/druid/pool/DruidDataSource shrink
```

- Compiled ：编译成功数
- Failed ： 编译失败数
- Invalid ： 无效数量
- FailedType ： 最后一次编译失效类型
- FailedMethod ：最后一次编译失效的方法

**jstat -gc：垃圾回收堆的行为统计**

```shell
C:\Users\Administrator>jstat -gc 2284
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
104832.0 104832.0  0.0    0.0   838912.0 600103.2 1048576.0   565828.4  238672.0 232272.3 22392.0 21028.5     59    4.898  11      6.323   11.221
```

+ S0C ： 第一个幸存区的总容量（KB）
+ S1C ： 第二个幸存区的总容量（KB）
+ S0U ： 第一个幸存区已使用的容量（KB）
+ S1U ： 第二个幸存区已使用的容量（KB）
+ EC ： 伊甸区的总容量（KB）
+ EU ： 伊甸区已使用容量（KB）
+ OC ： 老年区的总容量（KB）
+ MC ： 元空间的总容量（KB）
+ MU ： 元空间已使用容量（KB）
+ CCSC ： 压缩类空间总容量（KB）
+ CCSU ： 压缩类空间已使用总容量（KB）
+ YGC ： 新生代GC次数
+ YGCT ：新生代GC总耗费时间
+ FGC ： 老年代GC次数
+ FGCT ： 老年代GC总耗费时间
+ GCT ： GC总耗费时间

**jstat -gccapacity ： 各个回收区内存情况**

```shell
C:\Users\Administrator>jstat -gccapacity 2284
 NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      OGCMX       OGC         OC       MCMN     MCMX      MC     CCSMN    CCSMX     CCSC    YGC    FGC
1048576.0 1048576.0 1048576.0 104832.0 104832.0 838912.0  1048576.0  3145728.0  1048576.0  1048576.0      0.0 1265664.0 238672.0      0.0 1048576.0  22392.0
59    11
```

+ NGCMN ： 新生代占用的最小空间大小（KB）
+ NGCMX ： 新生代占用的最大空间大小（KB）
+ NGC ： 当前新生代空间大小（KB）
+ S0C ： 第一幸存区当前空间大小（KB）
+ S1C ：第二幸存区当前空间大小（KB）
+ EC ： 伊甸区当前空间大小（KB）
+ OGCMN ： 老年区最小空间大小（KB）
+ OGCMX ： 老年区最大空间大小（KB）
+ OGC ： 老年区当前空间大小（KB）
+ MCMN ： 元空间最小空间大小（KB）
+ MCMX ： 元空间最大空间大小（KB）
+ MC ： 元空间当前空间大小（KB）
+ CCSMN ： 压缩类空间最小大小（KB）
+ CCSMX ： 压缩类最大空间大小（KB）
+ CCSC ： 压缩类当前空间大小（KB）
+ YGC ： 新生代GC次数
+ FGC ： 老年代GC次数

**jstat -gccause ：垃圾收集统计概述**

```shell
C:\Users\Administrator>jstat -gccause 2284
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC
  0.00   0.00  91.90  53.96  97.32  93.91     59    4.898    11    6.323   11.221 System.gc()          No GC
```

+ S0 ：第一幸存区已使用空间百分比.
+ S1 ： 第二幸存区已使用空间百分比
+ E ： 伊甸区已使用空间百分比
+ O ： 老年区已使用空间百分比
+ M ： 元空间使用百分比
+ CCS ： 压缩类空间使用百分比
+ YGC ： 新生代GC次数
+ FGC ： 老年代GC次数
+ LGCC ：最近一次GC原因
+ GCC ： 当前GC原因

**6 . gc -gcmetacapacity ：元空间使用情况**

```shell
C:\Users\Administrator>jstat -gcmetacapacity 2284
   MCMN       MCMX        MC       CCSMN      CCSMX       CCSC     YGC   FGC    FGCT     GCT
       0.0  1265664.0   238672.0        0.0  1048576.0    22392.0    59    11    6.323   11.221

```

+ MCMN ： 元空间最小空间大小（KB）
+ MCMX ： 元空间最大空间大小（KB）
+ MC ： 元空间当前空间大小（KB）
+ CCSMN ： 压缩类空间最小空间大小（KB）
+ CCSMX ： 压缩类空间最大空间大小（KB）
+ CCSC ： 压缩类空间当前空间大小（KB）
+ YGC ： 新生代GC次数
+ FGC ： 老年代GC次数
+ FGCT： 老年代GC耗费时间
+ GCT ： GC总耗费时间

**7 . jstat -gcnew ： 显示新生代统计信息**

```shell
C:\Users\Administrator>jstat -gcnew 2284
 S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT
104832.0 104832.0 46710.2    0.0  6   6 52416.0 838912.0  22526.9     60    4.972
```

+ S0C ： 第一幸存区总空间大小（KB）
+ S1C ： 第二幸存区总空间大小（KB）
+ S0U ： 第一幸存区已使用空间大小（KB）
+ S1U ： 第二幸存区已使用空间大小（KB）
+ TT ： 提升阈值(提升阈值)
+ MTT ： 最大阈值
+ DSS ： survivor区域大小 (KB)
+ EC ： 伊甸区总空间大小（KB）
+ EU ： 伊甸区已使用空间大小（KB）

**8 . jstat -gcnewcapacity ： 统计新生代及内存使用情况**

```shell
C:\Users\Administrator>jstat -gcnewcapacity 2284
  NGCMN      NGCMX       NGC      S0CMX     S0C     S1CMX     S1C       ECMX        EC      YGC   FGC
 1048576.0  1048576.0  1048576.0 104832.0 104832.0 104832.0 104832.0   838912.0   838912.0    60    11
```

+ NGCMN ： 新生代最小空间大小（KB）
+ NGCMX ： 新生代最大空间大小（KB）
+ NGC ： 当前新生代空间大小（KB）
+ S0CMX ： 第一幸存区最大空间大小（KB）
+ S0C ： 第一幸存区当前空间大小（KB）
+ S1CMX ： 第二幸存区最大空间大小（KB）
+ S1C ： 第二幸存区当前空间大小（KB）
+ ECMX ： 伊甸区最大空间大小（KB）
+ EC ： 伊甸区当前空间大小（KB）
+ YGC ： 新生代GC次数
+ FGC ： 老年代GC次数

**9 . jstat -gcold ： 统计老年代和元空间使用情况**

```shell
C:\Users\Administrator>jstat -gcold 2284
   MC       MU      CCSC     CCSU       OC          OU       YGC    FGC    FGCT     GCT
251088.0 244521.5  23544.0  22058.7   1048576.0    565828.4     60    11    6.323   11.295
```

+ MC ： 元空间总大小（KB）
+ MU ： 元空间已使用大小（KB）
+ CCSC ： 压缩类空间总大小（KB）
+ CCSU ： 压缩类空间已使用大小（KB）
+ OC ： 老年区空间总大小（KB）
+ OU ： 老年区已使用大小（KB）
+ YGC ： 新生代GC次数
+ FGC ： 老年代GC次数
+ FGCT ： 老年代GC总耗时
+ GCT ： GC总耗时

**10 . jstat -gcoldcapacity ： 统计老年代内存使用情况**

```shell
C:\Users\Administrator>jstat -gcoldcapacity 2284
   OGCMN       OGCMX        OGC         OC       YGC   FGC    FGCT     GCT
  1048576.0   3145728.0   1048576.0   1048576.0    60    11    6.323   11.295
```

+ OGCMN ：老年区占用最小空间（KB）
+ OGCMX ： 老年区占用最大空间（KB）
+ OGC ： 当前老年区空间（KB）
+ OC ： 当前老年区空间（KB）
+ YGC ：新生代GC次数
+ FGC ： 老年代GC次数
+ FGCT ： 老年代GC总耗时
+ GCT ： GC总耗时

**11 . jstat -gcutil ： 垃圾回收统计**

```shell
C:\Users\Administrator>jstat -gcutil 2284
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
 44.56   0.00   9.85  53.96  97.38  93.69     60    4.972    11    6.323   11.295
```

+ S0 ：第一幸存区已使用空间百分比.
+ S1 ： 第二幸存区已使用空间百分比
+ E ： 伊甸区已使用空间百分比
+ O ： 老年区已使用空间百分比
+ M ： 元空间使用百分比
+ CCS ： 压缩类空间使用百分比
+ YGC ： 新生代GC次数
+ FGC ： 老年代GC次数
+ GCT ：GC总耗时

**12 . jstat -printcompilation ： Hotspot方法编译统计情况**

```shell
C:\Users\Administrator>jstat -printcompilation 2284
Compiled  Size  Type Method
   21538    150    1 java/util/Collections reverse
```

+ Compiled ： 已编译方法次数
+ Size ： 最近一次方法编译大小
+ Type ： 最近一次编译方法类型
+ Method ： 最近一次编译方法

参考：

jstat：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html
jstat命令使用：https://www.cnblogs.com/lizhonghua34/p/7307139.html
JVM调优命令-jstat：https://www.cnblogs.com/myna/p/7567769.html

---



##	jstack：查看某个Java进程内的线程堆栈信息

参考原文：https://www.jianshu.com/p/8d5782bc596e

查看JAVA进程当中，线程内容。

**jstack用法**

```kotlin
/opt/java8/bin/jstack

Usage:
    jstack [-l] <pid>
        (to connect to running process) 连接活动线程
    jstack -F [-m] [-l] <pid>
        (to connect to a hung process) 连接阻塞线程
    jstack [-m] [-l] <executable> <core>
        (to connect to a core file) 连接dump的文件
    jstack [-m] [-l] [server_id@]<remote server IP or hostname>
        (to connect to a remote debug server) 连接远程服务器

Options:
    -F  to force a thread dump. Use when jstack <pid> does not respond (process is hung)
    -m  to print both java and native frames (mixed mode)
    -l  long listing. Prints additional information about locks
    -h or -help to print this help message
```

**jstack查看输出**

```bash
/opt/java8/bin/jstack -l 28367

2019-06-25 15:04:46
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.77-b03 mixed mode):

"Attach Listener" #453 daemon prio=9 os_prio=0 tid=0x00007f9f94001000 nid=0xf30 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"grpc-default-executor-263" #452 daemon prio=5 os_prio=0 tid=0x00007f9f4c01f800 nid=0x9aa waiting on condition [0x00007f9f398bd000]
   java.lang.Thread.State: TIMED_WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000007400243f0> (a java.util.concurrent.SynchronousQueue$TransferStack)
        at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:215)
        at java.util.concurrent.SynchronousQueue$TransferStack.awaitFulfill(SynchronousQueue.java:460)
        at java.util.concurrent.SynchronousQueue$TransferStack.transfer(SynchronousQueue.java:362)
        at java.util.concurrent.SynchronousQueue.poll(SynchronousQueue.java:941)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1066)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1127)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
        - None

"http-bio-8080-exec-10" #235 daemon prio=5 os_prio=0 tid=0x0000000001bcc800 nid=0x3c13 waiting on condition [0x00007f9f384a9000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x0000000743d26638> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
        at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1067)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1127)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
        - None
```

**jstack统计线程数**

```bash
/opt/java8/bin/jstack -l 28367 | grep 'java.lang.Thread.State' | wc -l
```

**jstack检测死锁**

*死锁代码*

```csharp
public class DeathLock {

    private static Lock lock1 = new ReentrantLock();
    private static Lock lock2 = new ReentrantLock();

    public static void deathLock() {
        Thread t1 = new Thread() {
            @Override
            public void run() {
                try {
                    lock1.lock();
                    TimeUnit.SECONDS.sleep(1);
                    lock2.lock();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        Thread t2 = new Thread() {
            @Override
            public void run() {
                try {
                    lock2.lock();
                    TimeUnit.SECONDS.sleep(1);
                    lock1.lock();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };

        t1.setName("thread1");
        t2.setName("thread2");
        t1.start();
        t2.start();
    }

    public static void main(String[] args) {
        deathLock();
    }
}
```

*死锁日志*

```bash
"mythread2" #12 prio=5 os_prio=0 tid=0x0000000058ef7800 nid=0x1ab4 waiting on condition [0x0000000059f8f000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000d602d610> (a java.util.concurrent.lock
s.ReentrantLock$NonfairSync)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInt
errupt(AbstractQueuedSynchronizer.java:836)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireQueued(A
bstractQueuedSynchronizer.java:870)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(Abstrac
tQueuedSynchronizer.java:1199)
        at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLo
ck.java:209)
        at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:285)

        at DeathLock$2.run(DeathLock.java:34)

   Locked ownable synchronizers:
        - <0x00000000d602d640> (a java.util.concurrent.locks.ReentrantLock$Nonfa
irSync)

"mythread1" #11 prio=5 os_prio=0 tid=0x0000000058ef7000 nid=0x3e68 waiting on condition [0x000000005947f000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000d602d640> (a java.util.concurrent.lock
s.ReentrantLock$NonfairSync)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInt
errupt(AbstractQueuedSynchronizer.java:836)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireQueued(A
bstractQueuedSynchronizer.java:870)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(Abstrac
tQueuedSynchronizer.java:1199)
        at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLo
ck.java:209)
        at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:285)

        at DeathLock$1.run(DeathLock.java:22)

   Locked ownable synchronizers:
        - <0x00000000d602d610> (a java.util.concurrent.locks.ReentrantLock$Nonfa
irSync)


Found one Java-level deadlock:
=============================
"mythread2":
  waiting for ownable synchronizer 0x00000000d602d610, (a java.util.concurrent.l
ocks.ReentrantLock$NonfairSync),
  which is held by "mythread1"
"mythread1":
  waiting for ownable synchronizer 0x00000000d602d640, (a java.util.concurrent.l
ocks.ReentrantLock$NonfairSync),
  which is held by "mythread2"

Java stack information for the threads listed above:
===================================================
"mythread2":
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000d602d610> (a java.util.concurrent.lock
s.ReentrantLock$NonfairSync)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInt
errupt(AbstractQueuedSynchronizer.java:836)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireQueued(A
bstractQueuedSynchronizer.java:870)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(Abstrac
tQueuedSynchronizer.java:1199)
        at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLo
ck.java:209)
        at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:285)

        at DeathLock$2.run(DeathLock.java:34)
"mythread1":
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000d602d640> (a java.util.concurrent.lock
s.ReentrantLock$NonfairSync)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInt
errupt(AbstractQueuedSynchronizer.java:836)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireQueued(A
bstractQueuedSynchronizer.java:870)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(Abstrac
tQueuedSynchronizer.java:1199)
        at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLo
ck.java:209)
        at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:285)

        at DeathLock$1.run(DeathLock.java:22)

Found 1 deadlock.
```

**jstack检测cpu高**

*步骤一：查看cpu占用高进程*

```cpp
top

Mem:  16333644k total,  9472968k used,  6860676k free,   165616k buffers
Swap:        0k total,        0k used,        0k free,  6665292k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND     
17850 root      20   0 7588m 112m  11m S 100.7  0.7  47:53.80 java       
 1552 root      20   0  121m  13m 8524 S  0.7  0.1  14:37.75 AliYunDun   
 3581 root      20   0 9750m 2.0g  13m S  0.7 12.9 298:30.20 java        
    1 root      20   0 19360 1612 1308 S  0.0  0.0   0:00.81 init        
    2 root      20   0     0    0    0 S  0.0  0.0   0:00.00 kthreadd    
    3 root      RT   0     0    0    0 S  0.0  0.0   0:00.14 migration/0 
```

*步骤二：查看cpu占用高线程*

```css
top -H -p 17850

top - 17:43:15 up 5 days,  7:31,  1 user,  load average: 0.99, 0.97, 0.91
Tasks:  32 total,   1 running,  31 sleeping,   0 stopped,   0 zombie
Cpu(s):  3.7%us,  8.9%sy,  0.0%ni, 87.4%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:  16333644k total,  9592504k used,  6741140k free,   165700k buffers
Swap:        0k total,        0k used,        0k free,  6781620k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
17880 root      20   0 7588m 112m  11m R 99.9  0.7  50:47.43 java
17856 root      20   0 7588m 112m  11m S  0.3  0.7   0:02.08 java
17850 root      20   0 7588m 112m  11m S  0.0  0.7   0:00.00 java
17851 root      20   0 7588m 112m  11m S  0.0  0.7   0:00.23 java
17852 root      20   0 7588m 112m  11m S  0.0  0.7   0:02.09 java
17853 root      20   0 7588m 112m  11m S  0.0  0.7   0:02.12 java
17854 root      20   0 7588m 112m  11m S  0.0  0.7   0:02.07 java
```

*步骤三：转换线程ID*

```bash
printf "%x\n" 17880          
45d8
```

*步骤四：定位cpu占用线程*

```bash
jstack 17850|grep 45d8 -A 30
"pool-1-thread-11" #20 prio=5 os_prio=0 tid=0x00007fc860352800 nid=0x45d8 runnable [0x00007fc8417d2000]
   java.lang.Thread.State: RUNNABLE
        at java.io.FileOutputStream.writeBytes(Native Method)
        at java.io.FileOutputStream.write(FileOutputStream.java:326)
        at java.io.BufferedOutputStream.flushBuffer(BufferedOutputStream.java:82)
        at java.io.BufferedOutputStream.flush(BufferedOutputStream.java:140)
        - locked <0x00000006c6c2e708> (a java.io.BufferedOutputStream)
        at java.io.PrintStream.write(PrintStream.java:482)
        - locked <0x00000006c6c10178> (a java.io.PrintStream)
        at sun.nio.cs.StreamEncoder.writeBytes(StreamEncoder.java:221)
        at sun.nio.cs.StreamEncoder.implFlushBuffer(StreamEncoder.java:291)
        at sun.nio.cs.StreamEncoder.flushBuffer(StreamEncoder.java:104)
        - locked <0x00000006c6c26620> (a java.io.OutputStreamWriter)
        at java.io.OutputStreamWriter.flushBuffer(OutputStreamWriter.java:185)
        at java.io.PrintStream.write(PrintStream.java:527)
        - eliminated <0x00000006c6c10178> (a java.io.PrintStream)
        at java.io.PrintStream.print(PrintStream.java:597)
        at java.io.PrintStream.println(PrintStream.java:736)
        - locked <0x00000006c6c10178> (a java.io.PrintStream)
        at com.demo.guava.HardTask.call(HardTask.java:18)
        at com.demo.guava.HardTask.call(HardTask.java:9)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at java.lang.Thread.run(Thread.java:745)

"pool-1-thread-10" #19 prio=5 os_prio=0 tid=0x00007fc860345000 nid=0x45d7 waiting on condition [0x00007fc8418d3000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000006c6c14178> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
```

---



## Jmap

原文参考：https://www.jianshu.com/p/c52ffaca40a5

> jmap是JDK自带的工具软件，主要用于打印指定Java进程(或核心文件、远程调试服务器)的`共享对象内存映射或堆内存细节`。可以使用jmap生成Heap Dump。在Java命令Java Dump和Java命令:Jstack中分别有关于Java Dump以及线程 Dump的介绍。 这篇文章主要介绍`Java的堆Dump`以及jmap命令

### 什么是堆Dump

### 基础知识

[Java虚拟机的内存组成以及堆内存介绍](链接地址)
 [Java GC工作原理](链接地址)

常见内存错误：

> outOfMemoryError 年老代内存不足。
>  outOfMemoryError:PermGen Space 永久代内存不足。
>  outOfMemoryError:GC overhead limit exceed 垃圾回收时间占用系统运行时间的98%或以上。

**jmap 用法摘要**

```bash
Usage:
    jmap [option] <pid>
        (to connect to running process)
    jmap [option] <executable <core>
        (to connect to a core file)
    jmap [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

where <option> is one of:
    <none>               to print same info as Solaris pmap
    -heap                to print java heap summary
    -histo[:live]        to print histogram of java object heap; if the "live"
                         suboption is specified, only count live objects
    -permstat            to print permanent generation statistics
    -finalizerinfo       to print information on objects awaiting finalization
    -dump:<dump-options> to dump java heap in hprof binary format
                         dump-options:
                           live         dump only live objects; if not specified,
                                        all objects in the heap are dumped.
                           format=b     binary format
                           file=<file>  dump heap to <file>
                         Example: jmap -dump:live,format=b,file=heap.bin <pid>
    -F                   force. Use with -dump:<dump-options> <pid> or -histo
                         to force a heap dump or histogram when <pid> does not
                         respond. The "live" suboption is not supported
                         in this mode.
    -h | -help           to print this help message
    -J<flag>             to pass <flag> directly to the runtime system
```

- 指定进程号(pid)的进程 jmap [ option ]
- 指定核心文件 jmap [ option ]
- 指定远程调试服务器 jmap [ option ] [server-id@]

**参数：**

- option 选项参数是互斥的(不可同时使用)。想要使用选项参数，直接跟在命令名称后即可。
- pid 需要打印配置信息的进程ID。该进程必须是一个Java进程。想要获取运行的Java进程列表，你可以使用jps。
- executable 产生核心dump的Java可执行文件。
- core 需要打印配置信息的核心文件。
- remote-hostname-or-IP 远程调试服务器的(请查看jsadebugd)主机名或IP地址。
- server-id 可选的唯一id，如果相同的远程主机上运行了多台调试服务器，用此选项参数标识服务器。

**选项:**

- <no option> 如果使用不带选项参数的jmap打印共享对象映射，将会打印目标虚拟机中加载的每个共享对象的起始地址、映射大小以及共享对象文件的路径全称。这与Solaris的pmap工具比较相似。
- -dump:[live,]format=b,file=<filename> 以hprof二进制格式转储Java堆到指定filename的文件中。live子选项是可选的。如果指定了live子选项，堆中只有活动的对象会被转储。想要浏览heap dump，你可以使用jhat(Java堆分析工具)读取生成的文件。
- -finalizerinfo 打印等待终结的对象信息。
- -heap 打印一个堆的摘要信息，包括使用的GC算法、堆配置信息和generation wise heap usage。
- -histo[:live] 打印堆的柱状图。其中包括每个Java类、对象数量、内存大小(单位：字节)、完全限定的类名。打印的虚拟机内部的类名称将会带有一个’*’前缀。如果指定了live子选项，则只计算活动的对象。
- -permstat 打印Java堆内存的永久保存区域的类加载器的智能统计信息。对于每个类加载器而言，它的名称、活跃度、地址、父类加载器、它所加载的类的数量和大小都会被打印。此外，包含的字符串数量和大小也会被打印。
- -F 强制模式。如果指定的pid没有响应，请使用jmap -dump或jmap -histo选项。此模式下，不支持live子选项。
- -h 打印帮助信息。
- -help 打印帮助信息。
- -J<flag> 指定传递给运行jmap的JVM的参数。

**示例：**

*查看java 堆（heap）使用情况,执行命令：  jmap -heap 31846*

```bash
Attaching to process ID 31846, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 24.71-b01

using thread-local object allocation.
Parallel GC with 4 thread(s)//GC 方式

Heap Configuration: //堆内存初始化配置
   MinHeapFreeRatio = 0 //对应jvm启动参数-XX:MinHeapFreeRatio设置JVM堆最小空闲比率(default 40)
   MaxHeapFreeRatio = 100 //对应jvm启动参数 -XX:MaxHeapFreeRatio设置JVM堆最大空闲比率(default 70)
   MaxHeapSize      = 2082471936 (1986.0MB) //对应jvm启动参数-XX:MaxHeapSize=设置JVM堆的最大大小
   NewSize          = 1310720 (1.25MB)//对应jvm启动参数-XX:NewSize=设置JVM堆的‘新生代’的默认大小
   MaxNewSize       = 17592186044415 MB//对应jvm启动参数-XX:MaxNewSize=设置JVM堆的‘新生代’的最大大小
   OldSize          = 5439488 (5.1875MB)//对应jvm启动参数-XX:OldSize=<value>:设置JVM堆的‘老生代’的大小
   NewRatio         = 2 //对应jvm启动参数-XX:NewRatio=:‘新生代’和‘老生代’的大小比率
   SurvivorRatio    = 8 //对应jvm启动参数-XX:SurvivorRatio=设置年轻代中Eden区与Survivor区的大小比值 
   PermSize         = 21757952 (20.75MB)  //对应jvm启动参数-XX:PermSize=<value>:设置JVM堆的‘永生代’的初始大小
   MaxPermSize      = 85983232 (82.0MB)//对应jvm启动参数-XX:MaxPermSize=<value>:设置JVM堆的‘永生代’的最大大小
   G1HeapRegionSize = 0 (0.0MB)

Heap Usage://堆内存使用情况
PS Young Generation
Eden Space://Eden区内存分布
   capacity = 33030144 (31.5MB)//Eden区总容量
   used     = 1524040 (1.4534378051757812MB)  //Eden区已使用
   free     = 31506104 (30.04656219482422MB)  //Eden区剩余容量
   4.614088270399305% used //Eden区使用比率
From Space:  //其中一个Survivor区的内存分布
   capacity = 5242880 (5.0MB)
   used     = 0 (0.0MB)
   free     = 5242880 (5.0MB)
   0.0% used
To Space:  //另一个Survivor区的内存分布
   capacity = 5242880 (5.0MB)
   used     = 0 (0.0MB)
   free     = 5242880 (5.0MB)
   0.0% used
PS Old Generation //当前的Old区内存分布
   capacity = 86507520 (82.5MB)
   used     = 0 (0.0MB)
   free     = 86507520 (82.5MB)
   0.0% used
PS Perm Generation//当前的 “永生代” 内存分布
   capacity = 22020096 (21.0MB)
   used     = 2496528 (2.3808746337890625MB)
   free     = 19523568 (18.619125366210938MB)
   11.337498256138392% used

670 interned Strings occupying 43720 bytes.
```

**查看堆内存(histogram)中的对象数量及大小。执行命令： jmap -histo 3331**

```bash
num     #instances         #bytes  class name
编号     个数                字节     类名
----------------------------------------------
   1:             7        1322080  [I
   2:          5603         722368  <methodKlass>
   3:          5603         641944  <constMethodKlass>
   4:         34022         544352  java.lang.Integer
   5:           371         437208  <constantPoolKlass>
   6:           336         270624  <constantPoolCacheKlass>
   7:           371         253816  <instanceKlassKlass>
```

**将内存使用的详细情况输出到文件，执行命令： jmap -dump:format=b,file=heapDump 6900**

然后用jhat命令可以参看 jhat -port 5000 heapDump 在浏览器中访问：http://localhost:5000/ 查看详细信息

> 这个命令执行，JVM会将整个heap的信息dump写入到一个文件，heap如果比较大的话，就会导致这个过程比较耗时，并且执行的过程中为了保证dump的信息是可靠的，所以会暂停应用。

### 总结

1.如果程序内存不足或者频繁GC，很有可能存在内存泄露情况，这时候就要借助Java堆Dump查看对象的情况。
 2.要制作堆Dump可以直接使用jvm自带的jmap命令
 3.可以先使用jmap -heap命令查看堆的使用情况，看一下各个堆空间的占用情况。
 4.使用jmap -histo:[live]查看堆内存中的对象的情况。如果有大量对象在持续被引用，并没有被释放掉，那就产生了内存泄露，就要结合代码，把不用的对象释放掉。
 5.也可以使用 jmap -dump:format=b,file=<fileName>命令将堆信息保存到一个文件中，再借助jhat命令查看详细内容
 6.在内存出现泄露、溢出或者其它前提条件下，建议多dump几次内存，把内存文件进行编号归档，便于后续内存整理分析。

### 出现问题：

1. Error attaching to process: sun.jvm.hotspot.debugger.DebuggerException: Can’t attach to the process
    在ubuntu中第一次使用jmap会报错：Error attaching to process: sun.jvm.hotspot.debugger.DebuggerException: Can't attach to the process，这是oracla文档中提到的一个bug:http://bugs.java.com/bugdatabase/view_bug.do?bug_id=7050524,解决方式如下：

- echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope 该方法在下次重启前有效。
- 永久有效方法 sudo vi /etc/sysctl.d/10-ptrace.conf 编辑下面这行: kernel.yama.ptrace_scope = 1 修改为: kernel.yama.ptrace_scope = 0 重启系统，使修改生效。

