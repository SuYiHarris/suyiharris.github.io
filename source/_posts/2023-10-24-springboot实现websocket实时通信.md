---
title:  "springboot实现websocket实时通信"
subtitle: "It's always a bit messy"
author: "suyi"
date:   2023-10-24 12:12:12
---

# springboot实现websocket实时通信

直接上代码

### 1.创建springboot项目

![1691150993767](../assets/1691150993767.png)

### 2.导入pom依赖

~~~xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-websocket</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.76</version>
    </dependency>
</dependencies>
~~~

### 3.创建pojo类

#### Message.java

~~~java
package com.hjc.mychat.pojo;

public class Message {
    public String toName;
    public String message;

    @Override
    public String toString() {
        return "Message{" +
                "toName='" + toName + '\'' +
                ", message='" + message + '\'' +
                '}';
    }

    public String getToName() {
        return toName;
    }

    public void setToName(String toName) {
        this.toName = toName;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public Message(String toName, String message) {
        this.toName = toName;
        this.message = message;
    }

    public Message() {
    }
}

~~~

#### Result.java

~~~java
package com.hjc.mychat.pojo;

public class Result {
    private boolean flag;
    private String message;

    public Result() {
    }

    public Result(boolean flag, String message) {
        this.flag = flag;
        this.message = message;
    }

    @Override
    public String toString() {
        return "Result{" +
                "flag=" + flag +
                ", message='" + message + '\'' +
                '}';
    }

    public boolean isFlag() {
        return flag;
    }

    public void setFlag(boolean flag) {
        this.flag = flag;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}

~~~

#### ResultMessage.java

~~~java
package com.hjc.mychat.pojo;

public class ResultMessage {
    private boolean isSystemMessage;
    private String fromName;
    private Object message;

    @Override
    public String toString() {
        return "ResultMessage{" +
                "isSystemMessage=" + isSystemMessage +
                ", fromName='" + fromName + '\'' +
                ", message=" + message +
                '}';
    }

    public boolean isSystemMessage() {
        return isSystemMessage;
    }

    public void setSystemMessage(boolean systemMessage) {
        isSystemMessage = systemMessage;
    }

    public String getFromName() {
        return fromName;
    }

    public void setFromName(String fromName) {
        this.fromName = fromName;
    }

    public Object getMessage() {
        return message;
    }

    public void setMessage(Object message) {
        this.message = message;
    }

    public ResultMessage(boolean isSystemMessage, String fromName, Object message) {
        this.isSystemMessage = isSystemMessage;
        this.fromName = fromName;
        this.message = message;
    }

    public ResultMessage() {
    }
}

~~~

### 4.utils工具类层

MessageUtils.java

~~~java
package com.hjc.mychat.utils;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.hjc.mychat.pojo.ResultMessage;

public class MessageUtils {

    public static String getMessage(boolean isSystemMessage,String fromName,Object message){
        try {
            ResultMessage result = new ResultMessage();
            result.setSystemMessage(isSystemMessage);
            result.setMessage(message);
            if (fromName!=null){
                result.setFromName(fromName);
            }
            //把字符串转成json格式的字符串
            ObjectMapper mapper = new ObjectMapper();
            return mapper.writeValueAsString(result);
        }catch (JsonProcessingException e){
            e.printStackTrace();
        }
        return null;
    }
}

~~~

### 5.controller层

#### pageController.java

~~~java
package com.hjc.mychat.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class pageController {
    @RequestMapping("/login")
    public String login(){
        return "login";
    }
    @RequestMapping("/main")
    public String main(){
        return "main";
    }
}

~~~

userController.java

~~~java
package com.hjc.mychat.controller;

import com.hjc.mychat.pojo.Result;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpSession;

@RestController
public class userController {
    @RequestMapping("/toLogin")
    public Result tologin(@RequestParam("user") String user, @RequestParam("pwd") String pwd, HttpSession session){
        Result result = new Result();
        if (user.equals("张三")&&pwd.equals("123")){
            result.setFlag(true);
            session.setAttribute("user",user);
        }else if (user.equals("李四")&&pwd.equals("123")){
            result.setFlag(true);
            session.setAttribute("user",user);
        }else if (user.equals("123")&&pwd.equals("123")){
            result.setFlag(true);
            session.setAttribute("user",user);
        }
        else if (user.equals("王五")&&pwd.equals("123")){
            result.setFlag(true);
            session.setAttribute("user",user);
        }else {
            result.setFlag(false);
            result.setMessage("登录失败");
        }
        return result;
    }

