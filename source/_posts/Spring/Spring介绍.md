---
title: Spring介绍
date: 2018-06-04 23:41
tags: 
  - Spring
categories:
  - [Spring]
---

*从这个地方随手翻译过来的*  
*https://www.tutorialspoint.com/spring/index.htm*  

## 概述
Spring是企业Java最流行的应用程序开发框架。全球数以百万计的开发人员使用Spring Framework创建高性能，易于测试和可重用的代码。

Spring框架是一个开源的Java平台。它最初由Rod Johnson编写，并于2003年6月首次在Apache 2.0许可下发布。

Spring框架的核心功能可以用于开发任何Java应用程序，但是在Java EE平台之上还可以扩展构建Web应用程序。Spring框架的目标是通过启用基于POJO的编程模型来使J2EE开发更易于使用，并促进良好的编程实践。

### 依赖注入（DI）
Spring最认常用技术是控制反转的依赖注入（DI）。该控制反转（IOC）是一个笼统的概念，它可以在许多不同的方式来表达。依赖注入仅仅是控制反转的一个具体例子。

### 面向切面编程（AOP）
Spring的关键组件之一是面向切面编程（AOP）框架。跨越应用程序多个点的功能称为横切关注点，这些横切关注点在概念上与应用程序的业务逻辑分离。有各种常见的很好的例子，包括日志记录，声明式事务，安全性，缓存等等。

----

## 框架
Spring框架提供了大约20个模块，可以根据应用需求使用。

### 核心容器
核心容器由Core，Beans，Context和Expression Language模块组成
- Core 提供了框架的基本部分，包括IOC和依赖注入特征。
- Beans 提供的BeanFactory，是工厂模式的一个复杂实现。
- Context 上下文模块构建在Core和Beans模块提供的基础之上，它是访问定义和配置的任何对象的媒介。ApplicationContext接口是上下文模块的焦点。
- Expression Language SpEL模块提供了一种强大的表达式语言，用于在运行时查询和操作对象图。
 

### 数据访问/集成
数据访问/集成层由JDBC，ORM，OXM，JMS和事务模块组成
- JDBC模块提供了JDBC抽象层，消除了对冗长的JDBC相关编码的需要。
- ORM模块为流行的对象-关系映射api提供集成层，包括JPA、JDO、Hibernate和iBatis。
- OXM模块提供了一个抽象层，支持JAXB、Castor、XMLBeans、JiBX和XStream的对象/XML映射实现。
- Java消息传递服务JMS模块包含用于生成和使用消息的功能。
- 事务模块支持实现特殊接口和所有pojo的类的编程和声明式事务管理。
 

### WEB
Web层由Web，Web-MVC，Web-Socket和Web-Portlet模块组成
- Web模块提供面向Web的基本集成特性，如多部分文件上传功能，以及使用servlet侦听器和面向Web的应用程序上下文初始化IoC容器。 
- Web-MVC模块包含用于web应用程序的Spring的模型-视图-控制器(MVC)实现。
- Web-Socket模块支持web应用程序中的客户端和服务器之间的基于websocket的双向通信。
- Web-Portlet模块提供了将在portlet环境中使用的MVC实现，并镜像了web servlet模块的功能。

### 杂项
重要模块像AOP，Aspects（切面），仪表，消息和测试模块等  
...  
...  


## Bean定义
构成应用程序主干和由Spring IoC容器管理的对象称为bean。bean是由Spring IoC容器实例化、组装和管理的对象。这些bean是用你提供给容器的配置元数据创建的。例如，XML <bean/>定义的形式

配置一个bean容器需要知道如下信息：
- 如何创建一个bean
- Bean的生命周期细节
- Bean的依赖关系

