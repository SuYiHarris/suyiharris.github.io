---
title:  "SpringBoot的application常用配置"
subtitle: "It's always a bit messy"
author: "suyi"
date:   2023-10-24 12:12:12
---

## server配置

~~~yaml
#1.配置tomcat访问端口
server:
  port: 7778
#2.配置http://localhost:7778/aa/bb.html这个访问地址的意思
  servlet:
    context-path: /aa
#3.Session设置存活时间是30分钟
  session-timeout: 30
#4.tomcat启动的最大线程数，即同时处理的任务个数，默认值为200
  tomcat.max-threads: 0
#5.Tomcat在解析参数的时候没有使用正确的编码格式（UTF-8）去解码
  tomcat.uri-encoding: UTF-8
~~~



## spring配置

~~~yaml
spring:
  application:
#1.服务名称信息
      name: mimi
#2.配置启动banner的编码和文字
  banner:
    charset: UTF-8
    location: classpath:banner.txt
#3.spring下的配置数据库
  datasource:
    #1.配置数据库连接池（Druid，），HikariCP 是默认的
    type: com.alibaba.druid.pool.DruidDataSource
    #2.数据库的地址
    url: jdbc:mysql://localhost:3306/testDB?characterEncoding=UTF-8&serverTimezone=UTC
    #3.配置数据库驱动，jdbc
    driver-class-name: com.mysql.cj.jdbc.Driver
    #4.配置数据库连接的账号
    username: root
    #5.配置数据库连接的密码
    password: 123456
#4.（spring:mvc下的）关于springmvc的设置
  mvc:
    #1.静态资源的过滤规则（在静态资源目录下，只有以/resources/开头的才能显示，不然不能显示）
       static-path-pattern: /resources/**
#5.（spring:resources下的）设置静态资源的路径（默认的是classpath:/static,classpath:/public,classpath:/resources,classpath:/META-INF/resource）
  resources:
    static-locations: ["classpath:/static/", "classpath:/public/","classpath:/mapper/"]
#6.thymeleaf的模板配置
  thymeleaf:
      prefix: classpath:/templates
      mode: HTML
      cache: false
      encoding: UTF-8
      #新版本不支持content-type: text/html，故新写法
      servlet:
        content-type: text/html
~~~



​         

## mybatis配置

  

~~~yaml
# mybatis的配置
mybatis:
#1.配置mapper文件放在resource下的mapper中
    mapper-locations: classpath:/mapper/*.xml
#2.配置实体扫描，多个package用逗号或者分号分隔
    type-aliases-package: com.example.demo.entity
#3.配置mybatis日志打印（这个生效后下面logging中的打印失效）
    configuration:
      log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
#4.配置mybatis配置文件
      location: classpath:mybatis/mybatis-config.xml
 #打印等级（com.example.demo.dao包下的日志会打印），不是mybatis子配置
logging:
    level:
      com.example.demo.dao:  debug
~~~

