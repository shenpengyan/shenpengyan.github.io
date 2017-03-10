---
layout:     post
title:      "Spring Java JDBC连接Oracle、Mysql数据库"
subtitle:   "Jdbc Connect Url With Spring(Oracle And Mysql)"
date:       2017-03-10 22:14:00
author:     "沈歌"
header-img: "img/home-bg-o.jpg"
tags:
- jdbc
- Spring
- dbcp
---



开发Java应用程序，时常会有创建数据库连接的需求，但是由于不同数据库的连接规则不同，容易走弯路，在这里总结一下Oracle和Mysql的连接规则。



### 1. Spring中使用DBCP连接池的配置


ORACLE：

```
		<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close"
			p:url="${db.url}" p:username="${db.username}" p:password="${db.password}">
			<property name="driverClassName">
				<value>oracle.jdbc.driver.OracleDriver</value>
			</property>
			<property name="validationQuery">
				<value>select 1 from dual</value>
			</property>
			<property name="testOnBorrow" value="true"/>
			<property name="maxActive" value="${bidb.maxActive}"/>
			
		</bean>
		
		<bean id="biAdminJdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
			<property name="dataSource" ref="dataSource" />
		</bean>

```

MySQL:

```
	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close" p:url="${db.url}"
		p:username="${db.username}" p:password="${db.password}">
		<property name="driverClassName">
			<value>com.mysql.jdbc.Driver</value>
		</property>
		<property name="validationQuery">
			<value>SELECT 1</value>
		</property>
		<property name="testOnBorrow" value="true" />
		<property name="maxActive" value="${db.maxActive}" />
	</bean>
	<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
		<property name="dataSource" ref="dataSource" />
	</bean>

```

说明：

- maxActive为可以从对象池中取出的对象最大个数，为0则表示没有限制，默认为8

- 设置validationQuery和testOnBorrow的意义在于数据库连接中断，当使用DBCP时，数据库连接因为某种原因断掉后，再从连接池中取得连接又不进行验证，这时取得的连接实际已经时无效的数据库连接。validationQuery为取得验证开关，使用validationQuery来做验证。

### 2.db.url的配置方式

Mysql

```
jdbc:mysql://<host>:<port>/<schema>?useUnicode=true&characterEncoding=UTF-8
```

Oracle


Java JDBC Thin Driver 连接 Oracle有三种方法。 

格式一: 使用ServiceName方式: 

```
jdbc:oracle:thin:@//<host>:<port>/<service_name> 
例 jdbc:oracle:thin:@//xxx.xxx.xxx.xxx:1526/CDEV 
```
@后面有//, 这是与使用SID的主要区别。（11g在@后不加//也OK）
这种格式是Oracle 推荐的格式.
因为对于集群来说，每个节点的SID是不一样的，而SERVICE NAME可以包含所有节点。 

格式二: 使用SID方式: 

```
jdbc:oracle:thin:@<host>:<port>:<SID> 
例 jdbc:oracle:thin:@xxx.xxx.xxx.xxx:1526:CDEV2
```
格式三：使用TNSName方式: 

```
jdbc:oracle:thin:@<TNSName> 
例 jdbc:oracle:thin:@CDEV 
```

注意，ORACLE从10.2.0.1后支持TNSNames
 
> 笔者在写连接属性的时候，被ServiceName和SID搞得晕头转向，故记录一下。

