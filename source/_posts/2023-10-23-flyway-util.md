---
title:  "A flyway Util"
subtitle: "It's always a bit messy"
author: "suyi"
date:   2023-10-23 12:12:12
---

# flyway学习笔记

## 一、flyway工作流程

1. 项目启动，应用程序完成数据库连接池的建立后，Flyway自动运行。
2. 初次使用时，flyway会创建一个 flyway_schema_history 表，用于记录sql执行记录。
3. Flyway会扫描项目指定路径下(默认是 classpath:db/migration )的所有sql脚本，与 flyway_schema_history 表脚本记录进行比对。如果数据库记录执行过的脚本记录，与项目中的sql脚本不一致，Flyway会报错并停止项目执行。
4. 如果校验通过，则根据表中的sql记录最大版本号，忽略所有版本号不大于该版本的脚本。再按照版本号从小到大，逐个执行其余脚本。

## 二、如何使用

### 1、导入maven依赖

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
    <version>6.1.0</version>
</dependency>
```

### 2、在yml中配置

```yml
spring:
  # 数据库连接配置
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/flyway-demo?characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai
    username: root
    password: root
  flyway:
    # 是否启用flyway
    enabled: true
    # 编码格式，默认UTF-8
    encoding: UTF-8
    # 迁移sql脚本文件存放路径，默认db/migration
    locations: classpath:db/migration
    # 迁移sql脚本文件名称的前缀，默认V
    sql-migration-prefix: V
    # 迁移sql脚本文件名称的分隔符，默认2个下划线__
    sql-migration-separator: __
    # 迁移sql脚本文件名称的后缀
    sql-migration-suffixes: .sql
    # 迁移时是否进行校验，默认true
    validate-on-migrate: true
    # 当迁移发现数据库非空且存在没有元数据的表时，自动执行基准迁移，新建schema_version表
    baseline-on-migrate: true
```

### 3、根据配置文件中的SQL脚本文件存放路径，创建对应的目录

resources/db/migration

### 4、添加SQL脚本，创建时注意命名规范

- sql脚本的命名有两种，一种是以V开头的+版本号+两段下划线+文件名.sql，这种以V开头的脚本是只执行一次的，另一种是R开头的，命名方式类似，这种脚本可以重复执行，但不建议修改执行过的sql语句

- V开头的sql文件比R开头的sql文件优先级高

### 5、启动项目后可以在flyway_schema_history表中查看相关语句执行记录

 
![](https://pic.imgdb.cn/item/6537ce38c458853aef669346.jpg)


![](https://pic.imgdb.cn/item/6537ce47c458853aef66cfcb.jpg)


![](https://pic.imgdb.cn/item/6537ce57c458853aef670fae.jpg)




### 6、说明

如果我们修改V2__add_user.sql中的内容，再次执行的话，就会报错，提示信息如下：

 　　[ERROR] Migration checksum mismatch for migration version 2 

　　如果我们修改了R__add_unknown_user.sql，再次执行的话，该脚本就会再次得到执行，并且flyway的历史记录表中也会增加本次执行的记录。 

## 三、Maven插件

### 1、配置

以上步骤中，每次想要migration都需要运行整个springboot项目，并且只能执行migrate一种命令，其实flyway还是有很多其它命令的。maven插件给了我们不需要启动项目就能执行flyway各种命令的机会。

在pom.xml中引入flyway的插件，同时配置好对应的数据库连接。

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.flywaydb</groupId>
            <artifactId>flyway-maven-plugin</artifactId>
            <version>5.2.4</version>
            <configuration>
                <url>jdbc:mysql://localhost:3306/flyway-demo?characterEncoding=utf-8&amp;useSSL=false&amp;serverTimezone=Asia/Shanghai
                </url>
                <user>root</user>
                <password>root</password>
                <driver>com.mysql.cj.jdbc.Driver</driver>
            </configuration>
        </plugin>
    </plugins>
</build>
```


![](https://pic.imgdb.cn/item/6537ce66c458853aef676645.jpg)


### 2、命令说明

1. baseline

   对已经存在数据库Schema结构的数据库一种解决方案。实现在非空数据库新建MetaData表，并把Migrations应用到该数据库；也可以在已有表结构的数据库中实现添加Metadata表。

2. clean

   清除掉对应数据库Schema中所有的对象，包括表结构，视图，存储过程等，clean操作在dev 和 test阶段很好用，但在生产环境务必禁用。

3. info

   用于打印所有的Migrations的详细和状态信息，也是通过MetaData和Migrations完成的，可以快速定位当前的数据库版本。

4. repair

   repair操作能够修复metaData表，该操作在metadata出现错误时很有用。

5. undo

   撤销操作，社区版不支持。

6. migrate

   迁移。

7. validate

   验证已经apply的Migrations是否有变更，默认开启的，原理是对比MetaData表与本地Migrations的checkNum值，如果值相同则验证通过，否则失败。



## 四、知识补充

1. flyway执行migrate必须在空白的数据库上进行，否则报错。
2. 对于已经有数据的数据库，必须先baseline，然后才能migrate。
3. clean操作是删除数据库的所有内容，包括baseline之前的内容。
4. 尽量不要修改已经执行过的SQL，即便是R开头的可反复执行的SQL，它们会不利于数据迁移。
5. 当需要做数据迁移的时候，更换一个新的空白数据库，执行下migrate命令，所有的数据库更改都可以一步到位地迁移过去。

## 五、配置清单

```properties
flyway.baseline-description对执行迁移时基准版本的描述.
flyway.baseline-on-migrate当迁移时发现目标schema非空，而且带有没有元数据的表时，是否自动执行基准迁移，默认false.
flyway.baseline-version开始执行基准迁移时对现有的schema的版本打标签，默认值为1.
flyway.check-location检查迁移脚本的位置是否存在，默认false.
flyway.clean-on-validation-error当发现校验错误时是否自动调用clean，默认false.
flyway.enabled是否开启flywary，默认true.
flyway.encoding设置迁移时的编码，默认UTF-8.
flyway.ignore-failed-future-migration当读取元数据表时是否忽略错误的迁移，默认false.
flyway.init-sqls当初始化好连接时要执行的SQL.
flyway.locations迁移脚本的位置，默认db/migration.
flyway.out-of-order是否允许无序的迁移，默认false.
flyway.password目标数据库的密码.
flyway.placeholder-prefix设置每个placeholder的前缀，默认${.
flyway.placeholder-replacementplaceholders是否要被替换，默认true.
flyway.placeholder-suffix设置每个placeholder的后缀，默认}.
flyway.placeholders.[placeholder name]设置placeholder的value
flyway.schemas设定需要flywary迁移的schema，大小写敏感，默认为连接默认的schema.
flyway.sql-migration-prefix迁移文件的前缀，默认为V.
flyway.sql-migration-separator迁移脚本的文件名分隔符，默认__
flyway.sql-migration-suffix迁移脚本的后缀，默认为.sql
flyway.tableflyway使用的元数据表名，默认为schema_version
flyway.target迁移时使用的目标版本，默认为latest version
flyway.url迁移时使用的JDBC URL，如果没有指定的话，将使用配置的主数据源
flyway.user迁移数据库的用户名
flyway.validate-on-migrate迁移时是否校验，默认为true
```


[原文链接](https://blog.csdn.net/qq_41378597/article/details/124134146)
