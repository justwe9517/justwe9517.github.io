---
layout: post
title:  "使用 Travis Ci + Jekyll + Github Pages 搭建个人博客"
date: 2020-03-22 15:24
categories: Jekyll
tags: ['Jekyll']
collection: Jekyll
---

# 使用 Travis Ci + Jekyll + Github Pages 搭建个人博客

一直想找一个比较好的博客平台来写博客，在尝试众多平台之后最终选择使用 Github Pages 来自己搭建一个博客。

Github Pages 的整个搭建过程其实并不复杂，但无奈网上的文章坑太多，让我在搭建的过程中走了不少弯路，于是计划新写一篇文章来好好的总结一下整个搭建过程。

太长不看版：

> 直接 fork 我这个[博客项目](https://github.com/justwe9517/justwe9517.github.io)，然后参考`集成 Travis CI 平台`那一节完成 Travis CI 的配置，之后直接在 write 分支中写文章即可。

下面进入正文：

## 什么是 Github Pages

> GitHub Pages 是一项静态站点托管服务，它直接从 GitHub 上的仓库获取 HTML、CSS 和 JavaScript 文件，（可选）通过构建过程运行文件，然后发布网站。

构建过程运行文件就是 Github Pages 默认集成的 Jekyll 服务，通过这个服务你可以很轻松的发布自己的 markdown 文本并交给 Jekyll 来解析，自动生成静态文件。

Github Pages 提供了三种类型的仓库，分别为项目、用户和组织。

> 你可在此文档中查看 Github Pages 支持的具体类型：[https://help.github.com/cn/github/working-with-github-pages/about-github-pages#](https://help.github.com/cn/github/working-with-github-pages/about-github-pages#)

后续文档我会以 `用户` 类型站点为模板进行讲解。

## 什么是 Travis CI

简单点理解 Travis Ci 就是一个自动化操作平台（实际上功能非常强大），你只需要知道它能代替你执行所有操作，你只需要关注如何写好文章，其他的事情都交给 Travis CI 来就好了。

## 为什么使用 Travis Ci 进行构建

虽然 Github Pages 内置了 Jekyll 服务，但却不支持使用自定义插件，这就限制了 Github Pages 的使用场景。

现在咱们来看看，如果不使用 Github Pages 内置的 Jekyll 服务构建，通过手动的方式发布文章，需要进行哪几步骤操作：

1. 安装 Ruby 环境。
2. 安装 Git 服务。
3. 安装 Jekyll 服务。
4. 克隆项目。
5. 在 _post 目录中编辑文章。
6. 通过命令生成 _site 目录。
7. 使用 git 推送更改。

再来看看，如果使用 Travis-Ci 发布文章需要哪几个步骤：

1. 安装 Git 服务，克隆项目，在 _post 目录中编辑文章，使用 git 推送更改。

好了，经过我上面的简单介绍，我想大家也都清楚 Travis-Ci 是个多么强大的生产力工具。你甚至可以直接在 github.com 网站上直接编写文章。

## 搭建环境

搭建一个 Github Pages 平台需要具备如下条件：

1. 拥有一个 Github 账号，没有的去这里 [https://github.com/](https://github.com/) 注册一个。
2. 会熟练使用 Git 命令，不会的需要在 [https://git-scm.com/book/zh/v2](https://git-scm.com/book/zh/v2) 简单学习一下。
3. 知道如何创建 Jekyll 项目，不会的需要在 [https://jekyllrb.com/docs/installation/](https://jekyllrb.com/docs/installation/) 学习一下如何创建项目。

在继续浏览后续文章之前，请务掌握上面三点，因为我不会在介绍如何使用 git 命令，以及如何在不同平台下创建 Jekyll 项目了。不过，你可以直接 fork 我的项目，这样你就不需要在掌握第三条了 :-)。

### 创建仓库

首先进入 Github 点击右上角的加号，如下图所示：

![c432a25a298f467960517f517ff72c9](/assets/uploads/c432a25a298f467960517f517ff72c9.png)

进入 `Create a new repository` 页面后按照图中的要求操作：

![08a4a7ce65eb13958c7f2e4e10a64cd](/assets/uploads/08a4a7ce65eb13958c7f2e4e10a64cd.png)

点击 `Create repository` 按钮即可完成项目创建。

项目创建完成之后先来对项目进行一下简单的配置，首选你需要先创建一个 write 分支，并将其设置为默认分支，操作步骤如下：

1. 在项目的 Code 选项卡中，找到左下角的 `Branch: master` 下拉框：
    ![e1575eca8e5984ae29b721b4c60db24](/assets/uploads/e1575eca8e5984ae29b721b4c60db24.png)
2. 点击下拉框并在输入框内输入 `write` 名称，如果 `write` 分支不存在，下方的文字应该为 `Create branch: write from 'master'`，点击它创建分支，创建成功后上图的 `1 branch` 会变成 `2 branch`。
    ![48f12cf7589d4548e783f5201dbed62](/assets/uploads/48f12cf7589d4548e783f5201dbed62.png)
3. 分支创建成功后，只需要在将其设置为默认分支即可。
   步骤：你需要在选项卡中点击 Settings 按钮进入项目设置页面，并在左侧的导航中找到 `Branches` 菜单，点击菜单后，右侧页面会出现 `Default branch` 文字，你在下来菜单中选中 `write` 分支，并点击旁边的 `update` 按钮即可。

完成上面的三步之后，你就可以导入代码了。

你需要先添加一些内容进去，可以直接去 `https://github.com/justwe9517/justwe9517.github.io` 下载我的模板，也可以导入其他 Jekyll 项目的模板。

### 配置项目

下图是一个标准的 Jekyll 项目目录结构。

```
_includes
_layouts
_posts
_sass
assets
script
.gitignore
404.html
README.md
_config.yml
index.md
```

我们需要在这个目录中加几个文件进去，告诉 Travis CI 如何处理这个项目。

如果你是使用我的网站源码，那你可以直接跳过这部分！

对于没使用我的站点源码的朋友，我现在仔细说说需要添加哪些文件进去。

#### Gemfile 文件

首先，你需要在项目的跟目录中添加名为 `Gemfile` 的配置文件，并将下面的内容复制进去，此文件的作用是处理项目依赖。

你可以在这里找到下面代码的作用：[Jekyll Deployment Travis CI](https://jekyllrb.com/docs/continuous-integration/travis-ci/)

```sh
source "https://rubygems.org"

gem "jekyll", "~> 3.8.5"
gem "minima", "~> 2.0"
group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.6"
end
gem "tzinfo-data", platforms: [:mingw, :mswin, :x64_mingw, :jruby]
gem "wdm", "~> 0.1.0" if Gem.win_platform?
gem "rake"
gem "html-proofer"
gem "jekyll-theme-console"
gem "jekyll-paginate-v2"
# 自定义插件都要写到这里
```

#### .travis.yml 文件

之后，你需要在项目的跟目录中添加名为 `.travis.yml` 的配置文件，继续将下面的内容复制进去，此文件的作用是告诉 Travis CI 如何处理此项目。

你可以在这里找到下面代码的作用：[Jekyll Deployment Travis CI](https://jekyllrb.com/docs/continuous-integration/travis-ci/)，[Travis CI GitHub Pages Deployment](https://docs.travis-ci.com/user/deployment-v2/providers/pages/)

Jekyll Deployment Travis CI 文章会告诉你如何在 Travis CI 中生成 _site 目录。

Travis CI GitHub Pages Deployment 文章会告诉你如何部署到 Github Pages 中。

```yml
language: ruby
rvm:
    - 2.6.3
before_script:
    - chmod +x ./script/cibuild
script: ./script/cibuild
branches:
    only:
        - write
        - master
env:
    global:
        - NOKOGIRI_USE_SYSTEM_LIBRARIES=true
addons:
    apt:
        packages:
            - libcurl4-openssl-dev
sudo: false
cache: bundler
notifications:
    email: false

deploy:
    provider: pages:git
    token: $PAGES_TOKEN
    edge: true # opt in to dpl v2
    target_branch: master
    local_dir: './_site'
    on:
        branch: write
```

#### script/cibuild 文件

还差最后一个文件，你需要在项目的跟目录中创建一个名为 `script` 的目录，并在这个目录中在创建一个 `cibuild` 文件，并将下面的内容复制进去，此文件是 Travis CI 的一部分。

你可以在这里找到下面代码的作用：[Jekyll Deployment Travis CI](https://jekyllrb.com/docs/continuous-integration/travis-ci/)

```sh
#!/usr/bin/env bash
set -e # halt script on error

bundle exec jekyll build
bundle exec htmlproofer ./_site
```

完成上面几步之后，你的目录看起来应该是下面这个样子：

```sh
_includes
_layouts
_posts
_sass
assets
script
.gitignore
404.html
README.md
_config.yml
index.md
Gemfile           # 新添加的文件
.travis-ci.yml    # 新添加的文件
script/cibuild    # 新添加的文件
```

如果确认无误，那这个项目就配置好了，接下来只需要简单的配置一下 Travis CI，你就可以享受由 Github Pages 带来的爽快感。

### 集成 Travis CI 平台

在浏览器中 `https://travis-ci.com/` 进入 Travis CI 网站，注意不要进入到 `https://travis-ci.org/` 网站，前者是收费版，但如果是开源项目可以免费享受到构建。

在这个网站中你可以点击右上角的 `Sign in` 按钮，使用 `SIGN IN WITH GITHUB` 进行登录。

完成登录后，你可以重新访问一次 `https://travis-ci.com/`，这时页面看起来应该是这个样子：

![02260587c5931746217a78efc3b271e](/assets/uploads/02260587c5931746217a78efc3b271e.png)

你只需要点击 `ACTIVATE ALL REPOSITORIES USING GITHUB APPS` 按钮，并按照他们的要求完成操作即可。

至此，你的 Travis CI 就配置完了。

-----

参考的文章：

- [https://docs.travis-ci.com/user/deployment-v2/providers/pages/](https://docs.travis-ci.com/user/deployment-v2/providers/pages/)
- [https://jekyllrb.com/docs/continuous-integration/travis-ci/](https://jekyllrb.com/docs/continuous-integration/travis-ci/)
- [https://travis-ci.community/t/failed-to-deploy-simple-github-pages/3335/4](https://travis-ci.community/t/failed-to-deploy-simple-github-pages/3335/4)
