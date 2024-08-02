---
title: 关于 java Agent
date: 2019-05-15 12:01
tags: 
  - Java
  - Java Agent
categories:
  - [Java]
---

### 参考资料

**instrument 规范**
>https://docs.oracle.com/javase/8/docs/api/java/lang/instrument/package-summary.html?is-external=true

**Class VirtualMachine**
>https://docs.oracle.com/javase/8/docs/jdk/api/attach/spec/com/sun/tools/attach/VirtualMachine.html#loadAgent-java.lang.String-

**Interface ClassFileTransformer**
>https://docs.oracle.com/javase/8/docs/api/java/lang/instrument/ClassFileTransformer.html


---


### java agent 实现
代理需要编译成jar包的方式来运行，JAR文件manifest中的属性指定将加载哪个代理类来启动代理。

有下面两种方式可以启动一个agent

#### 一：命令行接口
程序没有启动时可以通过在命令行上指定javaagent的方式来启动代理
```
-javaagent:jarpath[=options]

#如：
java -javaagent:xxx-agent.jar -cp xxx.jar com.wwh.xxxx
```

通过命令行的方式可以指定**多个**代理，并且支持参数。初始化Java虚拟机(JVM)之后，将按照指定代理的顺序调用每个premain方法，然后调用真正的应用程序main方法。每个premain方法必须返回，以便继续启动程序。

agent Jar包中的manifest文件必须包含Premain-Class， 指向代理的入口类，这个类中包含了一个公共静态的premain方法

premain 方法有两种签名，虚拟机会尝试先运行下面这个
```
public static void premain(String agentArgs, Instrumentation inst);
```
如果没有会尝试调用下面这个
```
public static void premain(String agentArgs);
```
当使用命令行选项启动代理时，不会调用agentmain方法。

代理类将由系统类加载器加载(参见ClassLoader.getSystemClassLoader)。这是类加载器，它通常加载包含应用程序主方法的类。premain方法将在与应用程序main方法相同的安全性和类加载器规则下运行。对于代理premain方法可以做什么，没有建模限制。application main可以做的任何事情，包括创建线程，都是合法的。

每个代理都通过agentArgs参数传递其代理选项。代理选项作为单个字符串传递，任何额外的解析都应该由代理本身执行。



#### 二：在虚拟机启动之后再启动代理
程序已经启动后可以通过VirtualMachine 来加载启动代理，如下：
```
VirtualMachine vm = VirtualMachine.attach("2177");
vm.loadAgent(jar);
vm.detach();
```

注意：  
1. 代理JAR的manifest中必须包含属性 Agent-Class。此属性的值是代理类的名称。  
2. 代理类必须实现一个公共静态的 agentmain  方法，如下所示。  
    
启动代理时先尝试运行
```
public static void agentmain(String agentArgs, Instrumentation inst);
```
找不到再尝试运行
```
public static void agentmain(String agentArgs);
```

agentmain方法不能阻塞，这个类同用可以拥有 premain  方法，不过并不会被调用

参数通过如下方式指定：
```
vm.loadAgent(jar, options);
```


#### Manifest 属性说明

代理JAR文件定义了以下清单属性:
1. **Premain-Class**  此属性指定代理类，也就是包含premain方法的类。如果该属性不存在，JVM将中止。注意这是一个类名，而不是文件名或路径。
2. **Agent-Class**  指定代理类，支持在VM启动后启动代理的机制，包含了agentmain 方法的类，如果该属性不存在，则代理将不会启动。注意这是一个类名，而不是文件名或路径。
3. **Boot-Class-Path** 引导类装入器要搜索的路径列表
4. **Can-Redefine-Classes** 布尔值( true 或 false ，不区分大小写)。代理是否可以重新定义类。此属性是可选的，默认为false。
5. **Can-Retransform-Classes** 布尔值( true 或 false ，不区分大小写)。代理是否可以重新转换类。此属性是可选的，默认为false。
6. **Can-Set-Native-Method-Prefix** 布尔值( true 或 false ，不区分大小写)。代理是否可以设置本地方法前缀。此属性是可选的，默认为false。


