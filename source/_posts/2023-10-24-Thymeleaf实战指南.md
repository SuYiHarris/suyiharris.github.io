---
title:  "Thymeleaf实战指南"
subtitle: "It's always a bit messy"
author: "suyi"
date:   2023-10-24 12:12:12
---

# Thymeleaf实战指南

Thymeleaf是一种用于在Web应用中渲染动态内容的模板引擎。它提供了丰富的语法和功能，使开发人员能够轻松地在HTML模板中插入动态数据和逻辑处理。我们实训中用到的一些常用语法和示例写法给大家总结下：

## Thymeleaf与Spring Boot集成步骤：

步骤1：在pom.xml中添加Thymeleaf和Spring Boot的依赖：

```xml
<dependencies>
    <!-- Spring Boot Starter Thymeleaf -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
</dependencies>
```

步骤2：配置Thymeleaf相关属性，如模板前缀、后缀等。在application.properties或application.yml中添加如下配置：

```properties
# Thymeleaf Configuration
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
spring.thymeleaf.mode=HTML
spring.thymeleaf.cache=false
```

步骤3：创建Thymeleaf模板文件。在`src/main/resources/templates/`目录下创建HTML模板文件。

步骤4：创建Controller类，处理请求并返回Thymeleaf模板。

```java
@Controller
public class MyController {

    @GetMapping("/")
    public String home(Model model) {
        model.addAttribute("message", "Hello Thymeleaf!");
        return "index"; // 返回对应的Thymeleaf模板文件名
    }
}
```

步骤5：在Thymeleaf模板中使用Thymeleaf的语法来渲染动态内容。

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Thymeleaf Example</title>
</head>
<body>
    <h1 th:text="${message}"></h1>
</body>
</html>
```

## Thymeleaf的常用语法

1. 输出变量值：

   ```html
   <p th:text="${variable}">Default value</p>
   ```

   这会将变量`${variable}`的值输出到`<p>`标签中。

2. 条件判断：

   ```html
   <p th:if="${condition}">Condition is true</p>
   <p th:unless="${condition}">Condition is false</p>
   ```

   当`${condition}`为真时，第一个`<p>`标签将显示；当`${condition}`为假时，第二个`<p>`标签将显示。

3. 循环迭代：

   ```html
   <ul>
     <li th:each="item : ${items}" th:text="${item}">Item</li>
   </ul>
   ```

   这会遍历`${items}`列表，并为每个元素创建一个`<li>`标签。

4. 表单处理：

   ```html
   <form th:action="@{/save}" method="post">
     <input type="text" name="name" />
     <button type="submit">Submit</button>
   </form>
   ```

   这是一个简单的表单，将数据提交到`/save`路径。服务端的代码：

后端代码，使用Spring MVC和Thymeleaf来处理表单提交：

```java
@Controller
public class MyController {

  @PostMapping("/save")
  public String saveForm(@RequestParam("name") String name) {
    // 处理表单提交的逻辑
    // 在这里可以对表单数据进行处理，如保存到数据库或进行其他操作
    System.out.println("Received name: " + name);
    
    // 返回重定向或渲染的视图
    return "redirect:/success";
  }

  @GetMapping("/success")
  public String showSuccessPage() {
    // 显示提交成功页面
    return "success";
  }

}
```

1. URL链接生成：

   ```html
   <a th:href="@{/users/{id}(id=${userId})}">User Details</a>
   ```

   这会生成一个链接，其中`${userId}`是动态的路径参数。

2. 模板片段（Fragment）：

   模板片段是一部分可以在多个页面中重用的HTML代码块。你可以将这些代码块提取出来，然后在不同的页面中引用它们。示例：

   ```html
   <!-- 定义模板片段 -->
   <div th:fragment="header">
     <h1>Welcome to My Website</h1>
   </div>
   
   <!-- 引用模板片段 -->
   <div th:replace="fragments/header :: header"></div>
   ```

   在以上示例中，`header`模板片段定义了一个标题，然后可以在其他页面中通过`th:replace`指令引用它。

3. 国际化支持（Internationalization）：

   Thymeleaf提供了简便的方式来处理国际化文本，以便在不同的语言环境下显示不同的内容。示例：

   ```html
   <!-- 使用国际化文本 -->
   <h2 th:text="#{welcome.message}">Welcome</h2>
   ```

   在以上示例中，`#{welcome.message}`表示从国际化资源文件中获取名为`welcome.message`的文本，并将其显示在页面上。

4. 片段缓存（Fragment Caching）：

   片段缓存允许你缓存特定的HTML代码块，以提高页面的渲染性能。示例：

   ```html
   <!-- 缓存模板片段 -->
   <div th:fragment="header" th:cacheable="true">
     <h1>Welcome to My Website</h1>
   </div>
   ```

   在以上示例中，通过将`th:cacheable`属性设置为`true`，可以将`header`模板片段进行缓存，以便在后续请求中直接使用缓存的结果，提高页面的响应速度。