---
title: LambdaCriteria
tags:
  - java8
categories:
  - JAVA
---
# LambdaCriteria

不知道各位读者有没有使用过[mybatis-plus](https://mp.baomidou.com/),如果使用过的话，那么条件构造器一定也比较熟悉。



那么先让我们来熟悉下所需要的一些基本知识。

## Function<T, R>

> JAVA8新增函数式接口

```java

/**
 * Represents a function that accepts one argument and produces a result.
 表示接受一个参数并产生结果的函数。
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #apply(Object)}.
 *
 * @param <T> the type of the input to the function:函数的输入类型
 * @param <R> the type of the result of the function:函数结果的类型
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Function<T, R> {

    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);
...
```

**@FunctionalInterface**

> 1. 该注解只能标记在"有且仅有一个抽象方法"的接口上。
> 2. JDK8接口中的静态方法和默认方法，都不算是抽象方法。
> 3. 接口默认继承java.lang.Object，所以如果接口显示声明覆盖了Object中方法，那么也不算抽象方法。
> 4. 该注解不是必须的，如果一个接口符合"函数式接口"定义，那么加不加该注解都没有影响。加上该注解能够更好地让编译器进行检查。如果编写的不是函数式接口，但是加上了@FunctionInterface，那么编译器会报错。

## 自定义`SFunction<T, R>`接口

> 参考mybatis-plus的Lambda条件构造器自定义`SFunction<T, R>`继承`Function<T, R>`, 和`Serializable`；使我们自定义的`SFunction<T, R>`函数式接口是支持序列化的。

```java
/**
 * 使Function获取序列化能力
 *
 * @author ZC
 * @date 2021/4/2 14:32
 */
@FunctionalInterface
public interface SFunction<T, R> extends Function<T, R>, Serializable {
}
```

那么函数式接口序列化之后能够获得什么呢？请继续往下看

## SerializedLambda

> `SerializedLambda`是jdk1.8提供的一个新的类，凡是继承了Serializable的函数式接口的实例都可以获取一个属于它的SerializedLambda实例，并且通过它获取到方法的名称，根据我们标准的java bean的定义规则就可以通过方法名称来获取属性名称

```java
public static <T> SerializedLambda getSerializedLambda(SFunction<T, ?> fn) {
    SerializedLambda lambda = null;
    try {
        //获取函数式接口的Class类型，再获取该Class里的序列化writeReplace方法
        Method method = fn.getClass().getDeclaredMethod("writeReplace");
        method.setAccessible(Boolean.TRUE);
        //然后调用该方法，最后获取的是SerializedLambda对象
        lambda  = (SerializedLambda) method.invoke(fn);
    } catch (Exception e) {
        throw e;
    }
    return lambda;
}
```

| 变量和类型 | 方法                                      | 描述                                                         |
| :--------- | :---------------------------------------- | :----------------------------------------------------------- |
| `Object`   | `getCapturedArg(int i)`                   | 获取lambda捕获站点的动态参数。                               |
| `int`      | `getCapturedArgCount()`                   | 获取lambda捕获站点的动态参数计数。                           |
| `String`   | `getCapturingClass()`                     | 获取捕获此lambda的类的名称。                                 |
| `String`   | `getFunctionalInterfaceClass()`           | 获取此lambda已转换为的调用类型的名称                         |
| `String`   | `getFunctionalInterfaceMethodName()`      | 获取已转换此lambda的功能接口的主要方法的名称。               |
| `String`   | `getFunctionalInterfaceMethodSignature()` | 获取此lambda已转换为的功能接口的主要方法的签名。             |
| `String`   | `getImplClass()`                          | **获取包含实现方法的类的名称。**                             |
| `int`      | `getImplMethodKind()`                     | 获取方法句柄类（参见[`MethodHandleInfo`](http://www.1024sky.cn/blog/article/MethodHandleInfo.html) ）的实现方法。 |
| `String`   | `getImplMethodName()`                     | **获取实现方法的名称。**                                     |
| `String`   | `getImplMethodSignature()`                | 获取实现方法的签名。                                         |
| `String`   | `getInstantiatedMethodType()`             | 在将类型变量替换为来自捕获站点的实例化之后，获取主要功能接口方法的签名。 |

这里我们主要使用的就是`getImplClass()`和`getImplMethodName()`

```java
public static <T> String getSerializedLambda(SFunction<T, ?> fn) {
    SerializedLambda serializedLambda = serializedLambda(fn);
    // 从lambda信息取出method、field、class等
    String fieldName = serializedLambda.getImplMethodName().substring("get".length());
    fieldName = fieldName.replaceFirst(fieldName.charAt(0) + "", (fieldName.charAt(0) + "").toLowerCase());
    Field field;
    try {
        field = Class.forName(serializedLambda.getImplClass().replace("/", ".")).getDeclaredField(fieldName);
    } catch (ClassNotFoundException | NoSuchFieldException e) {
        throw new RuntimeException(e);
    }
    return fieldName;
}
```

此时我们就能获取到字段名了

```java
@Data
class Student{
    private String name;
    private int age;
    private String sex;
}

public static void main(String[] args) {
    System.out.println(ColumnUtil.getName(Student::getName));
    System.out.println(ColumnUtil.getName(Student::getAge));
}
```

**好了，接下来我们就可以参照`mybatis-plus`的条件构造器，来实现我们自己的`mongoDB`或者`ES`的条件构造器了。**

