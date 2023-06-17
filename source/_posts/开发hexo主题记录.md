---
title: 开发hexo主题记录
date: 2023-04-13 10:13:55
tags: hexo
categories: 工具
---

# 关于 Hexo 及其工作原理

## 什么是 Hexo

Hexo.js 官网的概述是：快速、简洁且高效的博客框架。准确而具体地概述应该是 Hexo.js 是一个基于 Node.js ，可以将 markdown 文件转换为 html 静态界面的博客框架。

换句话说，hexo.js 可以使用 hexo serve 的命令运行提供网站功能，但其最主要的作用应当是生成静态文件，然后交给 nginx / tomcat 等服务器软件进行维护，对外提供网页服务。

## Hexo 工作原理

应该把 Hexo.js 看作一个工具，一个根据配置文件与 markdown 文件以及 html/css/js 代码转换为前端界面的工具。
默认情况下 Hexo.js 运行后，将会默认生成多个页面，并具有相应的路径。
因此，Hexo 主题的作用就是将每个页面进行特定样式的渲染。

## Hexo 快速开始

按照教程进行初始化项目：
hexo init 'xxxx'
cd 'xxxx'
npm install
hexo serve
访问 4000 端口

## 总体分析默认的 landscape 主题

languages 国际化
即对于同一个变量对应的语言表达方法，比如 英文 home 简体中文 主页。

layout 主题布局文件
在启动 hexo 的时候，博客渲染的入口为其中的 layout.ejs 文件，其他的内容将对应其他 ejs 文件，比如分类对应的是 category.ejs ，标签对应的是 tag.ejs。

这里详细展开 layout 中所有的文件：

\_partial: 将整个网页拆成若干个局部模块，这些子模块存储于这个文件夹。
\_partial/post：查看博客时的所有组件；
\_partial/post/category.ejs: 查看博客详情时显示博客的类别；
\_partial/post/date.ejs: 查看博客详情时显示博客的日期；
\_partial/post/gallery.ejs: 查看博客详情时显示博客的图片；
\_partial/post/nav.ejs: 查看博客详情时显示博客的上一篇下一篇的导航；
\_partial/post/tag.ejs: 查看博客详情时显示博客的标签；
\_partial/post/title.ejs: 查看博客详情时显示博客的标题；
\_partial/after-footer.ejs：所有页面 html 的最后面应用 js 部分；
\_partial/archive-post.ejs：对所有博客的归档；
\_partital/archive.ejs：包括分类，标签以及博客的归档；
\_partial/article.ejs：展示每篇博客的内容；
\_partial/footer.ejs：每页内容的最下面展示的内容，比如 copyright 等；
\_partial/gauges_analytics：对每篇博客的字数统计分析；
\_partial/google_analytics：对每篇博客的谷歌统计分析；
\_partial/head.ejs：html 的头部内容；
\_partial/mobile-nav.ejs：移动端时的导航；
\_partial/sidebar.ejs：侧边栏；
\_widget：小工具类，也就是侧边导航栏的组件；
\_widget/archive.ejs：侧边栏的归档；
\_widget/category.ejs：侧边栏的分类；
\_widget/recent_posts.ejs：侧边栏的最近博客；
\_widget/tag.ejs：侧边栏的标签；
\_widget/tagcloud.ejs：侧边栏的标签云（使用自带的函数）；
archive.ejs ：博客的归档，是直接绑定 archives/ 的入口，根据归档的不同会在这里进行分岔。比如博客的归档，分类的归档，标签的归档。
category.ejs：博客的分类，是直接绑定 categories/某类名称 的入口，根据某类名 的不同进行渲染。
index.ejs：直接绑定博客的主页，在访问根目录时，对应的 <%- body %> 输出的内容。
layout.ejs：整个主题的入口，包括 html 的 <head> 标签等所有内容。一般而言不同页面渲染结果不同是因为 <%- body %> 输出的内容不同。<%-body %> 是自带的内容，会根据访问的是主页还是分类或者标签或者归档进行渲染。
page.ejs：特殊页的渲染，除了分类、标签、主页和归档，用户可以自己定义页面，自己定义的页面对应的 markdown 文件的渲染方法与 page.ejs 对应。
post.ejs：博客的详细内容渲染，也就是对应 markdown 文件转换出的 html 界面。
tag.ejs：标签的分类，直接绑定某个标签的内容，比如访问 tags/某标签 则返回这个标签的所有内容，就是在这里进行控制的。

