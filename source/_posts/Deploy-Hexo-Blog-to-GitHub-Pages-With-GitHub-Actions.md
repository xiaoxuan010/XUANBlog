---
title: 使用 GtiHub Actions 自动将 Hexo 博客部署到 GitHub Pages，无需两个仓库
excerpt: 无需两个仓库，无需配置 GitHub Token，无需配置复杂的 Git 操作，使用 GitHub 官方部署工作流，即可快速将 Hexo 博客部署到 GitHub Pages。
date: 2025-03-03 19:26:18
tags: 
  - DevOps
  - Blog 
  - Hexo 
  - GitHub Actions
---

## 参考官方指南

[使用自定义 GitHub Actions 工作流进行发布](https://docs.github.com/zh/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#%E4%BD%BF%E7%94%A8%E8%87%AA%E5%AE%9A%E4%B9%89-github-actions-%E5%B7%A5%E4%BD%9C%E6%B5%81%E8%BF%9B%E8%A1%8C%E5%8F%91%E5%B8%83)
[创建自定义 GitHub Actions 工作流以发布站点](https://docs.github.com/zh/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#%E5%88%9B%E5%BB%BA%E8%87%AA%E5%AE%9A%E4%B9%89-github-actions-%E5%B7%A5%E4%BD%9C%E6%B5%81%E4%BB%A5%E5%8F%91%E5%B8%83%E7%AB%99%E7%82%B9)
GitHub Action: [deploy-pages 🚀](https://github.com/actions/deploy-pages)

## TL;DR

将以下内容添加到现有的 Hexo 仓库中，在 GtiHub 仓库设置（/settings/pages）中将 `Build and deployment > Source` 选择为 `GitHub Actions`，最后 `git push` 即可。

```yaml .github/workflows/deploy.yml
name: Deploy Hexo to GitHub Pages

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '22.14.0'

    - name: Install dependencies
      run: npm install

    - name: Generate static files
      run: npm run build

    - name: Upload static files as artifact
      id: deployment
      uses: actions/upload-pages-artifact@v3
      with:
        path: public

  deploy:
    needs: build
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## 问题引入

Hexo 的本地环境搭建和部署比较复杂，因此很多人选择了使用 GitHub Actions 自动部署 Hexo 博客，例如以下文章：

- [使用Github Action实现Hexo博客自动化部署](https://blog.csdn.net/qq_73142349/article/details/138245304)
- [【Hexo自动部署】优雅的使用 Github Actions 进行 Hexo 静态博客的持续集成与部署](https://cloud.tencent.com/developer/article/2369534)
- [Hexo + GitHub Action，零成本自动化部署个人博客](https://xulianjun.github.io/2024/09/08/Hexo%20+%20GitHub%20Action%EF%BC%8C%E9%9B%B6%E6%88%90%E6%9C%AC%E8%87%AA%E5%8A%A8%E5%8C%96%E9%83%A8%E7%BD%B2%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/)

然而，这些文章一般将 Hexo 仓库和 GitHub Pages 仓库分开，通过 GitHub Actions 将静态页面部署到另一个仓库。这样做的好处是可以将 Hexo 仓库设置为私有仓库，而 GitHub Pages 仓库设置为公开仓库，这样可以保护 Hexo 源码。但是这样做有以下缺陷：

- 在仓库列表观感上不够简洁，需要刻意区分源代码与部署产物。
- 可能会占用 `USERNAME.github.io` 的仓库名，导致无法使用该仓库名作为其他用途。
- 需要配置各种仓库的权限，需要配置 GitHub Token 等参数。
- GitHub Actions 工作流比较复杂，涉及到调用 Git 的 `config`、`commit`、`push --force` 等操作，需要根据自己的实际仓库配置路径等。

事实上，GitHub Pages 不仅可以使用独立的仓库托管，也可以直接从 GitHub Actions 的构建产物（Artifact）中部署，无需各种鉴权参数。在本文所载的方法中，GitHub Actions 将源代码构建为 Artifact，再直接将 Artifact 部署到 GitHub Pages 中，从而不需要另外创建仓库。

## 逐步教程

### 1. 环境准备

本文假设您刚刚使用 `hexo init` 创建了一个 Hexo 本地项目：

{% codeblock shell %}
hexo init ExampleBlog
cd ExampleBlog
{% endcodeblock %}

并初始化了一个 Git 仓库：

{% codeblock shell %}
git init
{% endcodeblock %}

### 2. 创建 GitHub Actions 配置文件

在 Hexo 项目的 `.github/workflows` 目录下创建一个名为 `deploy.yml` 的文件，内容如下：

```yaml .github/workflows/deploy.yml
name: Deploy Hexo to GitHub Pages

on:
  push:
    branches:
      - main # 仅在 main 分支 push 时触发

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '22.14.0'

    - name: Install dependencies
      run: npm install

    - name: Generate static files
      run: npm run build

    - name: Upload static files as artifact
      id: deployment
      uses: actions/upload-pages-artifact@v3
      with:
        path: public # 将 public 目录作为 Artifact 上传，这是 Hexo 默认的构建目录

  deploy:
    needs: build
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

有关以上工作流的详细说明，请参阅 GitHub 官方指南：[创建自定义 GitHub Actions 工作流以发布站点](https://docs.github.com/zh/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#%E5%88%9B%E5%BB%BA%E8%87%AA%E5%AE%9A%E4%B9%89-github-actions-%E5%B7%A5%E4%BD%9C%E6%B5%81%E4%BB%A5%E5%8F%91%E5%B8%83%E7%AB%99%E7%82%B9)

### 3. 配置 GitHub Pages

1. 在 GitHub 网页，进入仓库的 `Settings` 页面。
   ![一张显示仓库头部标签页的截图，"Settings"（设置）标签页被红色边框高亮突出显示。](repo-actions-settings.png)
2. 在左侧导航栏的 `Code and automation` 部分中，点击 `Pages`。
3. 在 `Build and deployment` 部分的 `Source` 选择框中，选择 `GitHub Actions`。
   {% image build-and-deployment-source.png Build and deployment 设置中，将 Source 选项设置为 GitHub Actions width:250px bg:'#0d1117' %}
4. （可选）在 `Custom domain` 中设置自定义域名，详见 GitHub 官方文档：[配置 GitHub Pages 站点的自定义域](https://docs.github.com/zh/pages/configuring-a-custom-domain-for-your-github-pages-site)。

### 4. 提交并推送代码

```shell
git add .
git commit -m "First commit, with GitHub Actions"
git push origin main
```

### 5. 查看部署结果

在 GitHub 仓库的 `Actions` 页面查看工作流的执行情况，如果一切正常，您的 Hexo 博客应该已经部署到 GitHub Pages 上了。如果您之前没有为 GitHub Pages 配置自定义域名，您可以回到 `Settings` 页面的 `Pages` 部分查看和修改静态页面部署的网址。

{% image github-actions-deploy-result.png 在您的 GitHub 仓库的 Actions 页面中，可以查看工作流的运行结果 %}

## 优化部署速度

通过将 `npm` 换用为 `yarn`，并启用依赖缓存，可以大幅加快博客的构建速度，本博客实测的构建速度可从51秒缩短到26秒甚至更短。

注意，从 `npm` 迁移到 `yarn` 也需要在本地仓库进行一些更改，请参阅 [Yarn 文档](https://yarnpkg.com/getting-started) 了解如何使用 Yarn。

```yaml .github/workflows/deploy.yml
name: Deploy Hexo to GitHub Pages

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '22.14.0'
        cache: 'yarn'

    - name: Install dependencies
      run: yarn install --frozen-lockfile

    - name: Generate static files
      run: yarn run build

    - name: Upload static files as artifact
      id: deployment
      uses: actions/upload-pages-artifact@v3
      with:
        path: public

  deploy:
    needs: build
    permissions:
      pages: write
      id-token: write
    environment: 
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
