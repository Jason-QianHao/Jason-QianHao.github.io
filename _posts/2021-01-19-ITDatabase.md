---
title: 在线资源共享ITDatabase
tags: Project
---

*由简书搬迁而来[**原文链接**](https://www.jianshu.com/p/9e4868a3fd70)*

# ITDatabase 

## 系统总体架构

<img src="/../assets/ITDatabase/1240-20210704082006202.jpeg" alt="img" style="zoom:50%;" />

## 系统流程图

<img src="/../assets/ITDatabase/1240-20210704082006160.jpeg" alt="img" style="zoom: 67%;" />

## 主页面展示

![img](/../assets/ITDatabase/1240-20210704082006191.jpeg)

## ITDatabase-Web工程

### Maven依赖

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>1.5.2.RELEASE</version>
  <relativePath /> <!-- lookup parent from repository -->
</parent>
<dependencies>
  <!-- 集成web -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <!-- 集成mybatis -->
  <dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.1.1</version>
  </dependency>
  <!-- 集成mysql -->
  <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.46</version>
  </dependency>
  <!-- 集成freemarker -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
  </dependency>
  <!--Lombok -->
  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
  <version>1.18.6</version>
  </dependency>
  <dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
  <version>4.9</version>
  </dependency>
  <!-- redis -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
  </dependency>
  <!-- PageHelper -->
  <dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.10</version>
  </dependency>
</dependencies>
```

### 数据库设计

（1）用户表users

```mysql
#用户表

CREATE TABLE `users`(

`id` int(11) NOT NUll auto_increment COMMENT'主键（自增长）',

`username` VARCHAR(50) NOT NULL COMMENT'用户名',

`password` VARCHAR(32) NOT NULL COMMENT'密码，加密存储',

`phone` VARCHAR(20) DEFAULT NULL COMMENT'手机号',

`email` VARCHAR(50) DEFAULT NULL COMMENT'邮箱',

`openId` VARCHAR(100) DEFAULT NULL COMMENT'登录Id',

`created` TIMESTAMP not NULL DEFAULT CURRENT_TIMESTAMP COMMENT'自动插入，创建时间',

`updated` TIMESTAMP not null DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT'自动插入，修改时间',

PRIMARY KEY(`id`),

UNIQUE KEY `username` (`username`) USING BTREE,

UNIQUE KEY `phone` (`phone`) USING BTREE,

UNIQUE KEY `email` (`email`) USING BTREE

)ENGINE=INNODB COMMENT='用户表';
```

（2）资源内容表MessageInfo

```mysql
#资源内容表

CREATE TABLE `message_info`(

`id` int(11) NOT null auto_increment COMMENT'主键（自增长）',

`message_name` VARCHAR(150) NOT null COMMENT'资源名称',

`message_url` VARCHAR(100) NOT NULL COMMENT'资源url',

`message_typeId` int NOT NULL COMMENT'资源类型Id',

`message_pwd` VARCHAR(100) NOT NULL COMMENT'资源提取码',

`message_author` VARCHAR(100) NOT null COMMENT'上传者',

`created` TIMESTAMP not NULL DEFAULT CURRENT_TIMESTAMP COMMENT'自动插入，创建时间',

`updated` TIMESTAMP not null DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT'自动插入，修改时间',

PRIMARY KEY(`id`),

KEY `message_name` (`message_name`) USING BTREE,

INDEX `message_typeId` (`message_typeId`) USING BTREE,

KEY `message_author` (`message_author`),

UNIQUE KEY `message_url` (`message_url`)

);
```

（3）资源类型表MessageType

```mysql
#资源种类表

CREATE TABLE `message_type`(

`id` int(11) NOT NULL auto_increment COMMENT'主键（自增长）',

`type_name` VARCHAR(30) NOT NULL COMMENT'资源类型名',

PRIMARY KEY(`id`),

UNIQUE KEY `type_name` (`type_name`)

);
```

  资源类型表的id即为资源内容表的message_typeId。

### 实体类设计

（1）userEntity

​    和数据库字段名保持一致

（2）MessageInfoEntity

​    除保留数据库字段名外，添加**typeName字段**用于联合查询

（3）MessageTpyeEntity

​    和数据库字段名保持一致

### 配置文件

```yaml
server:
 port: 80
 context-path: /
mybatis:
 configuration:
# 开启驼峰uName自动映射到u_name
   map-underscore-to-camel-case: true
pagehelper:
# 方言配置
 helperDialect: mysql
# 3.3.0版本可用 - 分页参数合理化，默认false禁用
# 启用合理化时，如果pageNum<1会查询第一页，如果pageNum>pages会查询最后一页
# 禁用合理化时，如果pageNum<1或pageNum>pages会返回空数据
 reasonable: true
 supportMethodsArguments: true
 params: count=countSql
spring:
# mysql
 datasource:
   url: jdbc:mysql://localhost:3306/itdatabase
   username: root
   password: root
   driver-class-name: com.mysql.jdbc.Driver
 freemarker:
   allow-request-override: false
   cache: true
   check-template-location: true
   charset: UTF-8
   content-type: text/html
   expose-request-attributes: false
   expose-session-attributes: false
   expose-spring-macro-helpers: false
   suffix: .ftl
   template-loader-path: classpath:/templates/
#redis
 redis:
   host: localhost
   password: 123457
   port: 6379
   pool:
    max-idle: 100
    min-idle: 1
    max-active: 1000
    max-wait: -11
```

### 工程结构截图

<img src="/../assets/ITDatabase/1240-20210704082006194.png" alt="img" style="zoom: 67%;" />

## 系统介绍

  本系统是在线资源共享系统，最早版本采用SSM框架，即Spring+SpringMVC+Mybatis进行设计，后续考虑到Springboot配置项目的便捷性，经过不断优化最终采用Springboot+Mybatis进行系统设计。系统设计依然采用SpringMVC模式，分Controller层、Service层和Dao层。

  系统设计从数据库设计开始，数据库表分为用户表、资源内容表和资源类型表。用户表用来存储注册的用户信息，有用户名、邮箱、密码等字段，其中Id为主键，用户名、手机号和邮箱加之唯一索引。资源内容表用来存储用户上传的资源信息，有资源名称、资源url、资源类型Id等字段，其中资源url为唯一索引，资源类型Id和资源类型表的Id相关联用来查询种类名称。资源类型表用来存储资源类型名称，已经存在10种在表中，用户也可以自定义类型上传至数据库，其中类型名称为唯一索引。

  接着是对实体类和Dao层的设计，实体类的设计只有在资源实体类中添加了typeName字段，用来将联合查询的结果存储在对象中，其他字段均为数据库映射，在配置文件中开启了Mybatis的“驼峰命名”映射规则。Dao层这里即为Mybatis的mapper层。根据数据表的分类，同样进行UserMapping、MessageInfoMapping和MessageTypeMapping的对应SQL封装。

  在Service层设计时，除了用户服务和资源服务外，加入Redis服务用来存储用户的登录信息，这是在用户禁用Cookie后的补偿措施。用户服务封装注册、登录、Redis写入和查询。注册时，使用MD5算法对用户的密码进行加密再存储。登录服务需要验证用户名、手机和邮箱三种登录方式，并在登录成功后将"OpenId"作为键，用户的openid作为值加入Cookie和Reids。资源服务封装读取所有资源、添加资源、读取所有资源类型、删除资源、修改资源操作。特别对于自定义添加类型操作，在前端设计只有在选中“自定义”类型时，显示新增资源名称输入框；在后端需要判断新增资源名是否为空或重复等进行相应处理。

  对于Controller层的设计，考虑到每次返回主页需要查询资源和分页，添加和修改资源页面需要查询所有资源类型，使用BaseController类，封装三个操作，其他Controller对其继承即可。分页采用PageHelper进行分页处理，每页记录初步设定为5条。在用户控制类中，访问主页面会检测Cookie和Redis的缓存，判断是否需要登录。在资源控制类中，处正常操作外，还拥有对添加资源的提交通过Session判断是否重复提交，对分页跳转进行控制。

  最后通过@SpringbootApplication注解和@MapperScan启动工程即可。

## 一些开发笔记

1 Mybatis的Mapping接口，多参数时，一定要使用@Param，并且对象的字段使用user.name形式。

2 前端input控件不输入值时，后端注意区别""和null

3 前端Input控件配置了value默认值后，不可用在配置Placeholder，否则默认值无法传到后端，为null

4 PageHelper后才能查数据库，并自动绑定查询结果的List。所以其他的List一定要再pagehelper之前定义！！
