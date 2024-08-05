---
title: Nexus私服搭建与使用
date: 2017-04-19 23:41
tags: 
  - Maven
categories:
  - [Maven]
---


## 私服（存储库管理器）

存储库管理器是专用于管理二进制组件存储库的服务器应用程序。
对于Maven的大量使用，使用存储库管理器被认为是必不可少的最佳实践。

### 搭建私服的目的
- 充当公共Maven存储库的专用代理服务器
- 提供存储库作为您的Maven项目输出的部署目标

### 特点和好处
- 大大减少了从远程存储库下载的次数，节省了时间和带宽，从而提高了构建性能
- 由于减少了对外部存储库的依赖，提高了构建稳定性
- 与远程SNAPSHOT存储库交互的性能提高
- 控制消耗和提供的伪镜像的能力
- 将构建输出暴露给其他项目和开发人员等消费者，以及质量保证或运营团队甚至客户
- 提供了一个有效的平台来交换组织内外的二进制工件，而无需从源代码构建工件


## NEXUS 私服搭建
**NEXUS OSS 是我们常用的私服软件**

Nexus是一个强大的Maven存储库管理器，它极大地简化了自己内部仓库的维护和外部仓库的访问。  
利用Nexus你可以只在一个地方就能够完全控制访问和部署在你所维护仓库中的每个Artifact。

### 下载

```
wget https://sonatype-download.global.ssl.fastly.net/nexus/oss/nexus-2.13.0-01-bundle.tar.gz
```

### 安装配置
nexus 不建议以root用户运行
新添加一个用户：
```
useradd nexus

passwd nexus

```
切换到nexus用户
```
tar -zxvf nexus-2.13.0-01-bundle.tar.gz
```

解压后会在同级目录中，出现两个文件夹：`nexus-2.13.0-01`和`sonatype-work`，前者包含了nexus的运行环境和应用程序，后者包含了你自己的配置和数据。  

**关于目录：**  
`sonatype-work/nexus/storage/central` : 用于放置maven从中央仓库中下载下来的项目  
pom.xml中配置到的相关jar包  
>**注意**：nexus默认的jar包存储位置是:`sonatype-work/nexus/storage/central`由于Central仓库占用存储较大,所以要注意存储位置。

`sonatype-work/nexus/storage/thirdparty` : 用于放置自己手动上传的第三方jar包  

`sonatype-work/nexus/storage/releases` : 用于放置项目deploy后的发布版  

### 运行测试

进入目录 nexus-2.13.0-01  
输入nexus可以看到提示
```
bin/nexus
```
>Usage: bin/nexus { console | start | stop | restart | status | dump }

启动
```
bin/nexus start
```
>Starting Nexus OSS...  
>Started Nexus OSS.

查看控制台：
```
bin/nexus console
```

访问地址：http://192.168.1.250:8081/nexus  

登录默认账号/密码 ==admin/admin123==

### 设置开机启动

复制nexus 到/etc/init.d/
```
cp bin/nexus /etc/init.d/
```
修改文件
```
vi  /etc/init.d/nexus

#修改下面两个配置

#修改为解压缩的目录
NEXUS_HOME="/xxxx/nexus/nexus-2.13.0-01"

RUN_AS_USER=nexus

```
添加服务并设置为开机启动

```
chkconfig --add nexus
chkconfig nexus on

systemctl start nexus

systemctl status nexus
```


### 配置

#### 1. 下载索引  
打开 Repositories 将列表中所有Type为proxy 的项目的 Configuration 中的 Download Remote Indexes 设置为*True*  
然后在Central 仓库上右键然后点击 Repair Index 下载中心仓库的索引文件，若干时间后，可以点击下边的 Browse Index 即可看见下载的索引文件  
可以通过左侧到任务调度查看任务

#### 2. 关于部署  
将Releases仓库的Deployment Policy设置为*Allow ReDeploy*  
设置 deployment 账户密码  
左侧Security --> Users --> deployment --> 右键Set Password

#### 3. 使用阿里云的maven镜像库
在 Repositories 中增加一个 proxy

