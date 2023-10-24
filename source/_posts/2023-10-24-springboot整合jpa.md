---
title:  "springboot整合jpa"
subtitle: "It's always a bit messy"
author: "suyi"
date:   2023-10-24 12:12:12
---

# SpringBoot-data-JPA

## JpaRepository接口

 **==org.springframework.data.jpa.repository.JpaRepository;==**

1. 导入依赖

   ~~~xml
   <dependencies>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-data-jpa</artifactId>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-web</artifactId>
       </dependency>
       <dependency>
           <groupId>com.alibaba</groupId>
           <artifactId>druid-spring-boot-starter</artifactId>
           <version>1.1.9</version>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-devtools</artifactId>
           <scope>runtime</scope>
           <optional>true</optional>
       </dependency>
       <dependency>
           <groupId>mysql</groupId>
           <artifactId>mysql-connector-java</artifactId>
           <scope>runtime</scope>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-configuration-processor</artifactId>
           <optional>true</optional>
       </dependency>
       <dependency>
           <groupId>org.projectlombok</groupId>
           <artifactId>lombok</artifactId>
           <optional>true</optional>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-test</artifactId>
           <scope>test</scope>
       </dependency>
   </dependencies>
   ~~~

2. 配置application.yml

   ~~~yaml
   spring:
     datasource:
       type: com.alibaba.druid.pool.DruidDataSource
       username: root
       password: 123456
       url: jdbc:mysql://localhost:3306/hjcDB?serverTimezone=UTC&&characterEncoding=utf8&useUnicode=true
   
     jpa:
       show-sql: true
       database: mysql
       database-platform: mysql
       hibernate:
         ddl-auto: update
       properties:
         hibernate:
           dialect: org.hibernate.dialect.MySQL57Dialect
   ~~~

3. 写实体类

   ~~~java
   package com.sec.pojo;
   
   import lombok.AllArgsConstructor;
   import lombok.Data;
   import lombok.NoArgsConstructor;
   
   import javax.persistence.Entity;
   import javax.persistence.GeneratedValue;
   import javax.persistence.GenerationType;
   import javax.persistence.Id;
   
   @Entity(name = "userInfo")//表名
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   public class user {
       //主键
       @Id
       //自动增长
       @GeneratedValue(strategy = GenerationType.IDENTITY)
       private Integer id;
       private String name;
       private String pwd;
   
   }
   
   ~~~

4. 写接口，继承JpaRepository<user,Integer>

   ~~~java
   package com.sec.dao;
   
   import com.sec.pojo.user;
   import org.springframework.data.jpa.repository.JpaRepository;
   
   public interface userDao extends JpaRepository<user,Integer> {
       //传统样式查询参数（`？`）不再支持；使用JPA样式的序号参数（例如，`？1’）
       /*@Query("from user_info where name = ?")
       List<user> queryByNameUseHQL(String name);*/
   
       @Query(value = "select * from user_info where name=?",nativeQuery = true)
       List<user> queryByNameUseSQL(String name);
   
       @Query(value = "update user_info set name=? where id=?",nativeQuery = true)
       @Modifying//需要执行一个更新操作
       void updateUsersNameById(String name,Integer id);
   }
   
   ~~~

5. 测试

   ~~~java
   @Test
   void contextLoads() {
       /*//相当于insert语句
       userDao.save(new user(null,"zhangsan","123456"));
       userDao.save(new user(null,"lisi","123456"));
       userDao.save(new user(null,"wangwu","123456"));
       userDao.save(new user(null,"laksjkda","123456"));
   
       //相当于select语句
       List<user> all = userDao.findAll();
       for (user u: all
            ) {
           System.out.println(u);
       }*/
   
       
   
       //HQL   已不支持
       /*List<user> wangwu = userDao.queryByNameUseHQL("wangwu");
       for (user user : wangwu) {
           System.out.println(user);
       }*/
   
   	//自定义查询方法测试
       /*List<user> lisi = userDao.queryByNameUseSQL("lisi");
       for (user user : lisi) {
           System.out.println(user);
       }*/
       
       //根据id删除
       //userDao.deleteById(6);
       
       //修改
       Optional<user> userDaoById = userDao.findById(2);
           user user = userDaoById.get();
           user.setName("猪八戒");
           user.setPwd("zhubajie");
           userDao.save(user);
   }
   ~~~

6. 自定义方法测试    修改

   ~~~java
   @Test
   @Transactional //@Transactional与@Test 一起使用时 事务是自动回滚的。
   @Rollback(false) //取消自动回滚
   public void upd(){
       userDao.updateUsersNameById("haojiacheng",1);
   }
   ~~~

   

## Repository接口

