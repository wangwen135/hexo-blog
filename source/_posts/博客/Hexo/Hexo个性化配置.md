---
title: Hexo个性化配置
date: 2024-07-25 
tags: 
  - Hexo
categories:
  - [博客, Hexo]
---


### _config.yml配置文件说明

#### 站点基本信息
```
# 站点名称
title: 王某某的笔记

# 站点副标题
subtitle: 记录我的编程之路

# 站点描述
description: 这是一个使用 Hexo 构建的博客，用于分享我的编程经验和学习笔记。

# 作者名称
author: 王某某

# 站点语言
language: zh-CN

```

#### 站点的 URL 结构
```
# URL 前缀，例如如果站点部署在子目录下，需要设置此项
root: /

# 生成文件的目录
public_dir: public

# Source 文件夹
source_dir: source

# 默认布局
default_layout: post

# permalink 配置
permalink: :year/:month/:day/:title/

```

> 这里将permalink:   
> permalink: :title/


#### 设置站点的时间格式和时区
```
# 时间格式
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# 时区设置
timezone: Asia/Shanghai

```


#### 配置站点的分页功能
```
# 每页显示的文章数
per_page: 10

# 分页路径
pagination_dir: page

```


#### 配置404页面
在站点根目录创建一个名为404的新文件夹，然后在其中创建一个新页面：
```
cd hexo-site
hexo new page 404

```
确保relative_link已禁用
```
relative_link: false

```

> 能否将用户重定向到 404 页面取决于网站托管服务商或 Web 服务器的设置，而非 Hexo 的设置。例如，如果您使用 Nginx 作为服务器，您还需要在nginx.conf文件中配置 404 页面。


----


### 使用NexT主题
**NexT** 主题

github地址：  
https://github.com/next-theme/hexo-theme-next

详细安装说明：  
https://theme-next.js.org/docs/getting-started/installation.html
 
#### 安装
使用npm安装
```
cd hexo-site
npm install hexo-theme-next
```
或者你可以克隆整个存储库：
```
cd hexo-site
git clone https://github.com/next-theme/hexo-theme-next themes/next
```


#### 启用主题
修改：Hexo的 _config.yml 文件
```
theme: next
```
> 默认主题是：landscape


#### 复制主题配置文件

将主题的配置文件复制到根目录，命名为：_config.next.yml

```
cd hexo-site

## npm方式安装则：
cp ./node_modules/hexo-theme-next/_config.yml _config.next.yml

## git克隆则：
cp ./themes/next/_config.yml _config.next.yml

```

然后相关配置都是修改 **_config.next.yml** 这个配置文件


#### 选择方案
Scheme 是 NexT 支持的一个功能，通过使用 Scheme，NexT 可以为您提供不同的视图。
到目前为止，NexT 支持 4 种方案，它们是：

- Muse→ Default Scheme，这是NexT的初始版本。使用黑白色调，主要看起来很干净。
- Mist→ Muse 的更紧凑版本，具有整洁的单列视图。
- Pisces→ 双列方案，像邻居的女儿一样新鲜。
- Gemini→ 看起来像双鱼座，但有明显的带阴影的柱块，看起来更加敏感。

可以通过编辑来更改方案NexT 配置文件，搜索scheme关键字

```
# Schemes
scheme: Muse
#scheme: Mist
scheme: Pisces
#scheme: Gemini
```
> 这里选Pisces

#### 侧边栏头像设置

```
# Sidebar Avatar
avatar:
  # Replace the default image and set the url here.
  url: /images/avatar.jpg
  # If true, the avatar will be displayed in circle.
  rounded: true
  # If true, the avatar will be rotated with the cursor.
  rotated: false
```

将头像图片avatar.jpg 放到 ./source/images/ 目录中


