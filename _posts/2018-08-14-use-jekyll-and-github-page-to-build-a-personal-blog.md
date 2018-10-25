---
layout: post
title: '使用 jekyll + github pages 搭建个人博客的记录'
subtitle: '终于搭建起了个人博客，写下第一篇文章记录一下'
date: 2018-08-14
categories: web
cover: 'http://120.77.171.203:8080/images/blog-img/0.jpg'
tags: jekyll github-pages blog
---


这是我的个人博客，是用 `jekyll` + `github pages` 搭建起来的。

然后还使用了 [`jekyll-theme-H2O`](https://github.com/kaeyleo/jekyll-theme-H2O) 这个主题。  
目前就先这样吧，现在是离职状态，主要任务是学习，然后写写博文做记录。以后有空再自己写一个主题，也会慢慢地把自己以前的一些文章迁移过来。

---

下面简单写一下搭建这个博客的过程。

## 1. 新建 `github.io` 项目

其实 [`github pages`](https://pages.github.com/) 有两个用途，大家可以在[官方网页](https://pages.github.com/)看到。其中一个是作为个人/组织的主页（每个账号只能有一个），另一个是作为 `github` 项目的项目主页（每个项目可以有一个）。
而 `github pages` 本身就支持 `jekyll` ，所以二者的结合使用非常方便。

这两种静态页面怎么生成在 [`https://pages.github.com/`](https://pages.github.com/) 这里都有详细步骤。

所以现在既然我们要建的是个人博客，就是第一种用途了。

1. 在 `github` 上新建一个 repo，命名为 `username.github.io`，其中 `username` 就是你的 `github` 用户名。
2. 把该项目 clone 到本地。
3. 进入项目目录，在根目录下创建 `index.html` ，这个会自动作为博客的主页。
4. 把所有的修改通过 `git push` 到刚刚创建的 repo。
5. 好了，现在我们可以通过 `https://username.github.io` 来浏览我们的个人网站了。

## 2. 安装、使用 `jekyll`

网站跑起来了，但是里面没内容。当然我们可以自己写各个页面去丰富她。但是这样比较麻烦，弄个人博客我们主要关注的当然是写博文，而不想每次写一篇文章都要自己去排版一个页面。`jekyll` 就是可以帮助我们实现只要关注写文章本身，她会帮我们自动转化成静态页面。

可以看一下官网的描述
> Jekyll 是一个简单的博客形态的静态站点生产机器。它有一个模版目录，其中包含原始文本格式的文档，通过一个转换器（如 Markdown）和我们的 Liquid 渲染器转化成一个完整的可发布的静态网站，你可以发布在任何你喜爱的服务器上。Jekyll 也可以运行在 GitHub Pages 上，也就是说，你可以使用 GitHub 的服务来搭建你的项目页面、博客或者网站，而且是完全免费的。

支持 `markdown` ，非常方便。

下面我们开始安装 `jekyll` 并且新建一个项目看一下。
```bash
# 安装jekyll
gem install jekyll

# 新建项目
jekyll new myblog # myblog 是项目名

# 进入项目目录
cd myblog
```

创建成功后的项目目录是这样的

![](http://120.77.171.203:8080/images/blog-img/1.jpg)

因为现在最新版(3.8.3)的 `jekyll new` 命令创建的项目默认已经用了主题了（可以在 `_config.yml` 配置文件下看到 `theme: minima` 这一行），因此我们要安装相应的依赖。

```bash
# 安装依赖包
gem install jekyll bundler
bundle install # 这个命令会自动安装项目所需的依赖

# 启动一个本地服务器，而且启动后还会自动监听文件，如果本地修改了某个文件，会重新生成静态页面，我们只需要在浏览器刷新一下就好
jekyll serve
```

启动后应该能看到以下的输出

![](http://120.77.171.203:8080/images/blog-img/2.jpg)

这时候我们就能在本地通过 `http://127.0.0.1:4000/` 来预览效果了。

![](http://120.77.171.203:8080/images/blog-img/3.jpg)

PS：有一些预设的设置可以在 `_config.yml` 配置文件里面修改。

## 3. 怎么写一篇文章

大家可以先看看 [`jekyll` 项目的目录结构](https://www.jekyll.com.cn/docs/structure/)

所以新增文章应该在 `_posts` 目录下操作，而且注意命名要按照 `年-月-日-标题.后缀名` 的格式。

现在我们来新建文章。
```bash
cd _post
touch 2018-08-14-my-first-article.md
```

`2018-08-14-my-first-article.md` 文件输入以下内容
```
---
layout: post # 使用post模版
title:  "我的第一篇文章"
date:   2018-08-14
categories: life
---

这是我的第一篇文章
```

再启动一下 `jekyll serve` 就能看到效果了。

![](http://120.77.171.203:8080/images/blog-img/4.jpg)
![](http://120.77.171.203:8080/images/blog-img/5.jpg)

## 4. 使用 `jekyll` 主题美化网站

内容知道怎么创建了，但是现在网站太简陋了。那怎么办？不用担心，`jekyll` 本身已经提供了挺多主题供我们选择使用。大家浏览 [`Jekyll Themes`](http://jekyllthemes.org/) 这个网页慢慢选吧~~

至于每个主题怎么用最好还是看这个主题项目的说明，具体可能不太一样，我就不细说了。

记得最后修改完在本地预览没问题后要 `git push` 到 `github` 上哦~~

---

参考链接：  
[GitHub Pages](https://pages.github.com/)  
[Jekyll文档](http://jekyllcn.com/docs/home/)  
[快速在 Windows 上搭建 Jekyll 开发环境](https://walterlv.github.io/post/setup-jekyll-in-windows.html)
