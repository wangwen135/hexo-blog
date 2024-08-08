---
title: 使用 github actions 部署 github pages
date: 2024-8-5
tags: 
  - Github
  - Actions
  - Pages
  - Git
  - Hexo
categories:
  - [博客, Hexo]
---


官方的和我实际想要实现的有一点差异，官方的是部署到当前仓库的 GitHub Pages  

我这边是原来有一个网站，这次又使用Hexo重新做了一个博客，又新建了一个`hexo-blog`仓库

我想实现的效果是，往**hexo-blog** 仓库提交Markdown内容之后自动编译，然后将编译后的内容提交到 **wangwen135.github.io**这个仓库

----

## 官方的

先参考一下官方的文档说明

官方说明文档：https://hexo.io/zh-cn/docs/github-pages

`.github/workflows/pages.yml` 文件内容如下：

```yml
name: Pages

on:
  push:
    branches:
      - main # 当推送到 main 分支时触发工作流

jobs:
  build:
    runs-on: ubuntu-latest # 工作运行在最新的 Ubuntu 环境中
    steps:
      - uses: actions/checkout@v4 # 检出代码
        with:
          token: ${{ secrets.GITHUB_TOKEN }} # 使用 GitHub token
          submodules: recursive # 如果仓库依赖子模块，则递归检出
      - name: Use Node.js 20 # 使用 Node.js 20
        uses: actions/setup-node@v4
        with:
          node-version: "20" # 指定 Node.js 版本为 20
      - name: Cache NPM dependencies # 缓存 NPM 依赖
        uses: actions/cache@v4
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies # 安装依赖
        run: npm install
      - name: Build # 构建项目
        run: npm run build
      - name: Upload Pages artifact # 上传构建产物
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public # 指定上传的路径
  deploy:
    needs: build # 依赖 build 任务
    permissions:
      pages: write # 给予 GitHub Pages 写权限
      id-token: write # 给予 ID 令牌写权限
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest # 工作运行在最新的 Ubuntu 环境中
    steps:
      - name: Deploy to GitHub Pages # 部署到 GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4

```
> 这个 GitHub Actions 配置文件用于将编译后的 HTML 文件自动部署到当前仓库的 GitHub Pages。  
> 这里加上了一点注释

---

## 修改后的

### 完整的 GitHub Actions 工作流 YAML 文件

`.github/workflows/deploy.yml` 
```yml
name: Deploy to GitHub Pages

# 触发条件：当推送到 main 分支时
on:
  push:
    branches:
      - main
  workflow_dispatch: # 允许手动触发

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # 检出 hexo-blog 仓库的代码
      - name: Checkout repository
        uses: actions/checkout@v4

      # 设置 Node.js 环境
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      # 缓存 NPM 依赖
      - name: Cache NPM dependencies
        uses: actions/cache@v4
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-

      # 安装依赖
      - name: Install dependencies
        run: npm install

      # 构建 Hexo 网站
      - name: Build Hexo site
        run: npm run build

      # 部署到 GitHub Pages
      - name: Deploy to GitHub Pages
        env:
          ACTIONS_DEPLOY_KEY: ${{ secrets.ACTIONS_DEPLOY_KEY }}
        run: |
          echo "创建 .ssh 目录"
          mkdir -p ~/.ssh
          echo '写入部署密钥到 id_ed25519 文件中'
          echo "${{ secrets.ACTIONS_DEPLOY_KEY }}" > ~/.ssh/id_ed25519
          echo "设置私钥文件权限"
          chmod 600 ~/.ssh/id_ed25519

          cat ~/.ssh/id_ed25519

          echo "添加 GitHub 的 SSH 主机密钥到 known_hosts 文件中"
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          
          echo "配置 Git 用户信息"
          git config --global user.name 'github-actions[bot]'
          git config --global user.email 'github-actions[bot]@users.noreply.github.com'
          
          echo "克隆 wangwen135.github.io 仓库的 main 分支到 .deploy_repo 目录"
          git clone --branch main git@github.com:wangwen135/wangwen135.github.io.git .deploy_repo

          echo "切换到 .deploy_repo 目录"
          cd .deploy_repo

          echo "更新一下代码"
          git pull

          echo "使用 rsync 将 public 目录中的内容同步到 .deploy_repo 目录"
          rsync -av --delete --exclude '.git' ../public/ .

          echo "添加所有更改"
          git add .
          echo '提交更改，提交信息为 "Deploy updates"'
          git commit -m "Deploy updates"
          echo "推送更改到远程仓库的 main 分支"
          git push
```
> 相关步骤上面都有注释

因为要使用git将文件推送到仓库`wangwen135.github.io` 需要配置ssh key，还需要进行下面的步骤

## 配置 Secrets
在 GitHub Actions 中，Secrets 是一种安全存储和管理敏感信息的方法，例如访问令牌、API 密钥、SSH 密钥等。Secrets 可以在工作流中使用，但不会在日志中显示，确保了敏感信息的安全。

### 设置步骤

#### 生成 SSH 部署密钥
在本地机器上生成 SSH 密钥对：

```
ssh-keygen -t ed25519 -C "wangwen135@gmail.com" -f ./id_ed25519
```
> 按提示操作，将会在当前执行的目录中生成 id_ed25519 和 id_ed25519.pub 两个文件

#### 添加 id_ed25519 到 hexo-blog 的 GitHub Secrets
将生成的私钥 id_ed25519 的内容添加到 hexo-blog 仓库的 GitHub Secrets 中，步骤如下：

- 打开 GitHub，进入你的 hexo-blog 仓库。
- 点击 "Settings" 选项卡。
- 在左侧边栏中，找到并点击 "Secrets and variables" 下的 "Actions"。
- 点击 "New repository secret" 按钮。
- 在 "Name" 字段中输入 **ACTIONS_DEPLOY_KEY**，在 "Secret" 字段中粘贴 id_ed25519 文件的内容。
- 点击 "Add secret" 按钮保存。


#### 添加 id_ed25519.pub 到目标仓库的部署密钥

将生成的公钥 id_ed25519.pub 添加到 wangwen135.github.io 仓库的部署密钥中，步骤如下：

- 打开 GitHub，进入你的 wangwen135.github.io 仓库。
- 点击 "Settings" 选项卡。
- 在左侧边栏中，找到并点击 "Deploy keys"。
- 点击 "Add deploy key" 按钮。
- 在 "Title" 字段中输入一个描述性的名称，例如 "GitHub Actions Deploy Key"。
- 在 "Key" 字段中粘贴 id_ed25519.pub 文件的内容。
- 确保勾选 "Allow write access" 复选框。
- 点击 "Add key" 按钮保存。


