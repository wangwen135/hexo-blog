---
title: Hexo安装使用
date: 2024-07-24 
tags: 
  - Hexo
categories:
  - [博客, Hexo]
---



https://hexo.io/zh-cn/

https://hexo.io/docs/

Hexo 是一个快速、简洁且高效的博客框架。 Hexo 使用 Markdown（或其他标记语言）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

> 使用 Node.js 编写


### 安装 Node.js 和 git

Hexo 需要 Node.js 和 npm（Node.js 的包管理器）。你可以从 Node.js 官网 下载并安装最新版本的 Node.js。安装 Node.js 后，npm 会自动随之安装。

> Node.js (Node.js 版本需不低于 10.13，建议使用 Node.js 12.0 及以上版本)

git 正常安装即可


### 安装 Hexo
打开终端或命令提示符，运行以下命令来全局安装 Hexo：

```
npm install -g hexo-cli

```
> 有点慢，好像需要科学上网  
> 可以指定国内镜像：--registry=https://registry.npmmirror.com


### 创建新的 Hexo 博客
选择一个你希望存放博客的目录，并进入该目录，然后运行：

```
hexo init blog
```

这将会在 blog 目录下创建一个新的 Hexo 博客项目。你也可以将 blog 替换为你希望的项目名称。


### 安装依赖
进入你的博客目录后，安装项目依赖：
```
cd blog
npm install

#或者指定淘宝镜像
npm install --registry=https://registry.npmmirror.com
```

### 创建新的文章
使用 Hexo 提供的命令来创建新的文章：

```
hexo new "我的第一篇文章"

```
这会在 source/_posts 目录下生成一个新的 Markdown 文件，你可以在这个文件中编写你的文章。


### 生成和预览网站
在生成网站之前，你可以先在本地预览网站：

```
hexo server
```

然后在浏览器中访问 http://localhost:4000 来查看你的博客。

当你对博客的内容感到满意时，可以生成静态文件：
```
hexo generate
```

生成的静态文件会放在 public 目录下。


----


### 部署博客
Hexo 支持多种部署方式，如 GitHub Pages、Netlify 等。以下是将博客部署到 GitHub Pages 的步骤：

1. 在 GitHub 上创建一个新的仓库。
2. 修改 Hexo 配置文件 _config.yml 中的部署设置，添加你的仓库地址。例如：
```
deploy:
  type: git
  repo: https://github.com/你的用户名/你的仓库名.git
  branch: main

```
3. 安装 Hexo 部署插件：
```
npm install hexo-deployer-git --save
```
4. 部署到 GitHub Pages：
```
hexo deploy
```


----


### 配置和主题

Hexo 的配置文件 _config.yml 可以用来设置网站的基本信息，如标题、描述、作者等。你还可以通过安装和配置主题来改变博客的外观，主题通常放在 themes 目录下。

要添加新的主题，你可以从 Hexo 官方主题库 下载，然后将其放在 themes 目录下，修改 _config.yml 中的 theme 选项以使用新的主题。