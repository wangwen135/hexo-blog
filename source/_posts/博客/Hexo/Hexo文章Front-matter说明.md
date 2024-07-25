---
title: Hexo文章Front-matter说明
date: 2024-07-24 
tags: 
  - Hexo
categories:
  - [博客, Hexo]
---


https://hexo.io/zh-cn/docs/front-matter

Front-matter 是文件开头的 YAML 或 JSON 代码块，用于配置写作设置。 以 YAML 格式书写时，Front-matter 以三个破折号结束；以 JSON 格式书写时，Front-matter 以三个分号结束。

**YAML**
```
---
title: Hello World
date: 2013/7/13 20:46:25
---
```

**JSON**
```
"title": "Hello World",
"date": "2013/7/13 20:46:25"
;;;
```


### 设置 & 默认值

设置	          | 描述	                       | 默认值|
---| ---|---
layout	          | 布局	                       |  config.default_layout
title	          | 标题	                       |  文章的文件名
date	          | 建立日期	                   |  文件建立日期
updated	          | 更新日期	                   |  文件更新日期
comments	      | 开启文章的评论功能	           |   true
tags	          | 标签（不适用于分页）	       |
categories	      | 分类（不适用于分页）	       |
permalink	      | 覆盖文章的永久链接. 永久链接应该以 / 或 .html 结尾	| null
excerpt	          | 纯文本的页面摘要。 使用 该插件 来格式化文本	        |
disableNunjucks	  | 启用时禁用 Nunjucks 标签 { {  } }/{ %  % } 和 标签插件 的渲染功能  |	false
lang	          | 设置语言以覆盖 自动检测        |	继承自 _config.yml
published	      | 文章是否发布                   |	对于 _posts 下的文章为 true，对于 _draft 下的文章为 false




### 分类和标签

只有文章支持分类和标签。  

分类按顺序应用于帖子，从而产生分类和子分类的层次结构。标签都定义在相同的层次结构上，因此它们的出现顺序并不重要。

```
categories:
  - Sports
  - Baseball
tags:
  - Injury
  - Fight
  - Shocking
```


#### 多级分类
如创建多个文章，其分类定义如：
```
categories:
  - [Java, Spring]
```
```
categories:
  - [Java, Spring, Spring MVC]
```
```
categories:
  - [Java, SpringBoot]
```

则展示效果如：
![1721879855319.png](https://img.wangwen135.top:23456/note/2024/07/66a1cd3524da0.png)