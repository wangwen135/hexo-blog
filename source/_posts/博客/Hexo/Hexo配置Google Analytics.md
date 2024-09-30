---
title: Hexo配置Google Analytics
date: 2024-09-29 01:32 
tags: 
  - Hexo
  - Google Analytics
categories:
  - [博客, Hexo ]
---


### 配置Google Analytics

#### 创建Google Analytics 账号

打开  [Google Analytics](https://analytics.google.com/) 并使用你的 Google 帐户登录。

https://analytics.google.com/

按提示创建账号 

![1727364435788.png](https://img.wangwen135.top:23456/note/2024/09/66f57d542223e.png)

#### 创建资源
（就是一个名字）

![1727364603317.png](https://img.wangwen135.top:23456/note/2024/09/66f57dfb93b95.png)

#### 目标选择流量分析
![1727364650222.png](https://img.wangwen135.top:23456/note/2024/09/66f57e2a81a39.png)

#### 配置网站
![1727364735473.png](https://img.wangwen135.top:23456/note/2024/09/66f57e7fc3cd0.png)

#### 获得追踪ID
之后就能得到一个ID，如：G-MXXXXXXXXX

![1727365759366.png](https://img.wangwen135.top:23456/note/2024/09/66f5827fb7bc2.png)

----

### 配置hexo

这里使用的next主题，打开配置文件：_config.next.yml

找到：
```
# Google Analytics
# See: https://analytics.google.com
google_analytics:
  tracking_id: G-XXXXXXXXXX   # 替换为你的 GA4 追踪 ID
  
  # By default, NexT will load an external gtag.js script on your site.
  # If you only need the pageview feature, set the following option to true to get a better performance.
  only_pageview: true   # 如果只需要页面浏览量追踪，可以设置为 true
  
```
> 替换为你的 GA4 追踪 ID

保存配置并重新生成网站

在源代码中应该能看到 Google Analytics 相关的内容：

![1727367707827.png](https://img.wangwen135.top:23456/note/2024/09/66f58a1c24c10.png)


### 测试验证

访问你的网站，并在 Google Analytics 后台的“实时（Real-time）”页面查看是否能看到访客信息。如果能看到你的访问记录，说明 Google Analytics 已成功集成。


当访问页面时会向 www.google-analytics.com 发送一个post请求，内容如：

![1727373391674.png](https://img.wangwen135.top:23456/note/2024/09/66f5a0501cced.png)


> **注意，这个脚本太老了，在google nalytics上看不到请求数据**

----

### 重新配置

使用google提供的代码，通过自定义head部分的方式配置

参考文档：  
https://theme-next.js.org/docs/advanced-settings/custom-files

在 **source** 目录中添加 **_data** 目录，然后添加：**head.njk** 文件

在文件中添加google提供的代码，内容如下：
```
<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-MBG120GJ9P"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'G-MBG120GJ9P');
</script>
```
> 注意替换你自己的ID

然后取消NexT配置文件`_config.next.yml` 中 custom_file_path 下 head 的注释
```
# Define custom file paths.
# Create your custom files in site directory `source/_data` and uncomment needed files below.
custom_file_path:
  head: source/_data/head.njk
  #header: source/_data/header.njk
  #sidebar: source/_data/sidebar.njk
```

### 重新测试验证

重新发布，刷新页面查看效果  

查看源代码可以看到head部分已经添加了上面的内容

请求如：
![1727717457398.png](https://img.wangwen135.top:23456/note/2024/10/66fae0522c24a.png)

实时效果如：
![1727717407814.png](https://img.wangwen135.top:23456/note/2024/10/66fae020a6e3e.png)

