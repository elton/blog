---
layout: post
title: 'SPRING-MVC访问静态文件'
description: 'spring mvc 访问静态资源，如css，js'
date: 2013-04-10
comments: true
categories:
- Java
tags:
- spring

---

{:toc}

# 问题
如何你的DispatcherServlet拦截 *.do这样的URL，就不存在访问不到静态资源的问题。如果你的DispatcherServlet拦截“/”，拦截了所有的请求，同时对*.js,*.jpg的访问也就被拦截了。
 
# 目的
可以正常访问静态文件，不要找不到静态文件报404。
 
# 解决方案
激活Tomcat的defaultServlet来处理静态文件
Xml代码  

```
<servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.jpg</url-pattern>
</servlet-mapping>
<servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.png</url-pattern>
</servlet-mapping>
<servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.gif</url-pattern>
</servlet-mapping>
<servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.js</url-pattern>
</servlet-mapping>
<servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.css</url-pattern>
</servlet-mapping>
```

要配置多个，每种文件配置一个   
要写在**DispatcherServlet**的前面， 让defaultServlet先拦截，这个就不会进入Spring了，我想性能是最好的吧。

# 常用容器默认名字
* Tomcat, Jetty, JBoss, and GlassFish  默认 Servlet的名字 -- "default"
* Google App Engine 默认 Servlet的名字 -- "_ah_default"
* Resin 默认 Servlet的名字 -- "resin-file"
* WebLogic 默认 Servlet的名字  -- "FileServlet"
* WebSphere  默认 Servlet的名字 -- "SimpleFileServlet"