1. 编写接口继承Repository

   ~~~java
   package com.sec.dao;
   
   import com.sec.pojo.user;
   import org.springframework.data.repository.Repository;
   
   import java.util.List;
   
   public interface userDaoByName extends Repository<user,Integer>  {
       /**
        * 根据名字查信息
        * @param name
        * @return
        */
       List<user> findByName(String name);
   
       /**
        * 根据名字和密码查信息
        * @param name
        * @param pwd
        * @return
        */
       List<user> findByNameAndPwd(String name,String pwd);
   
       /**
        * 根据id删除信息
        * @param id
        */
       void deleteById(Integer id);
   
       @Query(value = "update user_info set name=? where id=?",nativeQuery = true)
       @Modifying
       void updateUserName(String name,Integer id);
   }
   
   ~~~

2. 测试

   ~~~java
   @Test
   public void hjc(){
       //根据名字查询信息
       /*List<user> res = userDaoByName.findByName("zhangsan");
       for (user u:
               res) {
           System.out.println(u);
       }*/
   
       //根据名字和密码查询信息
       /*List<user> zhangsan = userDaoByName.findByNameAndPwd("adf", "123456");
       System.out.println(zhangsan);
       System.out.println(zhangsan==null);
       System.out.println(zhangsan.equals(""));
       for (user u :
               zhangsan) {
           System.out.println(u);
       }*/
       
       //根据id删除
       //userDaoByName.deleteById(7);
       
       //修改
       @Test
       @Transactional
       @Rollback(false) //取消自动回滚
       void test1(){
           userDaoByName.updateUserName("郝嘉诚",1);
       }
   }
   ~~~

   

## **CrudRepository** 接口

​	如果持久层接口较多，且每一个接口都需要声明相似的增删改查方法，直接继承 Repository 就显得有些啰嗦，这时可以继承 CrudRepository，它会自动为域对象创建增删改查方法，供业务层直接使用。开发者只是多写了 "Crud" 四个字母，即刻便为域对象提供了开箱即用的十个增删改查方法。

​	但是，使用 CrudRepository 也有副作用，它可能暴露了你不希望暴露给业务层的方法。比如某些接口你只希望提供增加的操作而不希望提供删除的方法。针对这种情况，开发者只能退回到 Repository 接口，然后到 CrudRepository 中把希望保留的方法声明复制到自定义的接口中即可。

​	分页查询和排序是持久层常用的功能，Spring Data 为此提供了 PagingAndSortingRepository 接口，它继承自 CrudRepository 接口，在 CrudRepository 基础上新增了两个与分页有关的方法。但是，我们很少会将自定义的持久层接口直接继承自 PagingAndSortingRepository，而是在继承 Repository 或 CrudRepository 的基础上，在自己声明的方法参数列表最后增加一个 Pageable 或 Sort 类型的参数，用于指定分页或排序信息即可，这比直接使用 PagingAndSortingRepository 提供了更大的灵活性。

​	JpaRepository 是继承自 PagingAndSortingRepository 的针对 JPA 技术提供的接口，它在父接口的基础上，提供了其他一些方法，比如 flush()，saveAndFlush()，deleteInBatch() 等。如果有这样的需求，则可以继承该接口。

Repository用于查询，CrudRepository用于增删改 

上代码：

1. 创建一个interface 接口 继承 CrudRepository

   ~~~java
   package com.sec.dao;
   
   import com.sec.pojo.user;
   import org.springframework.data.repository.CrudRepository;
   
   public interface userCrud extends CrudRepository<user,Integer> {
   
   }
   
   ~~~

2. 测试：

   ~~~java
   @Test
       void CrudRepositoryTest(){
           /*//增
           user user = new user();
           user.setName("苏弋");
           user.setPwd("452256");
           userCrud.save(user);*/
   
           //删
           //userCrud.deleteById(10);
   
           //改
   
   
           //查全部
           /*Iterable<user> all = userCrud.findAll();
           for (user user : all) {
               System.out.println(user);
           }*/
   
           //根据id查
           /*Optional<user> userCrudById = userCrud.findById(1);
           System.out.println(userCrudById.get());*/
   
           //查总记录数
           /*long count = userCrud.count();
           System.out.println("共："+count+"行");*/
   
           //根据id查询是否存在
           /*boolean flag = userCrud.existsById(1);
           if(flag=true){
               System.out.println("存在");
           }else{
               System.out.println("不存在");
           }*/
           
           //修改
           Optional<user> byId = userCrud.findById(2);
           user user = byId.get();
           user.setName("孙悟空");
           user.setPwd("sunwukong");
           userCrud.save(user);
           
       }
   ~~~

   