    @RequestMapping("/getUsername")
    public String getUsername(HttpSession session){
        String username = (String) session.getAttribute("user");
        return username;
    }

}


~~~

### 6.config层

#### WebsocketConfig.java

```java
package com.hjc.mychat.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.server.standard.ServerEndpointExporter;

@Configuration
public class WebsocketConfig {

    @Bean
    public ServerEndpointExporter serverEndpointExporter(){
        return new ServerEndpointExporter();
    }
}
```

### 7.websocket层

#### ChatEndpoint.java

~~~java
package com.hjc.mychat.ws;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.hjc.mychat.pojo.Message;
import com.hjc.mychat.utils.MessageUtils;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpSession;
import javax.websocket.*;
import javax.websocket.server.ServerEndpoint;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

@ServerEndpoint( value = "/hjcchat",configurator = GetHttpSessionConfigurator.class)
@Component
public class ChatEndpoint {


    //用来存储每个用户客户端对象的ChatEndpoint对象
    private static Map<String,ChatEndpoint> onlineUsers = new ConcurrentHashMap<>();

    //声明session对象，通过对象可以发送消息给指定的用户
    private Session session;

    //声明HttpSession对象，我们之前在HttpSession对象中存储了用户名
    private HttpSession httpSession;

    //连接建立
    @OnOpen
    public void onOpen(Session session, EndpointConfig config){
        this.session = session;
        HttpSession httpSession = (HttpSession) config.getUserProperties().get(HttpSession.class.getName());
        this.httpSession = httpSession;
        //存储登陆的对象
        String username = (String)httpSession.getAttribute("user");
        onlineUsers.put(username,this);

        //将当前在线用户的用户名推送给所有的客户端
        //1 获取消息
        String message = MessageUtils.getMessage(true, null, getNames());
        //2 调用方法进行系统消息的推送
        broadcastAllUsers(message);
    }

