---
title: Gecko-geckodriver-Selenium
date: 2021-10-12 23:41
tags: 
  - Selenium
categories:
  - [Selenium]
---

*wwh 2018年3月24日*

## Gecko

Gecko是套开放原始码的、以C++编写的网页排版引擎。目前为Mozilla家族网页浏览器以及Netscape 6以后版本浏览器所使用。这软件原本是由网景通讯公司开发的，现在则由Mozilla基金会维护。 这套排版引擎提供了一个丰富的程序界面以供因特网相关的应用程序使用，例如网页浏览器、HTML编辑器、客户端/服务器等等。虽然最初的主要对象是Mozilla的衍生产品，如Netscape和Mozilla Firefox，现在已有很多其他软件现在利用这个排版引擎。Gecko是跨平台的，能在Microsoft Windows、Linux和Mac OS X等主要操作系统上运行。

> **排版引擎**  
>网页浏览器的排版引擎也被称为页面渲染引擎，它负责取得网页的内容（HTML、XML、图象等等）、整理信息（例如加入CSS等），以及计算网页的显示方式然后会输出至显示器或打印机。所有网页浏览器、电子邮件客户端以及其它需要编辑、显示网络内容的应用程序都需要排版引擎。

*参考网页：*  
https://baike.baidu.com/item/gecko/7348782?fr=aladdin


## geckodriver
使用W3C WebDriver兼容客户端与基于Gecko的浏览器进行交互的代理。

该程序提供由WebDriver协议描述的HTTP API 与Gecko浏览器（如Firefox）进行通信的功能。它通过充当本地和远程端之间的代理将呼叫转换为Firefox远程协议。

geckodriver是一个单独的HTTP服务器，它是完整的WebDriver远程实现。通过符合W3C WebDriver标准的客户端库（或客户端），您可以与geckodriver HTTP服务器进行交互。

geckodriver是用Mozilla的一种系统编程语言Rust编写的 。最重要的是，它依靠  webdriver crate  来提供HTTPD，并完成大部分编组WebDriver协议的繁重工作。geckodriver将WebDriver 命令，响应和错误转换为Marionette协议，并作为WebDriver和Marionette之间的代理 。

*参考网页：*  
https://github.com/mozilla/geckodriver

### 支持的客户端

Selenium 用户必须更新到3.5或更高版本才能使用geckodriver。遵循W3C WebDriver规范的其他客户端也受支持。

### 支持的Firefoxen
geckodriver尚未完成功能。这意味着它尚未完全符合WebDriver标准或与Selenium完全兼容。  
Firefox 55及更高版本的支持最好，但通常Firefox版本越新，体验越好，因为它们具有更多的错误修复和功能。某些功能只会在最新的Firefox版本中提供。

### 功能
具体见官方文档，包括：
- 证书信任
- 页面加载策略
- 浏览器代理设置
- firefox的二进制文件设置，启动参数、profile、日志等设置。
- 计算指针，获取元素，执行点击操作等等

