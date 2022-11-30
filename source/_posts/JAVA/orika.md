---
title: orika 深拷贝
tags:
  - 深拷贝
categories:
  - JAVA
---
# orika 深拷贝

```pom
<dependency>
    <groupId>ma.glasnost.orika</groupId>
    <artifactId>orika-core</artifactId>
    <version>1.5.4</version>
</dependency>
```

**定义Account对象**
```java
@Data
public class Account {
    private long id;
    private String username;
    private String password;
}
```
**定义Friend**
```java
@Data
public class Friend {
    private String a;
    private String b;
}
```
**定义UserA对象**
```java
@Data
public class UserA {
    private long id;
    private String name;
    private int age;
    private Account account;

    private List<Friend> friends;

    private String a;
}
```
**定义UserB对象**
```java
@Data
public class UserB {
    private long id;
    private String name;
    private int age;
    private Account account;

    private List<Friend> friends;

    private String b;
}
```

**示例**
```java
@SpringBootTest
class OrikaApplicationTests {

    @Test
    void contextLoads() {

        Account account = new Account();
        account.setId(1);
        account.setUsername("zhangsan");
        account.setPassword("zhangsan");

        Friend friend1 = new Friend();
        friend1.setA("a");
        friend1.setB("a");
        Friend friend2 = new Friend();
        friend2.setA("b");
        friend2.setB("b");
        List<Friend> friendListA = new ArrayList<>();
        friendListA.add(friend1);
        friendListA.add(friend2);

        UserA userA = new UserA();
        userA.setId(1);
        userA.setName("张三");
        userA.setAge(18);
        userA.setAccount(account);
        userA.setFriends(friendListA);
        userA.setA("A");

        UserB userB = new UserB();
        List<Friend> friendListB = new ArrayList<>();
        Friend friend3 = new Friend();
        friend3.setA("c");
        friend3.setB("c");
        friendListB.add(friend3);
        userB.setFriends(friendListB);

        System.out.println(userA);
        System.out.println(userB);

        //转换
        MapperFactory mapperFactory = new DefaultMapperFactory.Builder().build();
        MapperFacade mapper = mapperFactory.getMapperFacade();
//        UserB userB = mapper.map(userA, UserB.class);
        mapper.map(userA, userB);
        System.out.println(userA);
        System.out.println(userB);
    }

}
```
