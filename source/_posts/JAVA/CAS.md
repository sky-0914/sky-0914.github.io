---
title: CAS
tags:
  - CAS
categories:
  - JAVA
---
# CAS

**compare and swap/comparen and exchange (比较并交换)**

![](https://i.loli.net/2021/02/07/LbyRn3SloFiBImT.png)

![](https://i.loli.net/2021/02/07/zuT3aS7b94lODKB.png)

**注意ABA问题：**

a=0，线程一读取到a执行+1操作，此时有另外一个线程，线程二也读取到了0，并且执行了+1操作，然后又执行了-1操作=0，线程二又写回到a，此时a又=0；当线程一执行写回的时候去判断a是不是等于原来的值0。

但是此时的0已经不是原来的0了，因为过程中+1又-1才又等于0。打个比方跟女朋友分手后，她又经历过两个男人，然后又复合。那么她还是原来的她了吗？

那么如何解决这个问题呢？

CAS就是乐观锁，乐观的认为并不会发生改变，那么我们就用乐观锁的实现机制，加上版本号，当任何一个线程对这个值进行修改的时候，都去给这个版本号+1，然后在比较的时候不仅仅需要比较原值，还需要比较版本号是否一致。

**举例说明：**

JDK里提供的原子类在这个包下`java.util.concurrent.atomic`

```java
//Integer原子类
AtomicInteger i = new AtomicInteger();
i.incrementAndGet();

//源码
//Atomically increments by one the current value. 将当前值原子地加1。
//Returns:the updated value 返回:更新后的值
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}

//Java和C++语言的一个重要区别就是Java中我们无法直接操作一块内存区域，不能像C++中那样可以自己申请内存和释放内存。Java中的Unsafe类为我们提供了类似C++手动管理内存的能力。
//Unsafe类，全限定名是sun.misc.Unsafe，从名字中我们可以看出来这个类对普通程序员来说是“危险”的，一般应用开发者不会用到这个类。
//Unsafe类是"final"的，不允许继承。且构造函数是private的:
//Unsafe无法实例化，那么怎么获取Unsafe呢？答案就是通过反射来获取Unsafe
public Unsafe getUnsafe() throws IllegalAccessException {
    Field unsafeField = Unsafe.class.getDeclaredFields()[0];
    unsafeField.setAccessible(true);
    Unsafe unsafe = (Unsafe) unsafeField.get(null);
    return unsafe;
}
//Java中的Unsafe类
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
//native C++方法
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
//汇编指令，硬件直接支持 lock cmpxchg
```