代理JAR文件可同时具有清单中的Premain-Class和Agent-Class属性。当使用-javaagent选项在命令行上启动代理时，执行Premain-Class属性指定的
代理类，而Agent-Class属性将被忽略。类似地，如果代理在VM启动之后再启动，则执行Agent-Class属性指定的代理类，而忽略Premain-Class属性的值。



---


### 相关类说明

几个关键类和接口

#### VirtualMachine
表示一个java虚拟机
VirtualMachine表示已附加到的Java虚拟机。它所附加的Java虚拟机有时称为目标虚拟机或目标VM。应用程序(通常是managemet控制台或分析器之类的工具)使用虚拟机将代理加载到目标VM中。例如，用Java语言编写的分析器工具可能附加到正在运行的应用程序，并加载其分析器代理来分析正在运行的应用程序。

通过调用带有标识目标虚拟机的标识符的attach方法来获得虚拟机。标识符依赖于实现，但在每个Java虚拟机都在自己的操作系统进程中运行的环境中，标识符通常是进程标识符(或pid)。另外，通过使用从list方法返回的虚拟机描述符列表中获得的VirtualMachineDescriptor调用attach方法来获得虚拟机实例。一旦获得对虚拟机的引用，就使用loadAgent、loadAgentLibrary和loadAgentPath方法将代理加载到目标虚拟机中。loadAgent方法用于加载用Java语言编写并部署在JAR文件中的代理。loadAgentLibrary和loadAgentPath方法用于加载部署在动态库或静态链接到VM并使用JVM工具接口的代理。

除了加载代理之外，虚拟机还提供对目标VM中的系统属性的读访问。这在某些环境中非常有用，比如java。home、os.name或os。arch用于构造将加载到目标VM的代理的路径。

一个启动jmx的例子
```
      // attach to target VM
      VirtualMachine vm = VirtualMachine.attach("2177");

      // start management agent
      Properties props = new Properties();
      props.put("com.sun.management.jmxremote.port", "5000");
      vm.startManagementAgent(props);

      // detach
      vm.detach();
```
虚拟机对于多个并发线程的使用是安全的。

#### ClassFileTransformer
代理提供此接口的实现，以便转换类文件。转换发生在JVM定义类之前。

一个代理提供者需要实现：ClassFileTransformer 接口，来转变class文件，这个接口有一个方法
```
    byte[]
    transform(  ClassLoader         loader,
                String              className,
                Class<?>            classBeingRedefined,
                ProtectionDomain    protectionDomain,
                byte[]              classfileBuffer)
        throws IllegalClassFormatException;
```

此方法的实现可以转换提供的类文件并返回一个新的替换类文件

一旦向addTransformer注册了一个transformer，每个新的类定义和每个类重定义都会调用这个transformer。每个类重新转换时也将调用具有重新转换能力的转换器。  
对新类定义的请求是用  ClassLoader.defineClass 或本地调用触发的。  
对类的重定义请求是通过 Instrumentation.redefineClasses 或本地调用触发的。  
对类重新转换的请求是通过 Instrumentation.retransformClasses 或本地调用触发的。  
在处理请求期间，在类文件字节被验证或应用之前调用转换器。
当有多个转换器时，转换由链接转换调用组成。也就是说，一个转换调用返回的字节数组将成为下一个调用的输入(通过classfileBuffer参数)。

关于transform输入的classfileBuffer参数：  
如果实现方法确定不需要转换，则返回null。否则，它应该创建一个新的byte[]数组，将输入classfileBuffer连同所有所需的转换复制到其中，并返回新数组。不能修改输入classfileBuffer。