    private void broadcastAllUsers(String message){
        try {
            //将消息推送给所有的客户端
            Set<String> names = onlineUsers.keySet();
            for (String name : names) {
                ChatEndpoint chatEndpoint = onlineUsers.get(name);
                chatEndpoint.session.getBasicRemote().sendText(message);
            }
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    //返回在线用户名
    private Set<String> getNames(){
        return onlineUsers.keySet();
    }

    //收到消息
    @OnMessage
    public void onMessage(String message,Session session){
        //将数据转换成对象
        try {
            ObjectMapper mapper =new ObjectMapper();
            Message mess = mapper.readValue(message, Message.class);
            String toName = mess.getToName();
            String data = mess.getMessage();
            String username = (String) httpSession.getAttribute("user");
            String resultMessage = MessageUtils.getMessage(false, username, data);
            //发送数据
            onlineUsers.get(toName).session.getBasicRemote().sendText(resultMessage);
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
    //关闭
    @OnClose
    public void onClose(Session session) {
        String username = (String) httpSession.getAttribute("user");
        //从容器中删除指定的用户
        onlineUsers.remove(username);
        String message = MessageUtils.getMessage(true, null, getNames());
        broadcastAllUsers(message);
    }
}

~~~

#### GetHttpSessionConfigurator.java

~~~java
package com.hjc.mychat.ws;

import javax.servlet.http.HttpSession;
import javax.websocket.HandshakeResponse;
import javax.websocket.server.HandshakeRequest;
import javax.websocket.server.ServerEndpointConfig;


public class GetHttpSessionConfigurator extends ServerEndpointConfig.Configurator {
    @Override
    public void modifyHandshake(ServerEndpointConfig sec, HandshakeRequest request, HandshakeResponse response) {
        //获取HttpSession对象
        HttpSession httpSession = (HttpSession) request.getHttpSession();
        sec.getUserProperties().put(HttpSession.class.getName(),httpSession);
    }
}


~~~



### 8.前端页面

#### login.html

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>网络聊天室-登录</title>
  <!--核心样式-->
  <link rel="stylesheet" href="css/style.css">
</head>
<body>

<div class="container">
  <div class="card"></div>
  <div class="card">
    <h1 class="title">用户登录</h1>
    <form id="loginForm" method="post">
      <div class="input-container">
        <input type="text" id="user" name="user" required="required" />
        <label for="user">用户名</label>
        <div class="bar"></div>
      </div>
      <div class="input-container">
        <input type="password" id="pwd" name="pwd" required="required" />
        <label for="pwd">密码</label>
        <div class="bar"></div>
      </div>
      <div class="button-container">
        <button type="button" id="btn"><span>登录</span></button>
      </div>
      <div><p style="color: red;text-align: center" id="err_msg"></p></div>
    </form>
  </div>
</div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
<script>
  $("#btn").click(function () {
    $.get("toLogin?",$("#loginForm").serialize(),function(res){
      if (res.flag){
        console.log(res);
        location.href = "main";
      } else {
        console.log(res);
        $("#err_msg").html(res.message);
      }
    },"json");
  })
  $("#pwd").keypress(function (e) {
    if(e.keyCode==13){
      $.get("toLogin?",$("#loginForm").serialize(),function(res){
        if (res.flag){
          console.log(res);
          location.href = "main";
        } else {
          console.log(res);
          $("#err_msg").html(res.message);
        }
      },"json");
    }

  })
</script>
</body>
</html>

~~~



#### main.html

~~~html
<!DOCTYPE html>
<html lang="en" xmlns="http://www.w3.org/1999/html">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>网络聊天室</title>
    <link rel="stylesheet" href="css/mycss.css">
</head>

<body>
<div id = "contains">
    <div id="username"><h3 style="text-align: center;">用户：张三<span>-在线</span></h3></div>
    <div id="Inchat"></div>
    <div id="left">
        <div id="content">

        </div>
        <div id="input">
            <textarea type="text" id="input_text" style="width: 695px;height: 200px;"></textarea>
            <button id="submit" style="float: right;">发送</button>
        </div>
    </div>
    <div id="right">
        <p id="hy" style="text-align: center;">好友列表</p>
        <div id="hyList">

        </div>
        <p id="xt" style="text-align: center">系统消息</p>
        <div id="xtList">

        </div>
    </div>
</div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
<script>
    var toName;
    var username;
    //点击好友名称展示相关消息
    function showChat(name){
        toName = name;
        //现在聊天框
        $("#content").html("");
        $("#content").css("visibility","visible");
        $("#Inchat").html("当前正与"+toName+"聊天");
        //从sessionStorage中获取历史信息
        var chatData = sessionStorage.getItem(toName);
        if (chatData != null){
            $("#content").html(chatData);
        }
    }
    $(function () {
        $.ajax({
            url:"getUsername",
            success:function (res) {
                username = res;
            },
            async:false //同步请求，只有上面好了才会接着下面
        });
        //建立websocket连接
        //获取host解决后端获取httpsession的空指针异常
        var host = window.location.host;
        var ws = new WebSocket("ws://"+host+"/hjcchat");
        ws.onopen = function (evt) {
            $("#username").html("<h3 style=\"text-align: center;\">用户："+ username +"<span>-在线</span></h3>");
        }
        //接受消息
        ws.onmessage = function (evt) {
            //获取服务端推送的消息
            var dataStr = evt.data;
            //将dataStr转换为json对象
            var res = JSON.parse(dataStr);

            console.log(res)
            //判断是否是系统消息
            if(res.systemMessage){
                //系统消息
                //1.好友列表展示
                //2.系统广播的展示
                //此处声明的变量是调试时命名的，可以直接合并
                var names = res.message;
                var userlistStr = "";
                var broadcastListStr = "";
                var temp01 = "<a style=\"text-align: center; display: block;\" onclick='showChat(\"";
                var temp03 = "\")'>";
                var temp04 = "</a></br>";
                var temp = "";
                for (var name of names){
                    if (name != username){
                        temp = temp01 + name + temp03 + name + temp04;
                        userlistStr = userlistStr + temp;
                        broadcastListStr += "<p style='text-align: center'>"+ name +"上线了</p>";
                    }
                }
                //渲染好友列表和系统广播
                $("#hyList").html(userlistStr);
                $("#xtList").html(broadcastListStr);

            }else {
                //不是系统消息
                var str = "<span id='mes_left'>"+ res.message +"</span></br>";
                if (toName === res.fromName) {
                    $("#content").append(str);
                }
                var chatData = sessionStorage.getItem(res.fromName);
                if (chatData != null){
                    str = chatData + str;
                }
                //保存聊天消息
                sessionStorage.setItem(res.fromName,str);
            };
        }
        ws.onclose = function () {
            $("#username").html("<h3 style=\"text-align: center;\">用户："+ username +"<span>-离线</span></h3>");
        }

        //发送消息
        $("#submit").click(function () {
            sendmessage();
        });

        //监听回车发送
        $("#input_text").keypress(function (e) {
            if(e.keyCode==13){
                sendmessage();
            }
        });

        function sendmessage() {
            //1.获取输入的内容
            var data = $("#input_text").val();
            //2.清空发送框
            $("#input_text").val("");
            var json = {"toName": toName ,"message": data};
            //将数据展示在聊天区
            var str = "<span id='mes_right'>"+ data +"</span></br>";
            $("#content").append(str);

            var chatData = sessionStorage.getItem(toName);
            if (chatData != null){
                str = chatData + str;
            }
            sessionStorage.setItem(toName,str);
            //3.发送数据
            ws.send(JSON.stringify(json));
        }
    })
</script>
</body>

</html>
~~~



### 9.css样式

#### mycss.css

~~~css
#contains{
    background-color: pink;
    width: 1000px;
    height: 700px;
    margin: auto;
}
#username{
    background-color: powderblue;
    width: 1000px;
    height: 30px;
}
#Inchat{
    background-color: rgb(5, 130, 146);
    width: 1000px;
    height: 30px;
}
#left{
    background-color: salmon;
    width: 700px;
    height: 640px;
    float: left;
    position: relative;
}
#content{
    background-color: silver;
    width: 700px;
    height: 400px;
    /*display: none;*/
    visibility: hidden;
}
#right{
    background-color: rgb(107, 3, 3);
    width: 300px;
    height: 640px;
    float: right;
}
#hyList{
    height: 270px;
    overflow-y: scroll;
    background: antiquewhite;
}
#xtList{
    height: 270px;
    overflow-y: scroll;
    background: antiquewhite;
}
#input{
    margin-bottom: 0px;
    position: absolute;
    bottom: 0px;
}
#mes_left{
    float: left;
    border: 2px aqua;
    max-width: 490px;
}
#mes_right{
    float: right;
    border: 2px aqua;
    max-width: 490px;
}
#hy{
    display: block;
    width: 300px;
}
textarea {
    resize: none;
    border: none;
    outline: none
}

