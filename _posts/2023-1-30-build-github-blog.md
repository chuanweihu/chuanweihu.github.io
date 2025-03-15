---
layout: post
title:  搭建github博客
categories: [study]
comments: true
tags: [blog]
---

> 搭建Github博客的过程比较简单，跟着Github找到的仓库一步一步的做就可以了，关键在于通过readme文件的关键字来查找相关的github工程。

* TOC
{:toc}

<!--more-->

## 搭建github博客

### github搜索

在github搜索框中输入`blog easily start in:readme stars:>5000`

![search](/img/posts/202301/github-search.png)

### 查看仓库

查看相关仓库的内容是否符合自身所需

![dig](/img/posts/202301/dig_repos.png)

### 根据步骤设置

1. Fork该仓库并修改仓库名称为`yourgithubusername.github.io`

![step1](/img/posts/202301/jekyll-now-step1.png)

在Fork的仓库中找到`Settings`，然后修改仓库名称，例如用户名称为`huchuanwei1018`

![step1.1](/img/posts/202301/jekyll-now-step1.1.png)

2. 定制网站

通过修改配置文件`_config.yml`来生成定制的网站。

![step2](/img/posts/202301/jekyll-now-step2.png)

3. 推送博客

将博客文章名为`year-month-day-title.md`格式，在文章中使用`front-matter`配置，然后将其放在`/_posts`文件夹下就可以查看了。

![step3](/img/posts/202301/jekyll-now-step3.png)

## 推送自动工作流

通过构建`Github action`实现推送自动发布博客功能。Github Action 本质就是 Github 推出的持续集成工具, 每次提交代码到 Github 的仓库后，Github 都会自动创建一个虚拟机（例如 Mac / Windows / Linux），来执行一段或多段指令。

## giscus评论系统

经过比较，最终使用`giscus评论系统`，它使用`GitHub Discussions`作为存储和管理评论的后端。网站的访客可以通过GitHub账号登录并发表评论。

## 更新日志

- date: 2023-11-18
  - desc: 增加推送action自动工作流

- date: 2024-02-22
  - desc: 添加目录
  - desc: 添加更新日志，推荐阅读和文档

- date: 2024-02-23
  - desc: 增加giscus评论系统

## 推荐阅读

- [搭建github博客](https://www.huchuanwei.com/articles/2023-01/build-github-blog)

## Additional Resources

### Documentation

1. [Front Matter](https://jekyllrb.com/docs/front-matter/)
2. [Jekyll部署方法](https://jekyllcn.com/docs/deployment-methods/)
3. [GitHub Actions 快速入门](https://docs.github.com/zh/actions/quickstart)

### Useful Websites

1. [YAML](https://en.wikipedia.org/wiki/YAML)
2. [打造Github Issue到Hexo部署自动工作流](https://cloud.tencent.com/developer/article/1992687)
3. [5 分钟教你快速掌握 GitHub Actions 自动部署博客](https://www.cnblogs.com/enoy/p/16197448.html)
4. [基于 giscus 为网站添加评论系统](https://fengchao.pro/blog/comment-system-with-giscus/)