scripts 主题自带脚本文件夹
接下来查看 scripts 文件夹：

fancybox.js：在启动 hexo 时候会运行这个脚本。
1.4.4 source 主题自带的资源文件夹
主题的渲染过程样式非常重要，主题对应的 css 样式存储于这个文件夹，以及用得到的 js 脚本，也存储于此（与 scripts 不同，那个会启动时自动执行，只执行一次）。

css ：所有的样式文件夹。
fancybox：fancybox 对应的样式与 js 文件；
js：所有的 js 文件。
1.4.5 \_config.yml 主题配置文件
编写主题的时候必须考虑到不同的人审美、需求是不一样的，为了让用户用起来简单，把尽可能可以通过配置文件配置的内容均放在这个文件中比较合适。比如，主题默认情况下侧边栏的顺序是：
widgets:

- category
- tag
- archive

## hexo 主题工作整体流程

Hexo 启动后，

1. 读 scripts 下所有脚本并执行这些脚本；
2. 读取 layout 目录下 layout.ejs 文件；
3. 根据 <%-body%> 在 layout.ejs 的位置进行渲染，包括主页的渲染、分类的渲染、归档的渲染以及自定义页面的渲染。

# Hexo 主题编写

## 新建主题并配置

在 themes 目录下新建一个文件夹，我们的主题对应的就是这个文件夹，给自己的主题起个名字，我们暂时起名为 wnxx 吧。
目前 wnxx 还是为空的文件夹，我们修改根目录的 \_config.yml 也就是 Hexo 的配置文件，将主题修改为我们的主题名字 wnxx。
![](主题.png)
并且在 wnxx 目录下新建 layout 文件夹，并在 layout 下新建 layout.ejs 文件与 index.ejs 文件，前面第一章节介绍过，layout.ejs 是整个主题的入口，而 index.ejs 是必需文件，暂时为空白即可。
编辑 layout.ejs 内容如下：

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>wnxx</title>
</head>
<body>
    这是入口
</body>
</html>
```

重启 hexo ，可以看到效果为：
![](入口.png)

## 新建所有功能性文件

先理清楚所有功能对应的功能性文件，概述一下如表格所示。
意博客详情页时默认是 posts/年/月/日/博客标题，也可以自行修改，这里统称为 博客路径。
我们根据这个表格新建相应的文件。

## 熟悉 hexo 内置的变量

官网文档地址为：中文 https://hexo.io/zh-cn/docs/variables | 英文 https://hexo.io/docs/variables 建议两个都看一下。

## layout.ejs

layout.ejs 是主题的入口，因此整个博客网站不同页面都可以认为是由 layout.ejs “派生” 得到的。换句话说，layout.ejs 中 <%- body %> 具体输出什么是由用户访问的页面决定的，其他部分可以根据具体访问的页面而决定是否展示（比如访问 archive 可以考虑隐藏侧边栏等）。

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>wnxx</title>
</head>

<body>
    <!-- 网站名 -->
    <h1>
        <a href="/"><%= config.title  %></a>
        <a href="/">Home</a>
        <a href="<%-url_for(config.archive_dir)%>">Archive</a>
    </h1>
    <%- body %>

    <h3>categories</h3>
    <ul>
        <% site.categories.each(category=>{ %>
            <li>
                <a href="<%- url_for(category.path) %>">
                    <%- category.name  %>
                </a>
            </li>
        <% }) %>
    </ul>

    <h3>tags</h3>
    <ul>
        <% site.tags.each(tag=>{ %>
            <li>
                <a href="<%- url_for(tag.path) %>">
                    <%- tag.name  %>
                </a>
            </li>
        <% }) %>
    </ul>
</body>
</html>
```

## index.ejs

主页的内容展示带有分页效果的博客，具体的方法与前面的类似，读取 page 变量的所有 posts 然后遍历，渲染。

```
<!-- 主页 -->
<ul>
<% page.posts.each(post=>{ %>
    <li>
        <a href="<%=url_for(post.path)%>">
            <%= post.title %>
        </a>
        | <%- date( post.date , 'YYYY/MM/DD HH:mm') %>
    </li>
<% }) %>
</ul>

<%- paginator({
    prev_text: '<',
    next_text: '>'
}) %>
```