#### 配置菜单项
菜单设置项包含 3 个值：  
```
Key: /link/ || icon
```
- Key→ 是菜单项的名称（home、archives等）。如果在语言中可以找到此菜单的翻译，则将加载此翻译；如果没有，Key则将使用名称。
- 分隔符之前的值||（/link/）→ 是您网站内相对 URL 的目标链接。
- ||分隔符 ( ) →后的值icon是 Font Awesome 图标的名称。该图标的名称可以在Font Awesome [https://fontawesome.com/search]
中找。

> 默认情况下，所有菜单项均被注释掉，以确保您可以在备用主题配置中覆盖它们。

这里修改为：
```
menu:
  home: / || fa fa-home
  about: /about/ || fa fa-user
  tags: /tags/ || fa fa-tags
  categories: /categories/ || fa fa-th
  archives: /archives/ || fa fa-archive
```

#### 社交链接
```
# 社交链接设置
social:
  GitHub: https://github.com/your-username || fa fa-github
  Weibo: https://weibo.com/your-username || fa fa-weibo
  Zhihu: https://www.zhihu.com/people/your-username || fa fa-zhihu
  Email: mailto:your-email@example.com || fa fa-envelope

```


#### 配置Tags页面
Next主题默认没有Tags页面，点击左侧菜单“标签”会进入一个404页面
只需要增加一个tags页面，将页面类型设置为**tags** 即可

先增加页面
```
> hexo new page tags
INFO  Validating config
INFO  Created: D:\code\blog\blog1\source\tags\index.md
```
再修改页面 ./source/tags/index.md ，设置类型
```
---
title: tags
date: 2024-07-24 22:18:58
type: "tags"
comments: false
---
```

上面菜单已经配置好了，直接重启看效果


#### 配置Categories页面
默认也是没有Categories页面的  
配置同 tags，type设置为：**categories**

增加页面
```
hexo new page categories
```
页面./source/categories/index.md 内容修改为：
```
---
title: categories
date: 2024-07-24 22:29:50
type: "categories"
comments: false
---

```

#### 配置About页面
categories和tags是系统已经定义好的类型，系统会自动填充内容到里面。  
about页面就不行了，得自己写

首先创建页面
```
hexo new page about
```
然后再修改页面：./source/about/index.md  
内容如：
```
---
title: about
date: 2024-07-22 22:40:20
---
### 关于我
程序员  主要做Java

----
### 关于本博客
2024年使用Hexo + Next 重建
```


#### 帖子元数据设置
NexT 支持帖子创建日期、帖子更新日期和帖子类别显示。
```
# Post meta display settings
post_meta:
  item_text: true
  created_at: true
  updated_at:
    enable: false
    another_day: true
  categories: true
```
> 关闭“更新于”


#### 文章字数统计
安装：hexo-word-counter
```
npm install hexo-word-counter
hexo clean
```
Hexo配置（_config.yml）中增加：
```
#文章字数和阅读时间
symbols_count_time:
  # 是否显示文章的字符数
  symbols: true
  # 是否显示文章的预计阅读时间
  time: true
  # 是否显示全部文章的总字符数（站点底部）
  total_symbols: false
  # 是否显示全部文章总阅读时间（站点底部）
  total_time: true
  # AWL 代表 Average Words per Line（每行的平均单词数），用于估算阅读时间。通常用于更精确的阅读时间估算。
  awl: 4
  # WPM 代表 Words Per Minute（每分钟的单词数），用于计算阅读时间。
  wpm: 275
```
Next中有默认设置
```
symbols_count_time:
  separated_meta: true
  item_text_total: false
```
> Next默认显示字数和阅读时长  



#### 其他设置

> 测试时注意：这些设置一般需要重启服务后才能生效

##### 回到顶部
```
back2top:
  # 控制“返回顶部”按钮是否启用
  enable: true
  # 控制“返回顶部”按钮是否显示在侧边栏
  sidebar: false
  # 控制“返回顶部”按钮上是否显示滚动百分比标签
  scrollpercent: false
```
##### 阅读进度条
```
reading_progress:
  enable: true
```

##### 启用书签支持
```
bookmark:
  enable: true
```
>用户只需点击页面左上角的书签图标即可保存滚动位置。当他们下次访问你的博客时，他们可以自动恢复到上一次滚动位置。


##### 深色模式
```
# Dark Mode
darkmode: false
```