属性 | 说明
---|---
class | 该属性是强制性的，指定了用于创建bean的类。
name | 该属性唯一的指定了bean标识符。在基于XML的配置元数据中，使用id （and/or）name属性来指定bean标识符。
scope | 指定了从特定的bean定义创建的对象的范围
constructor-arg | 构造参数，用于注入依赖关系
properties | 属性，用于注入依赖关系
autowiring mode | 自动装配模式，用于注入依赖关系
lazy-initialization mode | 延时加载,告诉IoC容器在第一次请求时创建一个bean实例，而不是在启动时
initialization method | 初始化方法，在bean的所有必要属性设置之后立即调用的回调方面
destruction method | 销毁方法，在bean的容器被销毁时要调用的回调函数

### Spring 配置

配置Spring一般包括3中方式
- 基于XML的配置文件
- 基于注解的配置
- 基于Java的配置

基于XML的配置文件示例：
```
<?xml version = "1.0" encoding = "UTF-8"?>

<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- A simple bean definition -->
   <bean id = "..." class = "...">
      <!-- collaborators and configuration for this bean go here -->
   </bean>

   <!-- A bean definition with lazy init set on -->
   <bean id = "..." class = "..." lazy-init = "true">
      <!-- collaborators and configuration for this bean go here -->
   </bean>

   <!-- A bean definition with initialization method -->
   <bean id = "..." class = "..." init-method = "...">
      <!-- collaborators and configuration for this bean go here -->
   </bean>

   <!-- A bean definition with destruction method -->
   <bean id = "..." class = "..." destroy-method = "...">
      <!-- collaborators and configuration for this bean go here -->
   </bean>

   <!-- more bean definitions go here -->
   
</beans>
```

## Bean的范围 
定义一个<bean>时，你可以选择为该bean声明一个范围。例如，为了强制Spring在每次需要时产生一个新的bean实例，你应该声明bean的scope属性是**prototype**。同样，如果你希望Spring在每次需要时都返回相同的bean实例，你应该声明bean的scope属性为**singleton**。

Spring框架支持以下五个范围，其中三个仅在您使用Web感知的ApplicationContext时才可用。


范围 | 说明
---|---
singleton | 单实例，每次都使用同一个bean（默认值），对象创建好之后会被缓存，
prototype | 多实例，每次都返回新的实例
request | 在一次HTTP请求中使用相同实例。<br>只有在Web感知的Spring ApplicationContext的上下文中才有效。
session | 在一个HTTP会话中使用相同实例。<br>只有在Web感知的Spring ApplicationContext的上下文中才有效。
global-session | 在一个全局HTTP会话中使用相同实例。<br>只有在Web感知的Spring ApplicationContext的上下文中才有效。



## Bean的生命周期
### 初始化回调
```
public class ExampleBean implements InitializingBean {
   public void afterPropertiesSet() {
      // do some initialization work
   }
}
```
对于基于XML的配置元数据，可以使用init-method属性来指定具有void无参数签名的方法的名称。例如：
```
<bean id = "exampleBean" class = "examples.ExampleBean" init-method = "init"/>

public class ExampleBean {
   public void init() {
      // do some initialization work
   }
}
```
### 销毁回调
```
public class ExampleBean implements DisposableBean {
   public void destroy() {
      // do some destruction work
   }
}
```
对于基于XML的配置元数据，可以使用destroy-method属性来指定具有void无参数签名的方法的名称。例如：
```
<bean id = "exampleBean" class = "examples.ExampleBean" destroy-method = "destroy"/>

public class ExampleBean {
   public void destroy() {
      // do some destruction work
   }
}
```

### 默认初始化和销毁方法
如果你的bean有太多具有相同名称的初始化和/或销毁方法，则不需要在每个单独的bean上声明init-method和destroy-method。这种情况可以使用<beans>元素上的default-init-method和default-destroy-method属性来配置，如下所示：
```
<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd"
   default-init-method = "init" 
   default-destroy-method = "destroy">

   <bean id = "..." class = "...">
      <!-- collaborators and configuration for this bean go here -->
   </bean>
   
</beans>
```

