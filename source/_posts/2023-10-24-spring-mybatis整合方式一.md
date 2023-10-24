---
title:  "spring-mybatis整合方式一"
subtitle: "It's always a bit messy"
author: "suyi"
date:   2023-10-24 12:12:12
---

# 一、mybatis整合方式一

### 1.封装一个user实体类

~~~JAVA
package com.sec.pojo;

public class User {
    private int id;
    private String name;
    private String pwd;

    public User() {
    }

    public User(int id, String name, String pwd) {
        this.id = id;
        this.name = name;
        this.pwd = pwd;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPwd() {
        return pwd;
    }

    public void setPwd(String pwd) {
        this.pwd = pwd;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", pwd='" + pwd + '\'' +
                '}';
    }
}

~~~

### 2.导入pom.xml的依赖以及maven的资源顾虑问题的build

~~~xml
<dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.8.13</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.2</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.16.10</version>
        </dependency>
    </dependencies>
    <build>
        <resources>
            <resource>
                <directory>src/main/resources/</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
~~~

**注意事项**：版本要对应，否则无法会报错！

​

### 3.编写一个接口UserMapper

~~~java
package com.sec.mapper;

import com.sec.pojo.User;

import java.util.List;

public interface UserMapper {
    public List<User> select();
}

~~~

### 4.完成UserMapper.xml的配置

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.sec.mapper.UserMapper">
    <select id="select" resultType="user" >
        select * from user
    </select>

</mapper>

~~~

### 5.完成mybatis-config.xml的配置

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        <package name="com.sec.pojo"/>
    </typeAliases>
    <!--<settings>
        <setting name="" value=""/>
    </settings>-->
    <!--<environments default="development">
        <environment id="development">
            <transactionManager type="JDBC" />
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/hjcdb" />
                <property name="username" value="root" />
                <property name="password" value="123456" />
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="com/com.sec/mapper/UserMapper.xml" />
    </mappers>-->

</configuration>

~~~



### 6.完成Spring核心配置文件

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:context="http://www.springframework.org/schema/context" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx" xmlns:p="http://www.springframework.org/schema/p" xmlns:util="http://www.springframework.org/schema/util" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
       xmlns:cache="http://www.springframework.org/schema/cache"
       xsi:schemaLocation="
  http://www.springframework.org/schema/context
  http://www.springframework.org/schema/context/spring-context.xsd
  http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans.xsd
  http://www.springframework.org/schema/tx
  http://www.springframework.org/schema/tx/spring-tx.xsd
  http://www.springframework.org/schema/jdbc
  http://www.springframework.org/schema/jdbc/spring-jdbc-3.1.xsd
  http://www.springframework.org/schema/cache
  http://www.springframework.org/schema/cache/spring-cache-3.1.xsd
  http://www.springframework.org/schema/aop
  http://www.springframework.org/schema/aop/spring-aop.xsd
  http://www.springframework.org/schema/util
  http://www.springframework.org/schema/util/spring-util.xsd">
    <!--DataSource -->
    <bean id="DataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/hjcdb" />
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
    </bean>
    <!--SqlSessionFactory-->
    <bean id="SqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="DataSource"/>
        <!--绑定mybatis配置文件-->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <property name="mapperLocations" value="classpath:com/sec/mapper/UserMapper.xml"/>
    </bean>
    <!--就是我们使用的SqlSession-->
    <bean id="SqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <!--只能使用构造器注入SqlSessionFactory 。 因为它没有set方法-->
        <constructor-arg index="0" ref="SqlSessionFactory"/>
    </bean>


</beans>
~~~

### 7.完成applicationContext.xml

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:context="http://www.springframework.org/schema/context" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx" xmlns:p="http://www.springframework.org/schema/p" xmlns:util="http://www.springframework.org/schema/util" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
       xmlns:cache="http://www.springframework.org/schema/cache"
       xsi:schemaLocation="
  http://www.springframework.org/schema/context
  http://www.springframework.org/schema/context/spring-context.xsd
  http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans.xsd
  http://www.springframework.org/schema/tx
  http://www.springframework.org/schema/tx/spring-tx.xsd
  http://www.springframework.org/schema/jdbc
  http://www.springframework.org/schema/jdbc/spring-jdbc-3.1.xsd
  http://www.springframework.org/schema/cache
  http://www.springframework.org/schema/cache/spring-cache-3.1.xsd
  http://www.springframework.org/schema/aop
  http://www.springframework.org/schema/aop/spring-aop.xsd
  http://www.springframework.org/schema/util
  http://www.springframework.org/schema/util/spring-util.xsd">

    <import resource="beans-dao.xml" />
    <bean id="userMapper" class="com.sec.mapper.UserMapperImpl">
        <property name="sqlSession" ref="SqlSession"/>
    </bean>
</beans>
~~~

### 8.编写接口UserMapper的实现类UserMapperImpl

~~~java
package com.sec.mapper;

import com.sec.pojo.User;
import org.mybatis.spring.SqlSessionTemplate;

import java.util.List;

public class UserMapperImpl implements UserMapper {

    private SqlSessionTemplate sqlSession;

    public void setSqlSession(SqlSessionTemplate sqlSession) {
        this.sqlSession = sqlSession;
    }

    @Override
    public List<User> select() {
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        return mapper.select();
    }
}

~~~

### 9.测试类中进行测试

~~~java
ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationContext.xml");
        UserMapper userMapper = (UserMapper) context.getBean("userMapper");
        List<User> select = userMapper.select();
        for (User user : select) {
            System.out.println(user);
        }
~~~



# 二、mybatis整合方式二

### 1.编写接口的实现类

- 在方式一的基础上可以再编写一个接口的实现类，要继承SqlSessionDaoSupport
- 使用getSqlSession();方法获取SqlSession
- 使用SqlSession获取接口
- 调用方法



~~~java
package com.sec.mapper;

import com.sec.pojo.User;
import org.apache.ibatis.session.SqlSession;
import org.mybatis.spring.support.SqlSessionDaoSupport;

import java.util.List;

public class UserMapperImpl2 extends SqlSessionDaoSupport implements UserMapper{
    @Override
    public List<User> select() {
        SqlSession session = getSqlSession();
        UserMapper mapper = session.getMapper(UserMapper.class);
        return mapper.select();
    }
}

~~~

### 2.在applicationContext.xml中注册UserMapperImpl2的bean

~~~xml
<bean id="userMapper2" class="com.sec.mapper.UserMapperImpl2">
        <property name="sqlSessionFactory" ref="SqlSessionFactory"/>
</bean>
~~~

**注意**：

- 在继承了 SqlSessionDaoSupport类后可以不用SqlSessionTemplate了


- 直接在applicationContext.xml文件中的userMapper2中注入SqlSessionFactory就可以了



### 3.测试

~~~java
ApplicationContext context = new ClassPathXmlApplicationContext("ApplicationContext.xml");
        UserMapper userMapper = (UserMapper) context.getBean("userMapper2");
        List<User> select = userMapper.select();
        for (User user : select) {
            System.out.println(user);
        }
~~~