## 使用
使用文档可以在 [MDN](https://developer.mozilla.org/zh-CN/docs/Mozilla/QA/Marionette/WebDriver) 上进行查看 

下载地址：  
https://github.com/mozilla/geckodriver/releases

### Selenium
如果您通过 Selenium 使用 Geckodriver，则必须确保您拥有3.5或更高版本。由于geckodriver实现了 W3C WebDriver标准，而不是旧版驱动程序正在使用的Selenium wire协议，因此在从FirefoxDriver切换到geckodriver时，可能会遇到不兼容性和迁移问题。

一般来说，Selenium 3启用geckodriver作为Firefox的默认WebDriver实现。随着Firefox 47的发布，FirefoxDriver不得不停止支持 Gecko中新的多处理架构。

除非您通过设置Java VM系统属性 ==webdriver.gecko.driver== 来覆盖它，否则Selenium客户端绑定将从系统环境变量 ==Path== 中选取geckodriver二进制可执行文件。

```
System.setProperty("webdriver.gecko.driver", "/home/user/bin");
```

或者将它作为参数传递给java启动器：
```
% java -Dwebdriver.gecko.driver=/home/user/bin YourApplication
```

根据使用的编程语言不同，绑定的方法可能有所不同。   
但如果在系统的环境变量Path上进行设置，geckodriver将被使用。  
在bash兼容shell中，可以通过导出或设置PATH变量使其他程序知道其位置：
```
% export PATH=$PATH:/home/user/bin
% whereis geckodriver
geckodriver: /home/user/bin/geckodriver
```
在Window系统上，您可以通过右键单击我的电脑并选择属性来更改系统路径。在出现的对话框中，导航高级 → 环境变量 → Path。

或者在Windows控制台窗口中：
```
$ set PATH=%PATH%;C:\bin\geckodriver
```

### 参数
#### --binary
```
-b BINARY/--binary BINARY
```
指定Firefox二进制文件的路径。默认情况下，geckodriver会尝试查找并使用Firefox的系统安装，但是可以使用此选项更改该行为。请注意，创建新会话时传递binary的moz:firefoxOptions对象的功能 将覆盖此选项。

在Linux系统上，它将使用通过搜索环境变量找到的第一个firefox二进制文件PATH，这大致相当于调用 whereis并提取第二列：
```
% whereis firefox
firefox: /usr/bin/firefox /usr/local/firefox
```
在Windows系统上，geckodriver通过扫描Windows注册表来查找系统Firefox。

#### --connect-existing
```
--connect-existing
```
将geckodriver连接到现有的Firefox实例。这意味着geckodriver会放弃启动新的Firefox会话的默认设置。

现有的Firefox实例必须启用Marionette。要在Firefox中启用远程协议，您可以传递该 -marionette标志。除非marionette.port首选项已由用户设置，否则Marionette将在端口2828上进行监听。因此，在使用时--connect-existing，您可能还必须使用 --marionette-port设置正确的端口。

#### --host
```
--host HOST
```
用于WebDriver服务的地址。默认为127.0.0.1。

#### --log
```
--log LEVEL
```
设置Gecko和geckodriver日志级别。可能的值是fatal， error，warn，info，config，debug，和trace。

#### --marionette-port
```
--marionette-port PORT
```
为geckodriver连接到Marionette 远程协议选择端口。

在geckodriver启动并管理Firefox进程的默认模式下，它将选择由系统分配的空闲端口，设置在配置文件中的 marionette.port 将被优先使用。

当使用--connect-existing 时，Firefox进程不在geckodriver的控制下，它只会连接到PORT。

#### --port
```
-p PORT/--port PORT
```
用于WebDriver服务的端口。默认为4444。

一个有用的技巧是可以绑定到0来让系统自动分配一个空闲端口。

#### --jsdebugger
```
--jsdebugger
```
Firefox启动时附加浏览器工具箱调试器。这对调试Marionette内部件很有用。

#### -v[v]
```
-v[v]
```
通过-v 或 -vv 来调整日志记录的详细程度，这类似于传--log debug和--log trace。

-----
### Demo
**java**  
pom文件导入依赖

```
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>3.9.1</version>
</dependency>
```
示例代码：
```
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.firefox.FirefoxDriver;

public class Test {

    public static String BROWSER_PATH = "D:\\Program Files\\Mozilla Firefox\\firefox.exe";

    public static String GECKODRIVER_PATH = "D:\\Program Files\\Mozilla Firefox\\geckodriver.exe";

    public static void main(String[] args) throws InterruptedException {

        System.setProperty("webdriver.firefox.bin", BROWSER_PATH);

        System.setProperty("webdriver.gecko.driver", GECKODRIVER_PATH);

        WebDriver driver = new FirefoxDriver();

        driver.get("http://www.baidu.com");

        Thread.sleep(1000);
        
        WebElement searchBox = driver.findElement(By.id("kw"));

        searchBox.sendKeys("geckodriver");
        
        searchBox.submit();
        
        Thread.sleep(10000);

        driver.quit();
    }
}
```

-----

## 其他
### Marionette

Marionette是一种远程协议，它允许进程外程序与基于Gecko的浏览器进行通信，仪器和控制。

它提供了与基于Gecko的浏览器（如Firefox和Fennec）的内部JavaScript运行时和UI元素进行交互的接口。它可以控制文档和内容文档，提供高级别的控制和复制或模拟用户交互的能力。

#### 用法
```

% firefox -marionette
…
1491228343089   Marionette  INFO    Listening on port 2828
```
这将绑定到一个TCP套接字上，使用这个协议客户端可以和 Marionette 进行通信。
