---
title: Grafana仪表盘Angular依赖问题
date: 2024-8-21 23:54
tags: 
  - 监控与可视化
  - Grafana
  - Docker
categories:
  - [DevOps, 监控与可视化]
---

![1724251354622.png](https://img.wangwen135.top:23456/image/2024/08/66c5fcdd97018.png)

This dashboard depends on Angular, which is deprecated and will stop working in future releases of Grafana.

> 该仪表板依赖于 Angular，后者已被弃用，并将在 Grafana 的未来版本中停止工作。


官方说明：  
https://grafana.com/docs/grafana/latest/developers/angular_deprecation/

Angular 插件支持已弃用，并将在未来版本中删除。有些旧版核心 Grafana 可视化和外部插件依赖 Grafana 的 Angular 插件支持才能工作。

从 Grafana v9 及更高版本开始，有一个服务器配置选项，它是整个实例的全局选项，用于控制是否提供 Angular 插件支持。在 Grafana 11 中，将更改配置的默认值以删除支持。

在 Grafana 11 中（该版本将于 2024 年 4 月发布预览版，并于 5 月正式发布），将更改 **angular_support_enabled** 配置参数的默认值为：false，以关闭对基于 AngularJS 的插件的支持。如果您仍然依赖内部或社区开发的基于 AngularJS 的插件，则需要启用此选项才能继续使用它们。  
新的 Grafana Cloud 用户将无法请求将支持添加到他们的实例中。

目前的计划是完全删除版本 12 中对 Angular 插件的任何剩余支持。包括删除angular_support_enabled配置参数。



### 启用Angular支持
> 这里的版本是： Grafana  **V11.1.4**

### 进入容器：
```
docker exec -it --user root grafana /bin/bash
```

### 编辑 Grafana 配置文件
```
vi /etc/grafana/grafana.ini
```
在配置文件中查找或添加以下内容：

```
angular_support_enabled = true
```
配置文件中默认是：
```
;angular_support_enabled = false
```
> 这个分号表示注释

### 重启 Grafana 容器
```
docker restart grafana
```