~~~

#### style.css

~~~css
body {
  background: #e9e9e9;
  color: deeppink;
  font-family: "RobotoDraft", "Roboto", sans-serif;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 14px;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* Pen Title */
.pen-title {
  padding: 50px 0;
  text-align: center;
  letter-spacing: 2px;
}
.pen-title h1 {
  margin: 0 0 20px;
  font-size: 48px;
  font-weight: 300;
}
.pen-title span {
  font-size: 12px;
}
.pen-title span .fa {
  color: #006a71;
}
.pen-title span a {
  color: #006a71;
  font-weight: 600;
  text-decoration: none;
}

/* Container */
.container {
  position: relative;
  max-width: 460px;
  width: 100%;
  margin: 0 auto 100px;
}
.container.active .card:first-child {
  background: #f2f2f2;
  margin: 0 15px;
}
.container.active .card:nth-child(2) {
  background: #fafafa;
  margin: 0 10px;
}
.container.active .card.alt {
  top: 20px;
  right: 0;
  width: 100%;
  min-width: 100%;
  height: auto;
  border-radius: 5px;
  padding: 60px 0 40px;
  overflow: hidden;
}
.container.active .card.alt .toggle {
  position: absolute;
  top: 40px;
  right: -70px;
  box-shadow: none;
  -webkit-transform: scale(10);
  -moz-transform: scale(10);
  -ms-transform: scale(10);
  -o-transform: scale(10);
  transform: scale(10);
  transition: transform 0.3s ease;
}
.container.active .card.alt .toggle:before {
  content: "";
}
.container.active .card.alt .title,
.container.active .card.alt .input-container,
.container.active .card.alt .button-container {
  left: 0;
  opacity: 1;
  visibility: visible;
  transition: 0.3s ease;
}
.container.active .card.alt .title {
  transition-delay: 0.3s;
}
.container.active .card.alt .input-container {
  transition-delay: 0.4s;
}
.container.active .card.alt .input-container:nth-child(2) {
  transition-delay: 0.5s;
}
.container.active .card.alt .input-container:nth-child(3) {
  transition-delay: 0.6s;
}
.container.active .card.alt .button-container {
  transition-delay: 0.7s;
}

/* Card */
.card {
  position: relative;
  background: #ffffff;
  border-radius: 5px;
  padding: 60px 0 40px 0;
  box-sizing: border-box;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.12), 0 1px 2px rgba(0, 0, 0, 0.24);
  transition: 0.3s ease;
  /* Title */
  /* Inputs */
  /* Button */
  /* Alt Card */
}
.card:first-child {
  background: #fafafa;
  height: 10px;
  border-radius: 5px 5px 0 0;
  margin: 0 10px;
  padding: 0;
}
.card .title {
  position: relative;
  z-index: 1;
  border-left: 5px solid #006a71;
  margin: 0 0 35px;
  padding: 10px 0 10px 50px;
  color: #006a71;
  font-size: 32px;
  font-weight: 600;
  text-transform: uppercase;
}
.card .input-container {
  position: relative;
  margin: 0 60px 50px;
}
.card .input-container input {
  outline: none;
  z-index: 1;
  position: relative;
  background: none;
  width: 100%;
  height: 60px;
  border: 0;
  color: #212121;
  font-size: 24px;
  font-weight: 400;
}
.card .input-container input:focus ~ label {
  color: #9d9d9d;
  transform: translate(-12%, -50%) scale(0.75);
}
.card .input-container input:focus ~ .bar:before, .card .input-container input:focus ~ .bar:after {
  width: 50%;
}
.card .input-container input:valid ~ label {
  color: #9d9d9d;
  transform: translate(-12%, -50%) scale(0.75);
}
.card .input-container label {
  position: absolute;
  top: 0;
  left: 0;
  color: #757575;
  font-size: 24px;
  font-weight: 300;
  line-height: 60px;
  -webkit-transition: 0.2s ease;
  -moz-transition: 0.2s ease;
  transition: 0.2s ease;
}
.card .input-container .bar {
  position: absolute;
  left: 0;
  bottom: 0;
  background: #757575;
  width: 100%;
  height: 1px;
}
.card .input-container .bar:before, .card .input-container .bar:after {
  content: "";
  position: absolute;
  background: #ff7e67;
  width: 0;
  height: 2px;
  transition: 0.2s ease;
}
.card .input-container .bar:before {
  left: 50%;
}
.card .input-container .bar:after {
  right: 50%;
}
.card .button-container {
  margin: 0 60px;
  text-align: center;
}
.card .button-container button {
  outline: 0;
  cursor: pointer;
  position: relative;
  display: inline-block;
  background: 0;
  width: 240px;
  border: 2px solid #e3e3e3;
  padding: 20px 0;
  font-size: 24px;
  font-weight: 600;
  line-height: 1;
  text-transform: uppercase;
  overflow: hidden;
  transition: 0.3s ease;
}
.card .button-container button span {
  position: relative;
  z-index: 1;
  color: #ddd;
  transition: 0.3s ease;
}
.card .button-container button:before {
  content: "";
  position: absolute;
  top: 50%;
  left: 50%;
  display: block;
  background: #006a71;
  width: 30px;
  height: 30px;
  border-radius: 100%;
  margin: -15px 0 0 -15px;
  opacity: 0;
  transition: 0.3s ease;
}
.card .button-container button:hover, .card .button-container button:active, .card .button-container button:focus {
  border-color: #68b0ab;
}
.card .button-container button:hover span, .card .button-container button:active span, .card .button-container button:focus span {
  color: #68b0ab;
}
.card .button-container button:active span, .card .button-container button:focus span {
  color: #68b0ab;
}
.card .button-container button:active:before, .card .button-container button:focus:before {
  opacity: 1;
  -webkit-transform: scale(10);
  -moz-transform: scale(10);
  -ms-transform: scale(10);
  -o-transform: scale(10);
  transform: scale(10);
}
.card.alt {
  position: absolute;
  top: 40px;
  right: -70px;
  z-index: 10;
  width: 140px;
  height: 140px;
  background: none;
  border-radius: 100%;
  box-shadow: none;
  padding: 0;
  transition: 0.3s ease;
  /* Toggle */
  /* Title */
  /* Input */
  /* Button */
}
.card.alt .toggle {
  position: relative;
  background: #006a71;
  width: 140px;
  height: 140px;
  border-radius: 100%;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.12), 0 1px 2px rgba(0, 0, 0, 0.24);
  color: #ffffff;
  font-size: 58px;
  line-height: 140px;
  text-align: center;
  cursor: pointer;
}
.card.alt .toggle:before {
  content: "\f040";
  display: inline-block;
  font: normal normal normal 14px/1 FontAwesome;
  font-size: inherit;
  text-rendering: auto;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  transform: translate(0, 0);
}
.card.alt .title,
.card.alt .input-container,
.card.alt .button-container {
  left: 100px;
  opacity: 0;
  visibility: hidden;
}
.card.alt .title {
  position: relative;
  border-color: #ffffff;
  color: #ffffff;
}
.card.alt .title .close {
  cursor: pointer;
  position: absolute;
  top: 0;
  right: 60px;
  display: inline;
  color: #ffffff;
  font-size: 58px;
  font-weight: 400;
}
.card.alt .title .close:before {
  content: "\00d7";
}
.card.alt .input-container input {
  color: #ffffff;
}
.card.alt .input-container input:focus ~ label {
  color: #ffffff;
}
.card.alt .input-container input:focus ~ .bar:before, .card.alt .input-container input:focus ~ .bar:after {
  background: #ff7e67;
}
.card.alt .input-container input:valid ~ label {
  color: #ffffff;
}
.card.alt .input-container label {
  color: rgba(255, 255, 255, 0.8);
}
.card.alt .input-container .bar {
  background: rgba(255, 255, 255, 0.8);
}
.card.alt .button-container button {
  width: 100%;
  background: #ffffff;
  border-color: #ffffff;
}
.card.alt .button-container button span {
  color: #006a71;
}
.card.alt .button-container button:hover {
  background: rgba(255, 255, 255, 0.9);
}
.card.alt .button-container button:active:before, .card.alt .button-container button:focus:before {
  display: none;
}

/* Keyframes */
@-webkit-keyframes buttonFadeInUp {
  0% {
    bottom: 30px;
    opacity: 0;
  }
}
@-moz-keyframes buttonFadeInUp {
  0% {
    bottom: 30px;
    opacity: 0;
  }
}
@keyframes buttonFadeInUp {
  0% {
    bottom: 30px;
    opacity: 0;
  }
}
~~~







