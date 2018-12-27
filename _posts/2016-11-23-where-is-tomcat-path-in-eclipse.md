---
layout:     post
title:      "eclipse中的tomcat默认部署到了哪里"
subtitle:   "where is tomcat path in eclpise"
date:       2016-11-23 12:00:00
author:     "沈歌"
header-img: "img/home-bg-o.jpg"
tags:
        - eclipse
        - tomcat
---

> 笔者在使用eclipse+tomcat做本地调试，项目没跑起来，原因就很奇怪啊（某前辈说过：奇怪源于无知），然后就想它究竟是把项目放到哪个目录下呢，我的tomcat/webapps目录下并没有啊。

#### 默认部署到了哪里

eclipse并不像MyEclipse默认将项目部署到tomcat安装目录下的webapps中，而默认部署到工作目录(workspace)下的`.metadata/.plugins/org.eclipse.wst.server.core/tmp0/wtpwebapps`中。（tmp0、tmp1的不同是目前这个server容器的顺序）

#### 如何修改

为了使项目默认部署到tomcat安装目录下的webapps中，show view->servers->找到需要修改的tomcat->右击

1. 停止eclipse内的tomcat服务器（stop）
2. 删除该容器中部署的项目(add and remove)
3. 清除该容器相关数据(clean)
4. 打开tomcat的修改界面(open)
5. 找到servers location, 选择第二个(User tomcat Installation)
6. 修改deploy path为webapps
7. 保存关闭

#### tomcat信息配置页详解

> 核心：`Configuration Path`配置了本页面获取配置信息内容的地址，在tomcat run的时候，配置文件会同步到`Server path/conf` 目录下，部署的文件会部署到`Deploy path`目录下。

![image](https://shenpengyan.github.io/img/in-post/where-is-tomcat-path-in-eclipse/eclipse-tomcat1.png)
 
页面打开方式：
show view->servers->找到需要修改的tomcat->右击+open或者双击

###### General Information
包含一些基本信息

注意**Configuration path**,默认配置的是work
space中的配置文件，而不是tomcat目录下的配置文件。
每个tomcat一个配置文件，会在tomcat run的时候，将配置文件信息与`Server Path/conf`中相关文件保持同步。

文件目录示例如下：

```
+- Server
    +- Tomcat v7.0 Server at localhost-config
        --catalina.policy
        --catalina.properties
        --context.xml
        --server.xml
        --tomcat-users.xml
        --web.xml
```

###### Server Locations

Server Locations包含server path和Deploy path，
1. Server Path
- Use workspace metadata 默认位置(`.metadata/.plugins/org.eclipse.wst.server.core/tmp0`)

- Use Tomcat installation(配置的本地tomcat目录)
如：`/Users/shenpengyan/Documents/apache-tomcat-7.0.70/webapps`

- Use custom location（使用任意其他目录）

2. Deploy path:

和Server Path关联，设置serverpath的一个子目录，默认为(wtpwebapps)

###### Server Options

###### Publishing

###### Timeouts (启动和关闭超时)
在启动和关闭时，如果超过这里设定的时间就会报错。启动时如果项目启动比较耗时，可以调高start timeout。

###### Ports（端口）
这里列出了启动时候需要占用的端口号，如果启动时候显示端口号被占用。可以到这里去看，把相关端口占用清理掉，或者到对应的Configuration Path中的server.xml文件中修改对应的端口。

标准tomcat需要占用三个端口

端口 | 描述
---|---
Tomcat admin port | tomcat启动关闭时用的端口
HTTP/1.1 | http请求占用的端口
APJ/1.3 | apj请求占用的端口

###### MIME Mapping（MIME类型对照关系，更改会显示在web.xml中）

#### tomcat内两种添加Web Modules的方式

1. 添加web project
- view->servers->找到需要修改的tomcat->右击-> add and remove

- tomcat配置信息页，切换Modules tab。

![image](https://shenpengyan.github.io/img/in-post/where-is-tomcat-path-in-eclipse/eclipse-tomcat-2.jpg)
    
    如图，按钮`Add Web Module`用来添加内部project。

2. 添加外部web project

    如上图，按钮`Add External Web Module`用来添加外部project。使用这种方式，可以直接把maven web项目中的target目录下的产出放进去启动。