## Bean Post Processors
BeanPostProcessor接口定义了回调方法，你可以实现这些方法来提供您自己的实例化逻辑、依赖解析逻辑等。还可以在Spring容器完成实例化、配置和初始化bean之后，通过插入一个或多个BeanPostProcessor实现来实现一些自定义逻辑。  
可以配置多个BeanPostProcessor接口，并且您可以通过设置order属性来控制这些BeanPostProcessor接口执行的顺序。

ApplicationContext自动检测由BeanPostProcessor接口实现定义的任何bean，并将这些bean注册为postprocessor，然后容器在创建bean时相应地调用这些bean。

HelloWorld.java文件的内容：
```
package com.tutorialspoint;

public class HelloWorld {
   private String message;

   public void setMessage(String message){
      this.message  = message;
   }
   public void getMessage(){
      System.out.println("Your Message : " + message);
   }
   public void init(){
      System.out.println("Bean is going through init.");
   }
   public void destroy(){
      System.out.println("Bean will destroy now.");
   }
}
```
InitHelloWorld.java文件
```
package com.tutorialspoint;

import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.beans.BeansException;

public class InitHelloWorld implements BeanPostProcessor {
   public Object postProcessBeforeInitialization(Object bean, String beanName) 
      throws BeansException {
      
      System.out.println("BeforeInitialization : " + beanName);
      return bean;  // you can return any other object as well
   }
   public Object postProcessAfterInitialization(Object bean, String beanName) 
      throws BeansException {
      
      System.out.println("AfterInitialization : " + beanName);
      return bean;  // you can return any other object as well
   }
}
```

Beans.xml 文件
```
<?xml version = "1.0" encoding = "UTF-8"?>

<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id = "helloWorld" class = "com.tutorialspoint.HelloWorld"
      init-method = "init" destroy-method = "destroy">
      <property name = "message" value = "Hello World!"/>
   </bean>

   <bean class = "com.tutorialspoint.InitHelloWorld" />

</beans>
```

MainApp.java文件
```
package com.tutorialspoint;

import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MainApp {
   public static void main(String[] args) {
      AbstractApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");

      HelloWorld obj = (HelloWorld) context.getBean("helloWorld");
      obj.getMessage();
      context.registerShutdownHook();
   }
}
```
执行结果：
```
BeforeInitialization : helloWorld
Bean is going through init.
AfterInitialization : helloWorld
Your Message : Hello World!
Bean will destroy now.
```

## Bean定义继承
bean定义可以包含许多配置信息，包括构造函数参数、属性值和容器特定的信息，如初始化方法、静态工厂方法名称等。

子bean定义从父定义继承配置数据。子定义可以覆盖一些值，或者根据需要添加其他值。
Spring Bean定义继承与Java类继承没有关系，但是继承概念是相同的。可以将父bean定义定义定义为模板，其他子bean可以从父bean继承所需的配置。

当使用基于xml的配置元数据时，您通过使用父属性指定子bean定义，指定父bean作为该属性的值。

```
<?xml version = "1.0" encoding = "UTF-8"?>

<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id = "helloWorld" class = "com.tutorialspoint.HelloWorld">
      <property name = "message1" value = "Hello World!"/>
      <property name = "message2" value = "Hello Second World!"/>
   </bean>

   <bean id =" helloIndia" class = "com.tutorialspoint.HelloIndia" parent = "helloWorld">
      <property name = "message1" value = "Hello India!"/>
      <property name = "message3" value = "Namaste India!"/>
   </bean>
</beans>
```

### 定义模板

