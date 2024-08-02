---
title: java程序ClassLoad加密的通用破解方法
date: 2019-05-15 12:01
tags: 
  - Java
  - Class
  - 加密
  - 字节码
  - JavaAgent
categories:
  - [Java]
---


想要加密java程序是比较困难的，一般的做法就是代码混淆，增加代码阅读的难度，另外就是修改Class文件的字节码，然后再虚拟机加载类的时候进行解密。修改字节码的解密方法一般是修改JDK中的ClassLoad或者自定义一个ClassLoad来解密Class。有些为了不让别人看到解密算法有些还会用C来写解密程序，再用jni来调用。

一般用改ClassLoader来加密的这种方法都可以用 javaagent 获取到解密后class。

>使用 Instrumentation，开发者可以构建一个独立于应用程序的代理程序（Agent），用来监测和协助运行在 JVM 上的程序，甚至能够替换和修改某些类的定义。有了这样的功能，开发者就可以实现更为灵活的运行时虚拟机监控和 Java 类操作了，这样的特性实际上提供了一种虚拟机级别支持的 AOP 实现方式，使得开发者无需对 JDK 做任何升级和改动，就可以实现某些 AOP 的功能了。  
*介绍地址：*  
https://www.ibm.com/developerworks/cn/java/j-lo-jse61/


### 输出Class文件到指定目录的代码

#### 1、新建一个maven项目
如：class-agent

#### 2、写两个类  
**PreMainExecutor.java**
```
package com.wwh.agent;

import java.lang.instrument.Instrumentation;

public class PreMainExecutor {

    public static void premain(String agentOps, Instrumentation inst) {
        System.out.println("premain execute..........");
        System.out.println("参数：" + agentOps);
        // 添加Transformer
        inst.addTransformer(new PrintClassFileAgent(agentOps));

        // 可以用这个来加载jar包
        // inst.appendToSystemClassLoaderSearch(jarfile);
    }
}

```
**PrintClassFileAgent.java**
```
package com.wwh.agent;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.lang.instrument.ClassFileTransformer;
import java.lang.instrument.IllegalClassFormatException;
import java.security.ProtectionDomain;

public class PrintClassFileAgent implements ClassFileTransformer {

    public static final String OUT_FILE_DIR = "/opt/logs/wwh/classFile/";

    private File outFileDir;

    public PrintClassFileAgent(){

    }

    public PrintClassFileAgent(String fileDir){
        String fileOutDir = OUT_FILE_DIR;
        if (fileDir != null && !"".equals(fileDir)) {
            fileOutDir = fileDir;
        }
        outFileDir = new File(fileOutDir);
        outFileDir.mkdirs();

    }

    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined,
                            ProtectionDomain protectionDomain,
                            byte[] classfileBuffer) throws IllegalClassFormatException {
        System.out.println("类加载器：" + loader);
        System.out.println("类名称：" + className);

        String pathName = className.replaceAll("[.]", "/");
        pathName = pathName + ".class";

        File f = new File(OUT_FILE_DIR + pathName);

        f.getParentFile().mkdirs();
        try {
            f.createNewFile();
            FileOutputStream fos = new FileOutputStream(f);
            fos.write(classfileBuffer);
            fos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

        return null;
    }

}

```

#### 3、 修改pom文件
用于指定：Premain-Class
```
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.wwh.agent</groupId>
	<artifactId>class-agent</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<properties>
		<maven.compiler.target>1.8</maven.compiler.target>
		<maven.compiler.source>1.8</maven.compiler.source>
	</properties>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-jar-plugin</artifactId>
				<configuration>
					<archive>
						<manifest>
							<addClasspath>true</addClasspath>
						</manifest>
						<manifestEntries>
							<Premain-Class>
								com.wwh.agent.PreMainExecutor
							</Premain-Class>
						</manifestEntries>
					</archive>
				</configuration>
			</plugin>
		</plugins>
	</build>

</project>
```

### 用法
将上面的maven项目编译打包，将 class-agent-0.0.1-SNAPSHOT.jar 复制到目标位置。

用如下命令启动想要破解的程序：
```
java -javaagent:class-agent-0.0.1-SNAPSHOT.jar -cp runner-0.0.1-SNAPSHOT.jar com.wwh.runner.EmptyRunner

//指定输出Class文件目录的
java -javaagent:class-agent-0.0.1-SNAPSHOT.jar=/opt/xxx/out -cp runner-0.0.1-SNAPSHOT.jar com.wwh.runner.EmptyRunner

```

如果需要破解的程序本身就有使用javaagent，需要将上面的agent放到调用链的最后面。
```
java -javaagent:agentA.jar -javaagent:class-agent-0.0.1-SNAPSHOT.jar XXProgram

```

### 效果
将在目录下保存所有加载的类的class文件
```
E:\opt\logs\wwh\classFile>tree
......
......
    ├─nio
    │  ├─ch
    │  ├─cs
    │  └─fs
    ├─reflect
    │  ├─annotation
    │  └─generics
    │      ├─factory
    │      ├─parser
    │      ├─reflectiveObjects
    │      ├─repository
    │      ├─scope
    │      ├─tree
    │      └─visitor
    ├─security
    │  ├─action
    │  ├─jca
    │  ├─provider
    │  └─util
    ├─text
    │  └─resources
    │      └─zh
    ├─usagetracker
    └─util
        ├─calendar
        ├─locale
        │  └─provider
        ├─logging
        ├─resources
        │  ├─en
        │  └─zh
        └─spi
......        
......

```
