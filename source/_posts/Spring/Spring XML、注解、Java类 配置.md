---
title: Spring XML、注解、Java类 配置
date: 2018-06-04 23:41
tags: 
  - Spring
categories:
  - [Spring]
---

配置Spring一般包括3种方式
- 基于XML的配置文件
- 基于注解的配置
- 基于Java的配置


## 基于XML的配置

这个很常见，比如：
```
<?xml version = "1.0" encoding = "UTF-8"?>

<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id = "helloWorld" class = "com.tutorialspoint.HelloWorld">
      <property name = "message" value = "Hello World!"/>
   </bean>

</beans>
```

这种配置可以通过容器来加载，也可以通过FileSystemXmlApplicationContext、ClassPathXmlApplicationContext 等来加载，如：
```
AbstractApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        
context.getBean("helloWorld");
```
## 使用注解
配置了<context：annotation-config />开启注解
```
<?xml version = "1.0" encoding = "UTF-8"?>

<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xmlns:context = "http://www.springframework.org/schema/context"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
   http://www.springframework.org/schema/context
   http://www.springframework.org/schema/context/spring-context-3.0.xsd">

   <context:annotation-config/>
   <!-- bean definitions go here -->

</beans>
```

注解 | 说明
---|---
@Required | 用于bean属性的setter方法
@Autowired | 可以应用于bean属性setter方法、非setter方法、构造函数和属性
@Qualifier | @Qualifier注解和@Autowired一起可以通过指定连接哪个bean来消除混淆。
JSR-250 Annotations | Spring支持基于JSR-250的注解，包括@Resource、@PostConstruct和@PreDestroy注解。




## 基于java的方式

### @Configuration & @Bean

使用@Configuration注解一个类表明该类可以被Spring IoC容器用作bean定义的来源。该@Bean注解告诉Spring与@Bean注释的方法将返回应注册为Spring应用程序上下文的bean的对象。最简单的@Configuration类如下：
```
package com.tutorialspoint;
import org.springframework.context.annotation.*;

@Configuration
public class HelloWorldConfig {
   @Bean 
   public HelloWorld helloWorld(){
      return new HelloWorld();
   }
}
```
以上代码将等同于以下XML配置 -
```
<beans>
   <bean id = "helloWorld" class = "com.tutorialspoint.HelloWorld" />
</beans>
```

这里，方法名用@Bean作为bean ID进行注释，并创建并返回实际的bean。你的配置类可以拥有多个@Bean的声明。一旦你的配置类被定义，你可以使用AnnotationConfigApplicationContext加载并提供给Spring容器，如下所示：
```
public static void main(String[] args) {
   ApplicationContext ctx = new AnnotationConfigApplicationContext(HelloWorldConfig.class);
   
   HelloWorld helloWorld = ctx.getBean(HelloWorld.class);
   helloWorld.setMessage("Hello World!");
   helloWorld.getMessage();
}
```
可以按如下方式加载各种配置类：
```
public static void main(String[] args) {
   AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();

   ctx.register(AppConfig.class, OtherConfig.class);
   ctx.register(AdditionalConfig.class);
   ctx.refresh();

   MyService myService = ctx.getBean(MyService.class);
   myService.doStuff();
}
```

### @Import
@Import注解允许从另一个配置类加载@Bean定义  
如配置类A
```
@Configuration
public class ConfigA {
   @Bean
   public A a() {
      return new A(); 
   }
}
```
可以在另一个Bean声明中导入上面的Bean声明，如下所示
```
@Configuration
@Import(ConfigA.class)
public class ConfigB {
   @Bean
   public B a() {
      return new A(); 
   }
}
```
现在，不需要在实例化上下文时指定ConfigA.class和ConfigB.class，只需要提供ConfigB，如下所示
```
public static void main(String[] args) {
   ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);
   
   // now both beans A and B will be available...
   A a = ctx.getBean(A.class);
   B b = ctx.getBean(B.class);
}

```

### 生命周期回调
@Bean注解支持指定任意初始化和销毁回调方法，就像Spring XML的bean元素的init-method和destroy-method属性一样
```
public class Foo {
   public void init() {
      // initialization logic
   }
   public void cleanup() {
      // destruction logic
   }
}
@Configuration
public class AppConfig {
   @Bean(initMethod = "init", destroyMethod = "cleanup" )
   public Foo foo() {
      return new Foo();
   }
}
```

### 指定Bean范围
默认范围是单例，但您可以使用@Scope注释覆盖它，如下所示
```
@Configuration
public class AppConfig {
   @Bean
   @Scope("prototype")
   public Foo foo() {
      return new Foo();
   }
}
```
