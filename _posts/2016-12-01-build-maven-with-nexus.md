---
layout:     post
title:      "用NEXUS搭建maven私服"
subtitle:   "Build maven repository with NEXUS "
date:       2016-12-01 12:00:00
author:     "沈歌"
header-img: "img/home-bg-o.jpg"
tags:
        - maven
---



下载地址：

nexus2及3官网地址：<https://www.sonatype.com/download-oss-sonatype>
nexues2.5版本(支持jdk1.6的最后一个版本):<http://download.csdn.net/detail/chenbinqun/8133517>

####nexus支持的jdk版本：

| nexus版本 | jdk版本|
|:-------|:---------| 
| nexus2.5及以前 | jdk1.6  |
| nexus2.6及以后的2版本| jdk1.7或jdk1.8|
| nexus3版本| jdk1.8|

注意：不支持OpenJDK，仅支持Oracle JDK。jdk下载地址：<http://www.oracle.com/technetwork/java/javase/archive-139210.html>

**参考：**

2.6及以后的2版本：<http://books.sonatype.com/nexus-book/reference/_prerequisites_and_preparation.html#_java_runtime>

3版本：<https://books.sonatype.com/nexus-book/3.0/reference/install.html#installation-java>

####使用非全局jdk

> 有时候会有这样的需求，全局使用一个版本的jdk，nexus使用另一个版本的jdk，则需要使用配置来修改nexus内的jdk版本。

nexus2：
打开``bin/jsw/conf/wrapper.conf``修改称如下：

```
# Set the JVM executable 

# (modify this to absolute path if you need a Java that is not on the OS path)

wrapper.Java.command=/home/work/local/jdk1.6.0_45

```

nexus3:
    打开``bin/nexus``修改称如下：

    ```
    INSTALL4J_JAVA_HOME_OVERRIDE=/usr/lib/jvm/java-8-oracle
    ```
    
    ####可能因为jdk引起的问题：

    **问题1：**

    *报错* 

    ```
    jvm 3    | Error: failed /home/work/local/jre1.8.0_111/lib/amd64/server/libjvm.so, because /lib64/tls/libc.so.6: version `GLIBC_2.4' not found (required by /home/work/local/jre1.8.0_111/lib/amd64/server/libjvm.so)
    ```

    *解决*：没有GLIBC_2.4,系统版本不支持jdk1.7及以上。要么换jdk版本，要么升级系统。详细请参考[invoke jdk1.7 报错 “/lib/tls/libc.so.6: version `GLIBC_2.4' not found”](http://blog.csdn.net/zhouzihan520xj/article/details/44126919)
    
    ####安装nexus详细步骤
    
    1. 到官网下载压缩包：如``nexus-2.5.1-01-bundle.tar.gz``.
    
    2. 到达指定目录，解压：``tar -zxvf nexus-2.5.1-01-bundle.tar.gz``,会发现多出两个目录来，`nexus-2.5.1-01`放置启动程序，``sonatype-work``放置maven依赖包。
    
    3. 请先确认jdk版本号与nexus版本号一致。启动到目录``nexus-2.5.1-01/bin``,执行命令``./nexus start``（包括各类stop、restart等）
    
    4. 到log目录查看log：log目录``nexus-2.5.1-01/logs/wrapper.log``
    
    5. 访问nexus：默认端口为：8081， 访问链接为``域名:8081/nexus``,登录帐号密码：``admin/admin123``
    
    6. 修改默认端口：地址：``nexus-2.5.1-01/bin``,修改配置``application-port=8081``,修改完重启
    
    ####配置仓库
    
    Nexus的仓库分为这么几类：
    
    hosted 宿主仓库：主要用于部署无法从公共仓库获取的构件（如 oracle 的 JDBC 驱动）以及自己或第三方的项目构件；
    
    proxy 代理仓库：代理公共的远程仓库；
    
    virtual 虚拟仓库：用于适配 Maven 1；
    
    group 仓库组：Nexus 通过仓库组的概念统一管理多个仓库，这样我们在项目中直接请求仓库组即可请求到仓库组管理的多个仓库。
    
    > 默认的代理仓库使用的是apache的maven库，对于公司内的人来说，很多服务器没有外网访问权限，所以需要换成公司自己的maven仓库。
    
    ######代理maven仓库
    
    为了更好的使用 Nexus 的搜索，我们可以设置所有 proxy 仓库的 Download Remote Indexes 为 true，即允许下载远程仓库索引。
    
    而真正的jar包，则在代码中通过maven私服拉取的时候才会下载到本地服务器中。
    
    
    
    关于配置部分，可以参考 *使用Nexus搭建Maven私服<http://www.voidcn.com/blog/zpf336/article/p-5772708.html>*,里面图文并茂，写的很好，虽然只是windows平台的安装过程，但是很有参考价值。
    
    --------------------------------------------------------
    ####settting.xml
    
    关于setting.xml，请参考apache的官方文档，我认为查文档最好查最原生的文档，而不是别人理解之后写出来的博客，<http://maven.apache.org/settings.html>, 中文翻译：<http://www.cnblogs.com/yakov/archive/2011/11/26/maven2_settings.html>
    
    questions：
    
    1. Why maven is downloading metadata every time? 每次用maven编译代码，总是会有一些Download，比如：
    
    ```
    :call <SNR>114_SparkupNext()
    Downloading:  http://download.java.net/maven/2/org/apache/maven/plugins/maven-metadata.xml
    ```

**简要答案**：因为setting中的配置

```
:call <SNR>114_SparkupNext()
<repositories>
    <repository>
            <id>central</id>
                    <url>http://gotoNexus</url>
                            <snapshots>
                                        <enabled>true</enabled>
                                                    <updatePolicy>always</updatePolicy>
                                                            </snapshots>
                                                                    <releases>
                                                                                <enabled>true</enabled>
                                                                                            <updatePolicy>daily</updatePolicy>
                                                                                                    </releases>
                                                                                                        </repository>
</repositories>
```
:call <SNR>114_SparkupNext()

里面的updateProlicy告诉maven多久联系远程maven repo，如示例中，snapshots设置为always的时候，每次更新snapshots类型的maven依赖，都会到远程仓库中检查是否有更新。

详细参考答案：<http://stackoverflow.com/questions/16421454/why-maven-is-downloading-metadata-every-time>
```
```
    ```
    ```]
    ```
    ```
    ```
    ```
```
```
