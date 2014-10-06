---
layout: post
title: "开始用jekyll"
category: tools
---

从[jekyll-bootstrap](https://github.com/plusjade/jekyll-bootstrap.git) fork出来，按照[jekyll-quick-start](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)里的步骤来做，就成了现在这个样子。

看了一晚上关于jekyll的东西，有这些文章：

* [Jekyll QuickStart](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)
* [jekyll introduction](http://jekyllbootstrap.com/lessons/jekyll-introduction.html)
* [一篇介绍在github上建立主页的文章](http://www.worldhello.net/gotgithub/03-project-hosting/050-homepage.html)
* [using jekyll with pages](https://help.github.com/articles/using-jekyll-with-pages/)

现在遇到最大的问题是：

1. 生成的页面而已和样式不是自己想要的，不知道怎么修改。
2. rake post title="TITLE"，生成_post下的文件名不支持中文。

不管怎么样，先从这里开始吧。接下来把之前写的一些笔记迁移过来。然后再把页面慢慢修改成自己想要的风格。

## 常用命令

* rake post title="TITLE"

* rake page name="NAME"

* jekyll serve --watch --detach

* jekyll build

## 介绍Liquid的链接

- http://jekyllrb.com/docs/templates/
- https://github.com/Shopify/liquid/wiki/Liquid-for-Designers

