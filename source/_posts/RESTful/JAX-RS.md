---
title: JAX-RS
date: 2022-04-22 23:41
tags: 
  - RESTful
categories:
  - [RESTful]
---

# JAX-RS

**JAX-RS: Java API for RESTful Web Services**是一个Java编程语言的应用程序接口,支持按照 表象化状态转变 (REST)架构风格创建Web服务。JAX-RS使用了 ++Java SE 5++ 引入的Java 标注来简化Web服务客户端和服务端的开发和部署。


## 规范内容
JAX-RS提供了一些标注将一个资源类，一个POJOJava类，封装为Web资源。标注包括：

- @Path，标注资源类或方法的相对路径
- @GET，@PUT，@POST，@DELETE，标注方法是用的HTTP请求的类型
- @Produces，标注返回的MIME媒体类型
- @Consumes，标注可接受请求的MIME媒体类型
- @PathParam，@QueryParam，@HeaderParam，@CookieParam，@MatrixParam，@FormParam,分别标注方法的参数来自于HTTP请求的不同位置，例如@PathParam来自于URL的路径，@QueryParam来自于URL的查询参数，@HeaderParam来自于HTTP请求的头信息，@CookieParam来自于HTTP请求的Cookie。


## JAX-RS的实现
- Apache CXF，开源的Web服务框架。
- Jersey， 由Sun提供的JAX-RS的参考实现。
- RESTEasy，JBoss的实现。
- Restlet，由Jerome Louvel和Dave Pawson开发，是最早的REST框架，先于JAX-RS出现。
- Apache Wink，一个Apache软件基金会孵化器中的项目，其服务模块实现JAX-RS规范


---
## 说明

Java EE 6 引入了对 JSR-311 的支持。JSR-311（JAX-RS：Java API for RESTful Web Services）旨在定义一个统一的规范，使得 Java 程序员可以使用一套固定的接口来开发 REST 应用，避免了依赖于第三方框架。同时，JAX-RS 使用 POJO 编程模型和基于标注的配置，并集成了 JAXB，从而可以有效缩短 REST 应用的开发周期。  

JAX-RS 定义的 API 位于 javax.ws.rs 包中。  

JAX-RS 的具体实现由第三方提供，例如 Sun 的参考实现 Jersey、Apache 的 CXF 以及 JBoss 的 RESTEasy。
