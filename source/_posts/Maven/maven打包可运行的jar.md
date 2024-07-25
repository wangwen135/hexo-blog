---
title: maven打包可运行的jar
date: 2024-04-09 23:41
tags: 
  - Maven
categories:
  - [Maven]
---

## 一：maven-jar-plugin
```
<build>
	<plugins>
		<plugin>
			<artifactId>maven-jar-plugin</artifactId>
			<configuration>
				<archive>
					<manifest>
						<mainClass>com.xxx.xxx.Main</mainClass>
					</manifest>
				</archive>
			</configuration>
		</plugin>
	</plugins>
</build>
```
maven-jar-plugin用于生成META-INF/MANIFEST.MF文件的部分内容  
><mainClass>com.xxg.Main</mainClass>

指定MANIFEST.MF中的Main-Class

排除目录
```
<plugin>
	<artifactId>maven-jar-plugin</artifactId>
	<configuration>
		<!-- jar包中排除groovy脚本目录 -->
		<excludes>
			<exclude>groovy/</exclude>
		</excludes>
	</configuration>
</plugin>
```
排除文件，可以用通配符
```
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-jar-plugin</artifactId>
	<version>2.4</version>
	<configuration>
		<!-- jar包中排除这些文件 -->
		<excludes>
			<exclude>log4j.properties</exclude>
			<exclude>conf.properties</exclude>
			<exclude>system.properties</exclude>
			<exclude>logback.xml</exclude>
			<exclude>log4j2.xml</exclude>
		</excludes>
	</configuration>
</plugin>
```


## 二：maven-assembly-plugin

```
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-assembly-plugin</artifactId>
	<version>2.6</version>
	<configuration>
		<descriptorRefs>
			<descriptorRef>jar-with-dependencies</descriptorRef>
		</descriptorRefs>
		<archive>
			<manifest>
				<mainClass>com.xxx.xxx.Main</mainClass>
			</manifest>
		</archive>
	</configuration>
	<executions>
		<execution>
			<id>make-assembly</id>
			<phase>package</phase>
			<goals>
				<goal>single</goal>
			</goals>
		</execution>
	</executions>
</plugin>
```

这里没有指定
```
<descriptor>src/main/assembly/assembly.xml</descriptor>  
```
而是用的
```		
<descriptorRefs>
	<descriptorRef>jar-with-dependencies</descriptorRef>
</descriptorRefs>
```
这个jar-with-dependencies是assembly预先写好的一个，组装描述引用（assembly descriptor）

默认的compile scope范围是会打进jar包的，而且依赖也全部会打进去，所有有些包打完之后有可能会很大，可将scope范围改成provided，就不会打进去了。

指定打包输出的文件名
```
<configuration>
	<finalName>CSV转Excel工具</finalName>
	<appendAssemblyId>false</appendAssemblyId>
......
```

---

---


## 常规做法

一般项目都比较大，不会打成一个jar包，一般都是打成一个tar包，里面包含启动脚本，配置文件等
 
使用assembly插件  


### 一般目录结构如下：
```
│
└─assembly
	│  assembly.xml
	│
	├─bin
	│      dump.sh
	│      restart.sh
	│      server.sh
	│      start.sh
	│      stop.sh
	│
	└─conf
			log4j.properties
			system.properties
```

### pom中配置assembly 插件
```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <descriptor>src/main/assembly/assembly.xml</descriptor>
    </configuration>
    <executions>
        <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
 
### assembly.xml
```
<assembly>
	<id>assembly</id>
	<formats>
		<format>tar.gz</format>
	</formats>
	<includeBaseDirectory>true</includeBaseDirectory>
	<fileSets>
		<fileSet>
			<directory>src/main/assembly/bin</directory>
			<outputDirectory>bin</outputDirectory>
			<fileMode>0755</fileMode>
		</fileSet>
		<fileSet>
			<directory>src/main/assembly/conf</directory>
			<outputDirectory>conf</outputDirectory>
			<fileMode>0644</fileMode>
		</fileSet>
	</fileSets>
	<dependencySets>
		<dependencySet>
			<outputDirectory>lib</outputDirectory>
		</dependencySet>
	</dependencySets>
</assembly>
```
 



