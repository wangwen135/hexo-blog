---
title: Spring中的事件处理
date: 2018-06-04 23:41
tags: 
  - Spring
categories:
  - [Spring]
---

Spring的核心是ApplicationContext，它管理bean的整个生命周期。ApplicationContext在加载Bean时发布某些类型的事件。例如，ContextStartedEvent在上下文启动时发布，而ContextStoppedEvent在上下文停止时发布。

ApplicationContext中的事件处理通过ApplicationEvent类和ApplicationListener接口提供。如果一个bean实现了ApplicationListener，那么每当一个ApplicationEvent被发布到ApplicationContext时，该bean就会被通知。

Spring提供了以下标准事件

事件 | 描述
---|---
ContextRefreshedEvent | 此事件在ApplicationContext初始化或刷新时发布。这也可以使用ConfigurableApplicationContext接口上的refresh（）方法引发。
ContextStartedEvent | 使用ConfigurableApplicationContext接口上的start（）方法启动ApplicationContext时，会发布此事件。
ContextStoppedEvent | 使用ConfigurableApplicationContext接口上的stop（）方法停止ApplicationContext时发布此事件。
ContextClosedEvent | 使用ConfigurableApplicationContext接口上的close（）方法关闭ApplicationContext时发布此事件。一个关闭的上下文达到其生命的尽头，它不能被刷新或重新启动。
RequestHandledEvent | 这是一个特定于web的事件，它告诉所有bean已经提供HTTP请求。

>Spring的事件处理是单线程的，一个事件被发布，直到所有的接收者都处理完这个消息之前，进程会被阻塞，流程将不会继续。因此，如果要使用事件处理，应在设计应用程序时小心谨慎。

### 听上下文事件
要监听上下文事件，bean应该实现只有一个方法onApplicationEvent（）的ApplicationListener接口。

HelloWorld.java文件
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
}
```
CStartEventHandler.java文件
```
package com.tutorialspoint;

import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextStartedEvent;

public class CStartEventHandler 
   implements ApplicationListener<ContextStartedEvent>{

   public void onApplicationEvent(ContextStartedEvent event) {
      System.out.println("ContextStartedEvent Received");
   }
}
```
CStopEventHandler.java文件
```
package com.tutorialspoint;

import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextStoppedEvent;

public class CStopEventHandler 
   implements ApplicationListener<ContextStoppedEvent>{

   public void onApplicationEvent(ContextStoppedEvent event) {
      System.out.println("ContextStoppedEvent Received");
   }
}
```
MainApp.java文件
```
package com.tutorialspoint;

import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MainApp {
   public static void main(String[] args) {
      ConfigurableApplicationContext context = 
         new ClassPathXmlApplicationContext("Beans.xml");

      // Let us raise a start event.
      context.start();
	  
      HelloWorld obj = (HelloWorld) context.getBean("helloWorld");
      obj.getMessage();

      // Let us raise a stop event.
      context.stop();
   }
}
```
配置文件Beans.xml
```
<?xml version = "1.0" encoding = "UTF-8"?>

<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id = "helloWorld" class = "com.tutorialspoint.HelloWorld">
      <property name = "message" value = "Hello World!"/>
   </bean>

   <bean id = "cStartEventHandler" class = "com.tutorialspoint.CStartEventHandler"/>
   <bean id = "cStopEventHandler" class = "com.tutorialspoint.CStopEventHandler"/>

</beans>
```
程序运行结果
```
ContextStartedEvent Received
Your Message : Hello World!
ContextStoppedEvent Received
```

## 自定义事件
CustomEvent.java文件
```
package com.tutorialspoint;

import org.springframework.context.ApplicationEvent;

public class CustomEvent extends ApplicationEvent{
   public CustomEvent(Object source) {
      super(source);
   }
   public String toString(){
      return "My Custom Event";
   }
}
```
CustomEventPublisher.java文件
```
package com.tutorialspoint;

import org.springframework.context.ApplicationEventPublisher;
import org.springframework.context.ApplicationEventPublisherAware;

public class CustomEventPublisher implements ApplicationEventPublisherAware {
   private ApplicationEventPublisher publisher;
   
   public void setApplicationEventPublisher (ApplicationEventPublisher publisher) {
      this.publisher = publisher;
   }
   public void publish() {
      CustomEvent ce = new CustomEvent(this);
      publisher.publishEvent(ce);
   }
}
```
CustomEventHandler.java文件
```
package com.tutorialspoint;

import org.springframework.context.ApplicationListener;

public class CustomEventHandler implements ApplicationListener<CustomEvent> {
   public void onApplicationEvent(CustomEvent event) {
      System.out.println(event.toString());
   }
}
```
MainApp.java文件
```
package com.tutorialspoint;

import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MainApp {
   public static void main(String[] args) {
      ConfigurableApplicationContext context = 
         new ClassPathXmlApplicationContext("Beans.xml");
	  
      CustomEventPublisher cvp = 
         (CustomEventPublisher) context.getBean("customEventPublisher");
      
      cvp.publish();  
      cvp.publish();
   }
}
```
配置文件Beans.xml
```
<?xml version = "1.0" encoding = "UTF-8"?>

<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id = "customEventHandler" class = "com.tutorialspoint.CustomEventHandler"/>
   <bean id = "customEventPublisher" class = "com.tutorialspoint.CustomEventPublisher"/>

</beans>
```
运行结果
```
My Custom Event
My Custom Event
```



