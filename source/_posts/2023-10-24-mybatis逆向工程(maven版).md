---
title:  "mybatis逆向工程(maven版)"
subtitle: "It's always a bit messy"
author: "suyi"
date:   2023-10-24 12:12:12
---

### 通过 Maven 完成 Mybatis 逆向工程

#### 1. 新建一个 Maven Project 项目

新建一个 Maven 项目，然后新建文件夹 /mybatis-maven/src/main/resources，在文件夹下新建文件 generatorConfig.xml。 

![Maven 项目结构](https://pic.imgdb.cn/item/6537cc99c458853aef5f7964.jpg) 

#### 2. 配置 pom.xml 文件

配置 pom.xml 文件，在 pom.xml 文件的 project 标签里加入代码： 

~~~xml
<build>
	<plugins>
		<plugin>
			<groupId>org.mybatis.generator</groupId>
			<artifactId>mybatis-generator-maven-plugin</artifactId>
			<version>1.3.2</version>
			<dependencies>
				<dependency>
		            <groupId>mysql</groupId>
		            <artifactId>mysql-connector-java</artifactId>
		            <version>5.1.38</version>
		        </dependency>	
			</dependencies>
			<configuration>
				<overwrite>true</overwrite>
			</configuration>
		</plugin>
	</plugins>
</build>
~~~

配置插件 generator 版本是 1.3.2 并配置 Mysql 驱动是 5.1.38。 

#### 3. 配置文件 generatorConfig.xml

generatorConfig.xml 是在目录 src 下的 main 下的 resources 下。注意这里的`targetProject="./src"` 生成的文件也会在这个下面。 

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
	<context id="testTables" targetRuntime="MyBatis3">
		<commentGenerator>
			<!-- 是否去除自动生成的注释 true：是 ： false:否 -->
			<property name="suppressAllComments" value="false" />
		</commentGenerator>
		<!--数据库连接的信息：驱动类、连接地址、用户名、密码 -->
		<jdbcConnection driverClass="com.mysql.jdbc.Driver"
			connectionURL="jdbc:mysql://localhost:3306/mybatis" userId="root"
			password="123456">
		</jdbcConnection>
		<!-- <jdbcConnection driverClass="oracle.jdbc.OracleDriver"
			connectionURL="jdbc:oracle:thin:@localhost:1521:mybatis" 
			userId=""
			password="">
		</jdbcConnection> -->

		<!-- 默认false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer，为 true时把JDBC DECIMAL 和 
			NUMERIC 类型解析为java.math.BigDecimal -->
		<javaTypeResolver>
			<property name="forceBigDecimals" value="false" />
		</javaTypeResolver>

		<!-- targetProject:生成PO类的位置 -->
		<javaModelGenerator targetPackage="com.ssm.po"
			targetProject="./src">
			<!-- enableSubPackages:是否让schema作为包的后缀 -->
			<property name="enableSubPackages" value="false" />
			<!-- 从数据库返回的值被清理前后的空格 -->
			<property name="trimStrings" value="true" />
		</javaModelGenerator>
        <!-- targetProject:mapper映射文件生成的位置 -->
		<sqlMapGenerator targetPackage="com.ssm.mapper" 
			targetProject="./src">
			<!-- enableSubPackages:是否让schema作为包的后缀 -->
			<property name="enableSubPackages" value="false" />
		</sqlMapGenerator>
		<!-- targetPackage：mapper接口生成的位置 -->
		<javaClientGenerator type="XMLMAPPER"
			targetPackage="com.ssm.mapper" 
			targetProject="./src">
			<!-- enableSubPackages:是否让schema作为包的后缀 -->
			<property name="enableSubPackages" value="false" />
		</javaClientGenerator>
		<!-- 指定数据库表 -->
		<!-- 
			tableName:要生成的表名
       		domainObjectName:生成后的实例名
	        enableCountByExample:Count语句中加入where条件查询，默认true开启
	        enableUpdateByExample:Update语句中加入where条件查询，默认true开启
	        enableDeleteByExample:Delete语句中加入where条件查询，默认true开启
	        enableSelectByExample:Select多条语句中加入where条件查询，默认true开启
	        selectByExampleQueryId:Select单个对象语句中加入where条件查询，默认true开启
		 -->
		<table tableName="items">
			<!-- 
				常用：
				property:将所有字段逆向生成为类属性，默认全部
				ignoreColumn:生成时忽略列字段 
			 -->
		</table>
		<table tableName="orders"></table>
		<table tableName="orderdetail"></table>
		<table tableName="user"></table>

		
	</context>
</generatorConfiguration>
~~~

#### 4. 运行 Maven

运行命令`mybatis-generator:generate`。   操作步骤：选中项目右击 => Run As => Maven build… =>在 Goals 中输入`mybatis-generator:generate` => Run =>刷新工程。 

![生成的代码结构](https://pic.imgdb.cn/item/6537ccb5c458853aef600702.jpg) 

