---
layout:     post
title:      框架
subtitle:   浅析SpringMVC的工作流程
date:       2021-6-3
author:     焦旭峰
header-img: img/the-first.png
catalog:   true
tags:
    - SpringMVC
---
> ## 浅析SpringMVC的工作流程

### 前言

​		在面试中，经常会遇到一个很经典的问题，SpringMVC的工作流程是什么？初次遇到这个问题，可能显得非常棘手。我相信大家结合SpringMVC的流程图和实际代码去理解，一定能够很完美的回答这个问题。

### 1、SpringMVC的执行流程简介

![img](https://img-blog.csdnimg.cn/20200208211439106.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoaW5rV29u,size_16,color_FFFFFF,t_70)

（1）浏览器提交请求到**DispatcherServlet**中央调度器。

（2）中央调度器直接将请求转给HandlerMapping处理器映射器。

（3）处理器映射器会根据请求，找到处理该请求的处理器，并将其封装为处理器执行链后

返回给中央调度器。

（4）中央调度器根据处理器执行链中的处理器，找到能够执行该处理器的处理器适配器。

（5）处理器适配器调用执行Handler处理器。

（6）处理器将处理结果及要跳转的视图封装到一个对象 ModelAndView 中，并将其返回给

处理器适配器。

（7）处理器适配器直接将结果返回给中央调度器。

（8）中央调度器调用ViewResolver视图解析器，将 ModelAndView 中的视图名称封装为视图对象。

（9）视图解析器将封装了的视图对象View返回给中央调度器。

（10）中央调度器调用视图对象，让其自己进行渲染，即进行数据填充，形成响应对象。

（11）中央调度器响应浏览器。

### 2、从代码的角度分析SpringMVC的工作流程

#### 2.1、搭建一个简单的SpringMVC项目

首先新建一个maven web项目

##### 2.1.1、pom.xml

创建好项目后，在pom.xml文件中加入Servlet和SpringMVC依赖

```xml
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    <!--servlet依赖-->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>
    <!--springmvc依赖-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.2.5.RELEASE</version>
    </dependency>
```

##### 2..1.2、注册中央调度器

在web.xml文件中注册中央调度器：

```xml
    <servlet>
        <servlet-name>myweb</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>‘
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>myweb</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
```

##### 2.1.3、创建处理器

```java
package com.bjpowernode.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import javax.xml.ws.RequestWrapper;


@Controller
@RequestMapping("/user")
public class MyController {
    @RequestMapping(value = "/other.do",method = RequestMethod.POST)
    public ModelAndView doOther(){
        ModelAndView mv  = new ModelAndView();
        mv.addObject("msg","====使用springmvc做web开发====");
        mv.addObject("fun","执行的是doOther方法");
        mv.setViewName("other");
        return mv;
    }
}

```

##### 2.1.4、创建SpringMVC配置文件

在resources目录下，创建springmvc.xml文件，在该文件中声明组件扫描器和视图解析器。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--声明组件扫描器-->
    <context:component-scan base-package="com.bjpowernode.controller" />

    <!--声明 springmvc框架中的视图解析器， 帮助开发人员设置视图文件的路径-->
    <bean  class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--前缀：视图文件的路径-->
        <property name="prefix" value="/WEB-INF/view/" />
        <!--后缀：视图文件的扩展名-->
        <property name="suffix" value=".jsp" />
    </bean>
</beans>
```

##### 2.1.5、定义JSP页面

index.jsp如下：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
     <p>第一个springmvc项目</p>
     <br/>
     <form action="user/other.do" method="post">
          <input type="submit" value="post请求other.do">
     </form>

</body>
</html>

```

other.jsp如下：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h3>/WEB-INF/view/other.jsp从request作用域获取数据</h3><br/>
    <h3>msg数据：${msg}</h3><br/>
    <h3>fun数据：${fun}</h3>
</body>
</html>

```

此时，module的结构如下图所示：

![image-20210606221623843](C:\Users\29348\Desktop\个人博客\博文\2021-06-03-浅析SpringMVC的工作流程.assets\image-20210606221623843.png)

#### 2.2、代码层面下的SpringMVC工作流程

启动tomcat服务器，浏览器弹出index.jsp页面，在该页面上提交表单，最终浏览器返回的结果是other.jsp页面。这个过程中发生了。

（1）tomcat服务器启动之后，会创建DispatcherServlet对象，当用户通过浏览器在**index.jsp**页面提交请求，这个http请求会发送到中央调度器。

（2）中央调度器直接将请求转给HandlerMapping处理器映射器。

（3）处理器映射器会根据请求，去**MyController**类中找到处理该请求的处理器（**doOther方法**），并将其封装为处理器执行链后返回给中央调度器。

（4）中央调度器根据处理器执行链中的处理器，找到能够执行该处理器的处理器适配器。

（5）处理器适配器调用执行Handler处理器（**MyController类的doOther方法**）。

（6）该**doOther方法**将处理结果及要跳转的视图封装到一个ModelAndView 对象中，并将其返回给处理器适配器。

（7）处理器适配器直接将结果返回给中央调度器。

（8）中央调度器调用**Springmvc.xml**中声明的视图解析器，将 ModelAndView 中的视图名称封装为视图对象(**other.jsp**)。

（9）视图解析器将封装了的视图对象返回给中央调度器。

（10）中央调度器调用视图对象，让其自己进行渲染，即进行数据填充，形成响应对象。

（11）中央调度器响应浏览器。
