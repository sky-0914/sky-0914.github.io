---
title: SpringBoot Shiro 配置自定义密码加密器
tags:
  - SpringBoot
  - Shiro
categories:
  - SpringBoot
---
# SpringBoot Shiro 配置自定义密码加密器

**自定义认证加密方式**

```java
/**
 * 自定义认证加密方式
 */
public static class CustomCredentialsMatcher extends SimpleCredentialsMatcher {
    @Override
    public boolean doCredentialsMatch(AuthenticationToken authcToken, AuthenticationInfo info) {
        UsernamePasswordToken token = (UsernamePasswordToken) authcToken;
        //加密类型，密码，盐值，迭代次数
        Object tokenCredentials = new SimpleHash("md5", password, username, 2).toHex();
        //数据库存储密码
        Object accountCredentials = getCredentials(info);
        //将密码加密与系统加密后的密码校验，内容一致就返回true,不一致就返回false
        return equals(tokenCredentials, accountCredentials);
    }
}
```

### 第一种方式：配置Bean

```java
/**
 * Shiro自带密码管理器
 *
 * @return HashedCredentialsMatcher
 */
@Bean
public HashedCredentialsMatcher hashedCredentialsMatcher() {
  //Shiro自带加密
  HashedCredentialsMatcher credentialsMatcher = new HashedCredentialsMatcher();
  //散列算法使用md5
  credentialsMatcher.setHashAlgorithmName("md5");
  //散列次数，2表示md5加密两次
  credentialsMatcher.setHashIterations(2);
  credentialsMatcher.setStoredCredentialsHexEncoded(true);
  return credentialsMatcher;
}

/**
 * 将自己的身份验证器验证方式加入容器
 *
 * @return CustomRealm
 */
@Bean
public CustomRealm shiroRealm() {
  CustomRealm shiroRealm = new CustomRealm();
  //shiroRealm.setCacheManager(cacheManager());
  //加入密码管理
  //shiroRealm.setCredentialsMatcher(hashedCredentialsMatcher());//Shiro自带密码管理器
  shiroRealm.setCredentialsMatcher(new CustomCredentialsMatcher());//自定义密码管理器
  return shiroRealm;
}
```

### 第二种方式：实现AuthorizingRealm类setCredentialsMatcher方法

```java
public class CustomRealm extends AuthorizingRealm {
  ...
  ...
  ...
    /**
     * 设置自定义认证加密方式
     *
     * @param credentialsMatcher 默认加密方式
     */
    @Override
    public void setCredentialsMatcher(CredentialsMatcher credentialsMatcher) {
    		//自定义认证加密方式
        CustomCredentialsMatcher customCredentialsMatcher = new CustomCredentialsMatcher();
        // 设置自定义认证加密方式
        super.setCredentialsMatcher(customCredentialsMatcher);
    }
}
```