key | value
---|---
Repository ID | Aliyun-mavne
Repository Name | Aliyun
Repository type | proxy
Provider | Maven2
Format | maven2
Repository Policy | Release
Remote Storage Location | https://maven.aliyun.com/repository/public/
Download Remote Indexes | True
Auto Blocking Enabled | True
File Content Validation | True
Checksum Policy | Warn
Allow File Browsing | True
Include in Search | True
Publish URL | True

#### 4. Public Repositories
public repository默认包含本地仓库的Releases, snapshots和3rd party以及代理仓库的Maven Central. 你可以在Configuration配置页添加其他仓库到这个仓库组。  
请注意，“Ordered Group Repositories”中列出的存储库库 顺序 很重要。当存储库管理器在组中搜索构建时，它将返回第一个匹配项。  
> 建议将添加的阿里云maven镜像移动到 Central 前面  
> 一般顺序为：Release、Snapshot、3rd party、Aliyun、Central


----

----



## 项目中使用NEXUS私服
两种方式：
1. 在项目的POM文件中配置
2. 在全局的setting文件中配置


### 项目POM配置使用Nexus

在项目的 pom.xml 中配置私库地址，pom.xml 的下面添加：
```
 <!-- 仓库地址 -->
<repositories>
	<repository>
		<id>nexus</id>
		<name>Nexus Repository</name>
		<url>http://192.168.19.250:8081/nexus/content/groups/public</url>
		<releases>
			<enabled>true</enabled>
		</releases>
		<snapshots>
			<enabled>true</enabled>
		</snapshots>
	</repository>
</repositories>

<!-- 插件地址 -->
<pluginRepositories>
	<pluginRepository>
		<id>nexus</id>
		<name>Nexus Plugin Repository</name>
		<url>http://192.168.1.250:8081/nexus/content/groups/public</url>
		<releases>
			<enabled>true</enabled>
		</releases>
		<snapshots>
			<enabled>true</enabled>
		</snapshots>	
	</pluginRepository>
</pluginRepositories>

<!-- 打包发布 -->
<distributionManagement>
	<repository>
		<id>releases</id>
		<url>http://192.168.19.250:8081/nexus/content/repositories/releases</url>
	</repository>
	<snapshotRepository>
		<id>snapshots</id>
		<url>http://192.168.19.250:8081/nexus/content/repositories/snapshots</url>
	</snapshotRepository>
</distributionManagement>
```

如要部署构建到nexus，需在maven 的settings.xml文件中添加认证信息  
默认位置在：
```
${maven.home}/conf/settings.xml  
${user.home}/.m2/settings.xml  
```
两边配置文件中的ID要对得上

```
<servers>
    <server>
    	<id>releases</id>
    	<username>deployment</username>
    	<password>123456</password><!--这个密码就是你设置的密码-->
    </server>
    <server>
    	<id>snapshots</id>
    	<username>deployment</username>
    	<password>123456</password>
    </server>
</servers>
```

### Maven的Setting文件配置使用Nexus
为避免在每个项目中重复进行配置，可以将上面的配置写在Setting.xml中  
>settings.xml并不直接支持上面的两个元素，需要利用 profile，然后用activeProfiles来自动激活这个profile，以提供用户全局范围的仓库配置。

```
<profiles>	    
	<profile>
		<id>dev</id>
		<repositories>
			<repository>
				<id>nexus</id>  <!-- 如取名为central可覆盖超级POM中央仓库的配置，最佳实践是这里随便配，然后mirrorOf * -->
				<url>http://192.168.1.250:8081/nexus/content/groups/public</url>
				<releases>
					<enabled>true</enabled>
				</releases>
				<snapshots>
					<enabled>true</enabled>
				</snapshots>
			</repository>
		</repositories>
		<pluginRepositories>
			<pluginRepository>
				<id>nexus</id>
				<url>http://192.168.1.250:8081/nexus/content/groups/public</url>
				<releases>
					<enabled>true</enabled>
				</releases>
				<snapshots>
					<enabled>true</enabled>
				</snapshots>
			</pluginRepository>
		</pluginRepositories>
	</profile>  
</profiles>
```
激活配置
```
<activeProfiles>  
	<activeProfile>dev</activeProfile>
</activeProfiles>  
```

