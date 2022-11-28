---
title: JSON字符串带BOM头
tags:
  - JAVA
categories:
  - JAVA
---
**调用三方接口返回值JSON字符串带BOM头"\ufeff"，JSON解析死活报错。**

> 我是用SpringBoot的`RestTemplate`调用三方接口的，一开始返回值我是用对象接收返回值，发现一直报错，我以为是`RestTemplate`的接收转换有问题，就将返回值换成了`String`类型去接收。接收到字符串后再转JSON、JSON字符串解析死活报错。

接口返回值日志如下：

```
2020-03-25 13:18:55.687 DEBUG 8595 --- [           main] o.s.web.client.RestTemplate              : Response 200 OK
2020-03-25 13:18:55.688 DEBUG 8595 --- [           main] o.s.web.client.RestTemplate              : Reading to [java.lang.String] as "application/json;charset=UTF-8"
2020-03-25 13:19:57.370 DEBUG 8595 --- [           main] com.hopefun.scm.open.api.EyuanApi        : 返回值：﻿{"message":"成功","code":"1"}
```

> 在IDEA开发过程中，一开始光看返回值打印的日志是看不出来任何毛病的，并且我将这个返回值的JSON字符串复制到Sublime编辑器中也看不出问题所在。

> 一开始很自信没有Debug查看返回值，后来当我开启了Debug模式后终于发现了问题所在。原来在JSON字符串前面还带着"\ufeff"，导致JSON字符串解析报错，原来罪魁祸首是这个玩意"\ufeff"。Debug还是个好玩意啊。

> 终于发现问题所在解决就轻松多了。

```java
public static final String BOM = "\ufeff";
/**
 * 去除BOM
 *
 * @param bomStr JSON字符串
 * @return 去除BOM后的JSON字符串
 */
private String recursiveBom(String bomStr) {
    String str = "";
    if (bomStr.startsWith(BOM)) {
        str = bomStr.substring(1);
        if (str.startsWith(BOM)) {
            recursiveBom(str);
        }
    }
    return str;
}

//使用，如此得出来的字符串就是纯正的JSON字符串啦。妈妈再也不怕解析报错啦。。。
recursiveBom(bomStr.trim());
```

