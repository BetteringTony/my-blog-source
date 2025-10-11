---
title: Blog日常操作
date: 2025-10-11 20:14:36
categories: [其他]
tags: [备忘]
---

该博客页面由`hexo`配合 Github 搭建而成，该文章记录常用命令以及日常使用流程，主要是方便自己查阅。

**1、常用命令：**

- 生成静态文件

  `hexo generate` 或者简写为 `hexo g`，该命令会将博客的源文件（如 Markdown 文章、配置文件等）转换为静态 HTML 文件，生成的文件会放在 `public` 文件夹中。

- 启动本地服务器

  `hexo server` 或者简写为 `hexo s`，运行此命令后，Hexo 会启动一个本地服务器，可以通过浏览器访问 `http://localhost:4000` 来预览博客。在开发过程中，它可以实时显示更改，预览没有问题之后再进行后续部署操作。

- 部署到 GitHub Pages

  `hexo deploy` 或者简写为 `hexo d`，此命令会将 `public` 文件夹中的静态文件推送到 GitHub Pages 仓库，来更新博客，等待一会儿之后，就可以访问url地址来看见最新的对外展示博客。

- 新建文章

  `hexo new "Post Title"`，运行此命令后，Hexo 会在 `source/_posts` 文件夹中创建一个新的 Markdown 文件，文件名是根据文章标题生成的。可以在这个文件中编写博客文章内容。

- 清理缓存

  `hexo clean`，该命令用于清理缓存，可以先执行这个之后再预览或部署，确保缓存不会影响最新的博客展示。

**2、日常使用：**

先`hexo new "Post Title"`，生成想要书写的博客文章，写好内容之后，执行`hexo clean`清除可能的缓存，再`hexo g`生成静态文件，`hexo s`启动本地服务器，浏览器中预览确认没有问题之后，再`hexo clean`、`hexo g`、`hexo d`部署到Github仓库对外展示。