此外还可以在命令上用**-p**参数激活此配置文件，例如：
```
mvn -Pdev ...
```

#### 注意：
Maven会优先使用其他仓库，最后再使用中央仓库。（简单测试了一下）

当Maven需要下载或发布快照版本构件时它会先检查配置的其他仓库，最后再检查central看是否支持该类型的构件，得到正面的答复之后再向相应的仓库发起请求， 如果配置了镜像则根据镜像匹配规则转发。

如果配置了镜像那仓库的URL已经无关紧要，因为所有的请求都会通过镜像访问。  

如果将仓库和插件仓库的ID都为配置为central，可覆盖超级POM中央仓库的配置，并开启快照版本的支持等。    


### 使用镜像
如果你的地理位置附近有一个速度更快的central镜像，或者你想覆盖central仓库配置，或者你想为所有POM使用唯一的一个远程仓库（这个远程仓库代理所有必要的其它仓库），你可以使用settings.xml中的mirror配置。

**使用镜像的原因：**
1. 网上有了一个地理位置更近，速度更快的镜像
2. 使用自己内部存储替换特定的存储，从而更换的控制它
3. 运行了存储库管理软件以向镜像提供本地缓存，而需要使用其URL


配置用某个镜像覆盖Maven自带的central
```
<mirrors>
    <mirror>
      <id>other-mirror</id>
      <name>Other Mirror Repository</name>
      <url>https://other-mirror.repo.other-company.com/maven2</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
</mirrors>
```
> 将mirrorOf配置为central


将mirrorOf设置为 * 对所有请求强制使用单个存储库，这个存储库必须包含所有所需的构件，或者能够将请求代理到其他存储库。
```
<mirrors>
  <mirror>
    <id>nexus</id>
    <mirrorOf>*</mirrorOf>
    <name>Mirror all</name>
    <url>http://192.168.1.91:8081/nexus/content/groups/public</url>
  </mirror>
</mirrors>
```
> 所有的请求都将通过私服，包括下载之外的请求



-----

-----

### 最佳实践
配置镜像
```
<mirror>
    <id>nexus</id>
    <mirrorOf>*</mirrorOf>
    <url>http://nexus.xxx.com:8081/nexus/content/groups/public</url>
</mirror>
```
profile中配置repository
```
<profiles>
	<profile>
    	<id>nexus</id>
    	<repositories>
            <repository>
                <id>nexus</id>
                <url>http://nexus</url><!--地址随意-->
                <releases><enabled>true</enabled></releases>
                <snapshots><enabled>true</enabled></snapshots>
            </repository>
        </repositories>
        <pluginRepositories>
            <pluginRepository>
                <id>nexus</id>
                <url>http://nexus</url>
                <releases><enabled>true</enabled></releases>
                <snapshots><enabled>true</enabled></snapshots>
            </pluginRepository>
        </pluginRepositories>
    </profile>
  </profiles>
```
激活profile
```
<activeProfiles>
  <activeProfile>nexus</activeProfile>
</activeProfiles>
```


----

----



## Nexus其他配置
### 任务调度

左侧导航栏“Administration”中选择“Scheduled Tasks”  
可以打开任务调度面板，在面板中可以看到当前正在运行的任务  
添加任务：ADD --> 选择任务类型 --> 配置运行方式  

运行方式：  
>每天  
每周  
手动等

任务类型:  
>download indexes 为代理仓库下载远程索引  
清空缓存  
发布索引  
重建索引  
reindex repositories 编纂索引  
删除快照构件  



----

----

## 访问授权问题
当 Nexus 关闭匿名访问时可进行如下配置：

```
关闭镜像

覆盖中央仓库配置：
profile -> id(xxx) -> repositories -> repository 
 id 配置为：central
 url 配置为：私服地址

激活配置：
activeProfiles -> activeProfile(xxx)

配置用户名密码：
<servers>
    <server>
    	<id>central</id>
    	<username>admin</username>
    	<password>123456</password>
    </server>
</servers>
```
