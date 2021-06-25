---
titile: 基于jekyll和Github Pages搭建个人博客
tags: Git&Github
---

# 基于jekyll和Github Pages搭建个人博客

>参考链接：
>
>- [https://wu-kan.cn/2019/01/18/基于Jekyll搭建个人博客/](https://wu-kan.cn/2019/01/18/基于Jekyll搭建个人博客/)
>- [jekyll官网](https://jekyllcn.com/docs/structure/)
>- [Jekyll + Github Pages 博客搭建入门](https://www.jianshu.com/p/9f198d5779e6)
>- [https://github.com/wu-kan/wu-kan.github.io](https://github.com/wu-kan/wu-kan.github.io)

# 两个概念

## jekyll

​	Jekyll 是一个简单的博客形态的静态站点生产机器。它有一个模版目录，其中包含原始文本格式的文档，通过一个转换器（如 [Markdown](https://link.jianshu.com/?t=http%3A%2F%2Fdaringfireball.net%2Fprojects%2Fmarkdown%2F)）和我们的 [Liquid](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2FShopify%2Fliquid%2Fwiki) 渲染器转化成一个完整的可发布的静态网站，你可以发布在任何你喜爱的服务器上。Jekyll 也可以运行在 [GitHub Page](https://link.jianshu.com/?t=http%3A%2F%2Fpages.github.com%2F) 上，也就是说，你可以使用 GitHub 的服务来搭建你的项目页面、博客或者网站，而且是**完全免费**的。

​	简单来说，Jekyll就是将纯文本转化为静态博客网站，不需要数据库支持，也没有评论功能，想要评论功能的话可以借助第三方的评论服务。

## GitHub Pages
​	GitHub Pages是一个静态网站托管服务，直接从github仓库托管你个人、公司或者项目页面 ，并且不需要你写任何后端语言来支持。

Github Pages的服务是免费的，但是也有一些限制：

- 仓库空间不大于1G

- 每个月的流量不超过100G

- 每小时更新不超过 10 次

# 文章目的

​	这篇文章的目的不是为了介绍原理或是底层机理，这些网上已经有很多了，如果有不懂的地方，先看文章**开头的参考链接**。我主要为了描述搭建的过程和写清楚搭建的步骤，争取让给你小白看了都可以上手，所以尽量会以步骤为主～。

​	我在搭建[我的博客](Jason-QianHao.github.io)的时候，使用的是这位[大神的模板]([https://github.com/wu-kan/wu-kan.github.io](https://github.com/wu-kan/wu-kan.github.io))，github上也有很多其他的，这里有人专门整理了[传送门](https://github.com/cnfeat/GoodThingList/blob/master/GoodJekyllBlogList.md)。使用模板，既非常好看，也可以站在巨人的肩膀上，专心博客的编写就行了。

​	下面就开始了，一步一步来就行了。


# jekyll的安装

## 安装环境

- [Ruby](http://www.ruby-lang.org/en/downloads/)（including development headers, Jekyll 2 需要 v1.9.3 及以上版本，Jekyll 3 需要 v2 及以上版本）
- [RubyGems](http://rubygems.org/pages/download)
- Linux, Un ix, or Mac OS X
- [NodeJS](http://nodejs.org/), 或其他 JavaScript 运行环境（Jekyll 2 或更早版本需要 CoffeeScript 支持）。具体安装可以参考我的这篇博客[Mac下常见的服务和软件安装方式汇总](https://jason-qianhao.github.io/_posts/2021-06-22-Mac下常见的服务和软件安装方式汇总/)，找到NodeJs即可。
- [Python 2.7](https://www.python.org/downloads/)（Jekyll 2 或更早版本）

我是MacOS，这里自己安装了NodeJS，其他都没有动，应该都是自带的。

## 安装步骤

1. 打开终端输入命令`gem install jekyll`，安装Jekyll。

2. 输入命令`gem install bundler`，安装bundler，好像是包管理器什么的，先不管他，装了再说。

3. 去github上下载别人的博客模板，如果觉得我的还不错，也可以去主页下载 [传送](https://github.com/Jason-QianHao/Jason-QianHao.github.io)

   这里为什么要下载，而不是fork呢？只是我的个人测试习惯吧。Github Pages是基于jekyll的，但是缺点是，你写完博客没办法即时预览，所以这里安装Jekyll并不是因为博客需要，我觉得只是本地测试方便而已。本地测试过程中会产生`_site`目录，里面就是渲染出来的博客页面，但是这个其实不用推到github上。如果代码在本地测试完成后，就可以复制需要的文件夹（比如`_site`文件夹就不需要复制)到你博客对应的本地仓库，上传到github上。

4. 终端`cd`进入博客目录，执行`bundle install`，这里就会下载我博客用到的所有依赖（这也是为什么要下载代码的重要性）。如果`bundle install`在你的机器上还是报错，要看报错信息，缺啥装啥。

5. 启动Jekyll，输入命令`bundle exec jekyll s`。这里有很多博客，包裹官网上说用到`jekyll serve`命令，但是我就会一致报错，没办法启动，i do not know !!! 然后，顺利的话你就看到了下面的页面，就成功了。输入它提示的网址`http://127.0.0.1:4000/`就可以预览博客了。

   <img src="/../assets/Jekyll/image-20210625152439391.png" alt="image-20210625152439391" style="zoom:50%;" />

   <img src="/../assets/Jekyll/image-20210625152258072.png" alt="image-20210625152258072" style="zoom:25%;" />

# Github Pages的设置

​	本地可以预览博客后，剩下就是把项目推到github上托管代码，并生成自己的博客了。其实就是创建一个项目而已，项目的名字特殊点了。

1. 新建github仓库，并名字设置为`你的github用户名.github.io`。

   <img src="/../assets/Jekyll/image-20210625151538161.png" alt="image-20210625151538161" style="zoom:50%;" />

2. 然后检查你的pages配置，是不是主分支

   <img src="/../assets/Jekyll/image-20210625151724426.png" alt="image-20210625151724426" style="zoom:30%;" />

3. clone你的github仓库到本地，将你jekyll测试中的博客复制到本地文件夹，修改好后push到github上就完成了。

4. 访问`你的github用户名.github.io`就能看到你的博客

   <img src="/../assets/Jekyll/image-20210625152559749.png" alt="image-20210625152559749" style="zoom:25%;" />

# 怎么设置/编写博客

- 整个博客搭建好了后，如果你想修改博客的样式，主要看`_config.yml`文件，可以修改一些样式

- 添加博客就是在`_posts`文件夹中，但是注意文件的格式`时间+名称`

- 每篇博客里面，开头有个样式是设置标题和标签的，可以给一个博客打多个标签，用空格隔开就行，如下：

```
---
title: blogname
tags: tagname1 tagname2
---
```

# 踩坑

1. **关于图片和文件的url**

   我的博客一般会在电脑上用Typora上通过markdown编辑好，这里你的图片和文件的目录开头是不需要有`/`，直接就是linux的相对路径。**但是**，在Github pages的博客中，要在所有路径前面加上`/`！！！

2. 关于换行

   Typora可能会保留你在编辑时的换行，但是markdown的换行语法时段落后面有`至少2个空格`，记得检查一下。