在重新转换和重新定义的情况下，转换器必须支持重新定义语义:如果转换器在初始定义期间更改的类稍后被重新转换或重新定义，转换器必须确保第二个类输出类文件是第一个输出类文件的合法重新定义。

如果transformer抛出异常(它没有捕获异常)，后续的transformer仍然会被调用，并且负载、重新定义或重新转换仍然会被尝试。因此，抛出异常的效果与返回null相同。为了防止在transformer代码中生成未检查异常时出现意外行为，transformer可以捕获Throwable。如果转换器认为classFileBuffer不代表一个有效格式化的类文件，它应该抛出一个IllegalClassFormatException;而这与返回null具有相同的效果。它有助于记录或调试格式错误。

**参数说明：**  
1. loader   要转换的类的定义类加载器，如果是bootstrap loader则为空
2. className  类名的内部形式为Java虚拟机规范中定义的完全限定类名和接口名。例如："java/util/List"。
3. classBeingRedefined  如果这是由重新定义或重新转换触发的，则这个类存在重新定义或重新转换，否则为null。
4. protectionDomain 正在定义或重新定义的类的保护域
5. classfileBuffer 类文件格式的输入字节缓冲区-**不能修改**

**返回：**  
格式良好的类文件缓冲区(转换的结果)，如果没有执行转换，则为null。


#### Instrumentation

该类提供测试Java编程语言代码所需的服务。插装是将字节码添加到方法中，以便收集工具使用的数据。由于这些更改纯粹是附加的，所以这些工具不会修改应用程序状态或行为。此类良性工具的例子包括监视代理、分析器、覆盖率分析器和事件日志记录器。

获取Instrumentation  接口实例有两种方法:  
1. 当JVM以指示代理类的方式启动时。在这种情况下，将一个插装实例传递给代理类的premain方法。
2. 当JVM在启动后的某个时候提供启动代理的机制时。在这种情况下，将一个插装实例传递给代理代码的agentmain方法。
 
一旦代理获得一个Instrumentation 实例，代理可以在任何时候调用该实例上的方法。

```
Instrumentation.addTransformer(new Transformer(), true);  
```
第二个参数表示是否可以重新转换已经定义好了的类  
对于启动后再附加agent的方式，如果想要改变已经加载了的类，需要设置为true

并且注意修改manifest文件中的
```
Can-Retransform-Classes: true
```

否则会报错：
```
adding retransformable transformers is not supported in this environment
```


---


#### 示例

##### pom 文件示例：
```
<dependencies>
	<dependency>
		<groupId>jdk.tools</groupId>
		<artifactId>jdk.tools</artifactId>
		<version>1.8</version>
		<scope>system</scope>
		<systemPath>${JAVA_HOME}/lib/tools.jar</systemPath>
	</dependency>
</dependencies>

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
						<!-- 参数方式启动agent需要这个 -->
						<Premain-Class>
							com.wwh.agentmain.AgentMain
						</Premain-Class>

						<!-- 启动后附加启动agent需要这个 -->
						<Agent-Class>
							com.wwh.agentmain.AgentMain
						</Agent-Class>

						<!-- 是否可以重新转换类 -->
						<Can-Retransform-Classes>
							true
						</Can-Retransform-Classes>

						<!-- 是否可以重新定义类 -->
						<Can-Redefine-Classes>
							true
						</Can-Redefine-Classes>

					</manifestEntries>
				</archive>
			</configuration>
		</plugin>
	</plugins>
</build>
```

##### manifest文件示例：
META-INF/MANIFEST.MF
```
Manifest-Version: 1.0
Premain-Class: com.wwh.agentmain.AgentMain
Archiver-Version: Plexus Archiver
Built-By: Administrator
Agent-Class: com.wwh.agentmain.AgentMain
Can-Redefine-Classes: true
Can-Retransform-Classes: true
Created-By: Apache Maven 3.5.3
Build-Jdk: 1.8.0_151
```


---

### 代码例子

https://github.com/wangwen135/java-agent-test
