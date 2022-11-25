---
title: 2-内存模型
tags:
  - JVM
categories:
  - JVM
---
# 内存模型

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200507002258.png)

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200429225857.png)

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200429233701.png)

## 方法区

> JDK1.7 之前包含1.7 将方法区称为 Perm Space 永久代
>
> JDK1.8之后包含1.8 将方法区称为 MetaSpace 元空间。

## 堆（分配内存会大一些）

> 分配对象、new 实例。
>
> 堆内存当中划分为两个区域：老年代和新生代。内存分配为3（老年代）：1（新生代）
>
> 如何去划分老年代和新生代，根据对象的年龄。这个年龄是一个对象经过一次GC，如果还存在的话，年龄就加一。当年龄超过默认值（15）时，就会从新生代划分到老年代当中。

#### 1.新生代（Young）

1. new Object() ，实例化10个单位为1的对象 ===>>> 新生代分配
2. 新生代内存不够用时，触发GC
3. GC之后，释放空间，会存在空间碎片
4. 这时又new一个对象，这时这个对象的单位是3；GC之后释放的空间不连续，导致新生代不够分配，又会再一次触GC
5. GC的弊端就是会消耗线程资源，stop the world。

**以上的设计显然是不合理的，重新设计之后**

新生代new出来的对象是朝生夕死，将新生代划分为两个区域：Eden区，Survivor区。Survivor区又划分为S0、S1两个区域；

新生代内存分配情况是：Eden区80%，Survivor区20%（S0：10%，S1：10%）。

如果刚new出来的对象太大，超过了新生代的Eden区内存，会直接存入在老年代。

*举例说明：*

老年代：2G内存

新生代：1G内存

> Eden区800MB
>
> S0，S1各100MB

这是new一个900MB的对象，会直接分配在老年代（Old）里，这时这个对象是老年代在管理，新生代发生GC的时候并不会清理这个对象，老年代发生GC时才会清除这个对象。

**新生代（Young）GC：Minor GC**

**老年代（Old）GC：Major GC**

+ **Eden区**

所有刚刚new出来的对象，就会分配在Eden区。

+ **Survivor区**

1. S0、S1永远有一块内存是浪费的，一块被使用；

2. S0、S1两个区域互相转换身份，以空间的浪费换取内存空间的连续性；

3. Eden=80%；S0=10%；S1=10%；Eden：S0：S1=8：1：1；

4. 比如说直接new一个900MB的新对象，会直接在老年代（Old）区进行分配；

5. 如果新生代（Young）区的Young GC之后对象的年龄不断的+1+1+1 > 年龄15之后，会将该对象存放到老年代（Old）区；

   假如这时新生代（Young）区有120MB存活对象，S区不够放了，会跟老年代借20MB的空间存放，会触发担保机制，这20MB依旧还是属于老年代（Old）管理的。

6. 极端情况，如果有个对象超过老年代内存直接OOM。

#### 2.老年代（Old）

如果老年代的内存不够用了，会触发 **Old GC** 也可 称为 **Major GC**。Old GC会比较耗时。当然一旦触发了**Old GC（Major GC）**通常都会伴随着**Young GC（Minor GC）**。

**Old GC（Major GC）+ Young GC（Minor GC）+ MetaSpace GC（可以忽略它）= Full GC**

调优的原则：

避免触发Full GC，换句话说避免触发Old GC（Major GC）；如果要触发GC，尽量只触发Young GC（Minor GC）。

1. 尽量减少GC次数
2. 尽量只触发Young GC（Minor GC）

<img src="https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200429224902.png" style="zoom:150%;" />

**实操：**

在IDEA中VM options设置JVM堆内存：-Xms30M -Xmx30M（设置堆内存30MB，最大30MB）

```java
@RestController
@RequestMapping("/test/jvm")
public class TestJvmController {
    List<AuthAccount> list = new ArrayList<>();
    @GetMapping("/jvmTest")
    public void jvmTest() {
        while (true) {
            list.add(new AuthAccount());
        }
    }
}
```

运行SpringBoot程序后、在JAVA安装目录中，找到bin文件夹下的**jvisualvm**工具（这个工具是JDK自带的），**首先还要安装Visual GC的插件才能查看到JVM GC运行时状况**。

之后等待程序运行后，打开这个**jvisualvm**工具就可以查看到JVM内存运行时的状况

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200429231237.png)

如果堆内存中，没有可分配的内存空间了，就会报OOM。

同理方法区Metaspace也会报OOM，设置JVM中方法区大小：-XX:MetaspaceSize=40M -XX:MaxMetaspaceSize=40M。

栈也会报OOM，首先我们先测试栈的深度：

```java
	// 通过递归操作
	public static long count = 0;

    public static void test(long i) {
        System.out.println(count++);
        test(i);
    }

    public static void main(String[] args) {
        test(count);
    }
```

![](https://happyloves.oss-cn-hangzhou.aliyuncs.com/img/20200429233002.png)

通过测试我们发现，栈的默认深度是7000左右。之后就会报OOM错误。

可以根据需求去调整栈的深度大小；

一个栈的深度大小，太大或太小都会有弊端，太小的话影响方法链调用的深度、太大的话在整个JAVA进程当中它能够创建这样一个的线程的数量是有限的，如果太大会影响到其他线程创建栈的深度。

通过前人的经验来看，最佳值设置到5000左右就可以了。可以通过JVM参数去设置。
