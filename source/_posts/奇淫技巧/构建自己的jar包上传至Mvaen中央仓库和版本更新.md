---
title: 构建自己的jar包上传至Mvaen中央仓库和版本更新
tags:
  - Maven
categories:
  - 奇淫技巧
---
# 构建自己的jar包上传至Mvaen中央仓库和版本更新

一直羡慕别人制造轮子，开源项目，供别人使用；我也想这样，可以自己才疏学浅，本次就将自己写小工具上传到Maven的中央仓库。

一步一步详细教程演示如下：

**开始要注意这个几个Maven相关地址**：

- **工单管理**：[https://issues.sonatype.org](https://issues.sonatype.org/)

> 就是申请上传资格和groupId 的地方,注册账号、创建和管理issue，Jar包的发布是以解决issue的方式起步的

- **构件仓库** : https://oss.sonatype.org/#welcome

> 把jar包上传到这里，Release 之后就会同步到maven中央仓库。

- **仓库镜像**: http://search.maven.org/

> 最终工件可以在这里搜索到。

# 创建一个新的项目Issue

**第一步：注册工单管理(register sonatype)**

由于[Maven 中央仓库](https://maven.apache.org/repository/)是由企业*Sonatype,Inc.*负责运营维护的，因此你需要先前往 https://issues.sonatype.org/secure/Signup!default.jspa 注册一个账号（如果你已经有了账号，可以跳过此步骤）。

**第二步：创建工单（nexus)**

点击header 头 “create”按钮创建一个工单，主要用途注册你上传jar包基本信息，主要groupid，通过审核有两个目的：1.防止重复，约束groupid规范，定义grupid最好有所属的域名

登录了账号之后，你需要新建一个*Issue*。*Issue*是问题的意思，相当于你提交了一个工单给 Sonatype 的工作人员，申请开通Maven中央仓库的发布权限。

点击顶部的[【新建】](https://issues.sonatype.org/secure/CreateIssue!default.jspa)按钮，填写申请表单，即可提交申请。

![](https://i.loli.net/2020/06/10/NycF6o2IwB9TZi8.png)

+ Project URL：项目地址。
+ SCM URL ：项目clone的git地址。
+ Summary：你的jar包自己起个名字（无要求）。
+ Group Id：项目标识。

【提醒】确group id对应的是域名，推荐使用个人域名。

**注意**：*groupId* 不能随意填写。

- 如果你（或所属机构）拥有自己的域名，可以填写自己的域名。工作人员在审核时，会反馈给你，要求验证域名的所有权，你可以采取如下两种方式之一完成验证：
  1. 为域名添加一个指向 Issue编号 的TXT记录； 
  2. 将域名的访问重定向到你的项目主页地址（*Project URL*）。
- 如果你没有自己的域名，你只能使用项目托管方的域名。例如 你的项目在Github ，主页地址为 https://github.com/username/projectName，那么你只能使用 *io.github.username* 或 *com.github.username* 作为*groupId*。

> 点击 Create 之后，你就会进入一个 jira 页面(sonatype 使用jira 来追踪每一个项目进度)，与此同时你注册时使用的邮箱中也会收到一封邮件提示你，任务创建成功,正在等待处理。 

> 提交成功后，你需要耐心等待审核。不过，请不用担心，Sonatype工作人员的审核效率是很高的，工作时间范围内，基本上一小时内就会响应。

> 因为我这边用的是自己的域名，所以我需要为域名添加一个一个指向 Issue编号 的TXT记录（也就是你新建这个问题的地址：https://issues.sonatype.org/browse/OSSRH-58328）。

添加TXT记录之后需要回复工作人员

![](https://i.loli.net/2020/06/11/bJ6GNO1qpMTV9iA.png)

1. 新建工单之后，工作人员会回复你，需要审核验证你的域名所有权

   ![](https://i.loli.net/2020/06/11/OWrSZCgvmKNnjRc.png)

2. 我这边添加完TXT记录之后，回复工作人员

   ![](https://i.loli.net/2020/06/11/8kbcLDwT1nVGdHB.png)

3. 工作人员审核验证完之后会回复你，让你可以上传构建你的jar包了，当工单状态变为“已解决”，就可以上传maven项目了

   ![](https://i.loli.net/2020/06/11/9yD17hUrjNpR2P4.png)

## gpg 环境安装

**win下载地址**：https://www.gnupg.org/download/

![](https://i.loli.net/2020/06/09/T5NoarJPWxu6OLy.png)

安装就下一步下一步...

**Mac 安装**：`brew install gpg`

**win或者mac安装之后在命令行输入：**

```shell
#gpg 常用命令
gpg --help #帮助
gpg --version #查看版本
gpg --list-keys #查看已生成的密钥
gpg --delete-key [用户ID] #删除密钥
gpg --gen-key #生成密钥对
#第一步：输入用户名
#第二步：输入邮箱地址
#第三步：输入密码
#之后就生成好了，可以用gpg --list-keys查看已生成的密钥

#上传公钥到服务器
gpg --keyserver hkp://pool.sks-keyservers.net --send-keys ${公钥ID} #将公钥发布到 PGP 密钥服务器
gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys ${公钥ID} #查询公钥是否发布成功

gpg --keyserver hkp://pool.sks-keyservers.net:11371 --send-keys 3C50A16F87687255F1AB96BC7E7CB475C5DF4735
gpg --keyserver hkp://keyserver.ubuntu.com:11371 --send-keys 3C50A16F87687255F1AB96BC7E7CB475C5DF4735
gpg --keyserver hkp://keys.gnupg.net:11371 --send-keys 3C50A16F87687255F1AB96BC7E7CB475C5DF4735

#查看是否上传成功
gpg --keyserver hkp://pool.sks-keyservers.net:11371 --recv-keys ${公钥ID}

gpg --keyserver hkp://pool.sks-keyservers.net:11371 --recv-keys 3C50A16F87687255F1AB96BC7E7CB475C5DF4735
gpg --keyserver hkp://keyserver.ubuntu.com:11371 --recv-keys 3C50A16F87687255F1AB96BC7E7CB475C5DF4735
gpg --keyserver hkp://keys.gnupg.net:11371 --recv-keys 3C50A16F87687255F1AB96BC7E7CB475C5DF4735

#导出密钥对
gpg --armor --output "输出文件名" --export "用户ID"

gpg --armor --output C:\Config\gpg\test-public-key.asc --export  "用户ID"

gpg --armor --output C:\Config\gpg\test-private-key.asc --export-secret-keys

#导入密钥
gpg --import test-public-key.asc #导入公钥
gpg --import test-private-key.asc #导入私钥
```

1. 输入`gpg --gen-key`生成密钥对
2. 将密钥对公钥上传到服务器

## 配置Maven

**需要修改的 Maven 配置文件包括：`setting.xml`（全局级别）与 `pom.xml`（项目级别）**

### setting.xml（全局级别）

> 这里的username是你一开始注册sonatype账号的username，而不是注册时填写的邮箱。

```xml
    <servers>
        <server>
            <id>ossrh</id>
            <username>用户名</username>
            <password>密码</password>
        </server>
    </servers>
```

使用自己注册的 Sonatype 账号的用户名与密码来配置以上 server 信息。

此处 id `ossrh` 应和下面 `pom.xml` 中 snapshotRepository 和 repository 里面的 id 保持一致。

### pom.xml（项目级别）

#### 说明

```xml
<name>ali-tools</name>
<url>https://github.com/452425952/ali-tools</url>
<description>Ali Tools project for Spring Boot</description>
```

#### 证书

```xml
<licenses>
    <license>
        <name>The Apache Software License, Version 2.0</name>
        <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
        <distribution>repo</distribution>
    </license>
</licenses>
```

#### scm软件配置管理

```xml
<scm>
    <connection>https://github.com/452425952/ali-tools.git</connection>
    <developerConnection>https://github.com/452425952/ali-tools.git</developerConnection>
    <url>https://github.com/452425952/ali-tools</url>
</scm>
```

#### 开发者信息

```xml
<developers>
    <developer>
        <id>sky-0914</id>
        <name>sky-0914</name>
        <email>sky-0914@qq.com</email>
        <roles>
            <role>Developer</role>
        </roles>
        <timezone>+8</timezone>
    </developer>
</developers>
```

#### 打包配置,由于发布到maven中央仓库会要求我们在上传jar到同时，必须同步发布对应到Javadoc、source、asc(利用gpg生成到校验)，所以需要在maven中添加以下构建插件

```xml
<build>
    <plugins>
        <!-- 打包时跳过测试 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.18.1</version>
            <configuration>
                <skipTests>true</skipTests>
            </configuration>
        </plugin>

        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>${java.version}</source>
                <target>${java.version}</target>
                <encoding>UTF-8</encoding>
            </configuration>
        </plugin>
        <!-- 以下都为上传jar默认配置不做修改 -->
        <plugin>
            <groupId>org.sonatype.plugins</groupId>
            <artifactId>nexus-staging-maven-plugin</artifactId>
            <version>1.6.8</version>
            <!-- autoReleaseAfterClose的值为true，则脚本会自动完成在平台上close、release的操作，至此你将成功发布了 -->
            <extensions>true</extensions>
            <configuration>
                <serverId>ossrh</serverId>
                <nexusUrl>https://oss.sonatype.org/</nexusUrl>
                <autoReleaseAfterClose>true</autoReleaseAfterClose>
            </configuration>
        </plugin>

        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-release-plugin</artifactId>
            <version>2.5.3</version>
            <configuration>
                <autoVersionSubmodules>true</autoVersionSubmodules>
                <useReleaseProfile>false</useReleaseProfile>
                <releaseProfiles>release</releaseProfiles>
                <goals>deploy</goals>
            </configuration>
        </plugin>

        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-gpg-plugin</artifactId>
            <version>1.5</version>
            <executions>
                <execution>
                    <id>sign-artifacts</id>
                    <phase>verify</phase>
                    <goals>
                        <goal>sign</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>

        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>2.2.1</version>
            <executions>
                <execution>
                    <id>attach-sources</id>
                    <goals>
                        <goal>jar-no-fork</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>

        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-javadoc-plugin</artifactId>
            <version>2.9</version>
            <executions>
                <execution>
                    <id>attach-javadocs</id>
                    <goals>
                        <goal>jar</goal>
                    </goals>
                    <!-- JDK8必须使用下面的配置 -->
                    <configuration>
                        <encoding>UTF-8</encoding>
                        <charset>UTF-8</charset>
                        <additionalparam>-Xdoclint:none</additionalparam>
                    </configuration>

                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

#### 上传打包文件配置

> 这里的id需要和setting.xml中的 server 标签中的id对应

```xml
<distributionManagement>
    <snapshotRepository>
        <id>ossrh</id>
        <url>https://oss.sonatype.org/content/repositories/snapshots</url>
    </snapshotRepository>
    <repository>
        <id>ossrh</id>
        <url>https://oss.sonatype.org/service/local/staging/deploy/maven2</url>
    </repository>
</distributionManagement>
```

## 发布操作

> 执行以下命令进行发布（如果 pom.xml 中 autoReleaseAfterClose 的值为true，则脚本会自动完成在平台上close、release的操作，至此你将成功发布了，否则我们继续查看第2步）

```shell
mvn clean deploy -X # -X可以查看详细信息
```

**打包过程遇到问题：**

1. javadoc格式错误，需要将错误问题解决。
2. failureMessage  No public key，这个是gpg秘钥没有上传成功导致，我在实际操作时，明明上传成功，但是还是这样提示，只能多打包几次就可以了，不知道什么原因。解决方法: 如果你知道哪个密钥服务器会被查询，你可以直接在那里上传你的密钥。
3. 实验过程中我的实际执行结果是（注意可能会超时，多试几次就好）

**登录https://oss.sonatype.org，然后选择staging Repositories**

![](https://i.loli.net/2020/06/11/Ggdz5nprUNHfjJT.png)

**如果在pom文件中的 autoReleaseAfterClose的值没有设置的话，需要手动发布的组件，依次执行Close、Release操作**

![](https://i.loli.net/2020/06/11/xDv8NKnGCl9HQXy.png)

**如果Release成功，并且你是首次发布组件，则需要到你创建到issue中回复 “我已经成功发布该组件”，经历大概2小时后，该组件将会同步到对应的maven仓库 。**

![](https://i.loli.net/2020/06/11/X8nVBHkuAaxfspq.png)

**如果Close或者Release不成功，你可以如下图操作查看原因并谷歌定位，或者回复咨询**

![](https://i.loli.net/2020/06/11/dY13vnJi4oeNUOr.png)

### 查找开源库 

注意开源库发布出去之后，还是需要等待一段时间，才能在这里 https://search.maven.org/  查找我们的开源库，以及查看如何依赖自己的开源库。

![](https://i.loli.net/2020/06/11/H6F5ywXisr8VAlj.png)

# 后续发布到中央仓库

首次发布都是比较痛苦的，我们首次进行发布，实际上遇到很多问题，而且还要Sonatype进行验证。第一次总是比较麻烦的，所谓万事开通难拿，接下来后续的发布动作，其实并没有那么麻烦了

发布命令和原来的一样；如果是和第一次发布在同一台电脑（并且没有重装过了，gpg密钥对还在），那么直接执行

```shell
mvn clean deploy -X
```

如果换电脑发布了，那么就需要将原来的gpg密钥对（公钥和私钥）导出来保存之后，重新导入到新电脑上（上述内容介绍了gpg常用命令）

```shell
#导出密钥对
gpg --armor --output "输出文件名" --export "用户ID"

gpg --armor --output C:\Config\gpg\test-public-key.asc --export  "用户ID"

gpg --armor --output C:\Config\gpg\test-private-key.asc --export-secret-keys

#导入
gpg --import test-public-key.asc #导入公钥
gpg --import test-private-key.asc #导入私钥
```

---



参考文章：

https://blog.csdn.net/xp_lx1/article/details/104722832?utm_medium=distribute.pc_relevant.none-task-blog-baidujs-1

http://www.itmuch.com/work/sonartype-deploy-mvn-depencency-to-maven-central/

gpg密钥对过期解决方案：http://blog.didispace.com/maven-gpg-expire/



# gpg: signing failed: Inappropriate ioctl for device

最近上传jar到中央仓库的时候，遇到一个问题：

> gpg: signing failed: Inappropriate ioctl for device

原因是 gpg 在当前终端无法弹出密码输入页面。

解决办法很简单：

```bash
export GPG_TTY=$(tty)
```

重新执行，发现会弹出一个密码输入界面。

# Mac打maven包——gpg: 签名时失败

gpg: signing failed: Inappropriate ioctl for device
原因是GPG版本是最新版本，需要在.gnupg文件夹下增加两个配置，
mac下在~/.gnupg这两个文件夹下修改
gpg.conf和gpg-agent.conf两个文件，
在gpg-agent.conf中新建一行添加

```
allow-loopback-pinentry
```

在gpg.conf添加

```
use-agent 
pinentry-mode loopback 
```

保存就可以OK了。