定义Bean定义模板时，不应指定类属性
```
<?xml version = "1.0" encoding = "UTF-8"?>

<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id = "beanTeamplate" abstract = "true">
      <property name = "message1" value = "Hello World!"/>
      <property name = "message2" value = "Hello Second World!"/>
      <property name = "message3" value = "Namaste India!"/>
   </bean>

   <bean id = "helloIndia" class = "com.tutorialspoint.HelloIndia" parent = "beanTeamplate">
      <property name = "message1" value = "Hello India!"/>
      <property name = "message3" value = "Namaste India!"/>
   </bean>
   
</beans>
```
父bean不能自行实例化，因为它是不完整的，并且它也明确标记为抽象。当定义像这样抽象时，它只能用作纯模板bean定义，作为子定义的父定义。


## 依赖注入
依赖注入主要包括基于构造函数的依赖注入和基于Setter的依赖注入  

基于setter的注入但使用内部bean的配置
```
<?xml version = "1.0" encoding = "UTF-8"?>

<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- Definition for textEditor bean using inner bean -->
   <bean id = "textEditor" class = "com.tutorialspoint.TextEditor">
      <property name = "spellChecker">
         <bean id = "spellChecker" class = "com.tutorialspoint.SpellChecker"/>
      </property>
   </bean>

</beans>
```

### 注入集合
包括list、set、map、props
```
<?xml version = "1.0" encoding = "UTF-8"?>

<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- Definition for javaCollection -->
   <bean id = "javaCollection" class = "com.tutorialspoint.JavaCollection">
      
      <!-- results in a setAddressList(java.util.List) call -->
      <property name = "addressList">
         <list>
            <value>INDIA</value>
            <value>Pakistan</value>
            <value>USA</value>
            <value>USA</value>
         </list>
      </property>

      <!-- results in a setAddressSet(java.util.Set) call -->
      <property name = "addressSet">
         <set>
            <value>INDIA</value>
            <value>Pakistan</value>
            <value>USA</value>
            <value>USA</value>
         </set>
      </property>

      <!-- results in a setAddressMap(java.util.Map) call -->
      <property name = "addressMap">
         <map>
            <entry key = "1" value = "INDIA"/>
            <entry key = "2" value = "Pakistan"/>
            <entry key = "3" value = "USA"/>
            <entry key = "4" value = "USA"/>
         </map>
      </property>
      
      <!-- results in a setAddressProp(java.util.Properties) call -->
      <property name = "addressProp">
         <props>
            <prop key = "one">INDIA</prop>
            <prop key = "one">INDIA</prop>
            <prop key = "two">Pakistan</prop>
            <prop key = "three">USA</prop>
            <prop key = "four">USA</prop>
         </props>
      </property>
   </bean>

</beans>
```
将bean引用注入为集合元素中，使引用和值混合在一起
```
<?xml version = "1.0" encoding = "UTF-8"?>

<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- Bean Definition to handle references and values -->
   <bean id = "..." class = "...">

      <!-- Passing bean reference  for java.util.List -->
      <property name = "addressList">
         <list>
            <ref bean = "address1"/>
            <ref bean = "address2"/>
            <value>Pakistan</value>
         </list>
      </property>
      
      <!-- Passing bean reference  for java.util.Set -->
      <property name = "addressSet">
         <set>
            <ref bean = "address1"/>
            <ref bean = "address2"/>
            <value>Pakistan</value>
         </set>
      </property>
      
      <!-- Passing bean reference  for java.util.Map -->
      <property name = "addressMap">
         <map>
            <entry key = "one" value = "INDIA"/>
            <entry key = "two" value-ref = "address1"/>
            <entry key = "three" value-ref = "address2"/>
         </map>
      </property>
   </bean>

</beans>
```
要使用上面的bean定义，你需要定义你的setter方法，以便它们也能够处理引用。

### 注入空和空字符串值
空字符串作为值:
```
<bean id = "..." class = "exampleBean">
   <property name = "email" value = ""/>
</bean>
```
传递一个NULL值
```
<bean id = "..." class = "exampleBean">
   <property name = "email"><null/></property>
</bean>
```
等同于：exampleBean.setEmail(null)



