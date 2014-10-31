title: 建立自己的github 博客
date: 2014-10-31 16:11:23
tags:
---
前言：

> 对于写博客的好处这里不太多说，我个人的初衷是总结技术，研究技术，分享技术。推荐一篇总结如何写博客好的文章 http://rock3.info/blog/2013/11/26/如何写一篇好的技术博客

之前也尝试过自己购买空间和域名建站的方案，一方面要花钱，价格低得访问速度不行，价格高的也没必要。无意间看到 github page，当时的感觉就是，这才是我想要的！
### 下面简单介绍下github page：

- github page 使用的是生成的是静态网页，不能使用php，数据库，添加动态功能需要使用外部服务，比如添加评论等
- github page 使用免费，流量免费，能使用git 的版本管理功能
- github page 基于Jekyll引擎，它是一种简单的、适用于博客的、静态网站生成引擎
- github page 可以自定义主题，自定义域名，添加各种功能，适合简历个人博客。



### 下面就开始我们的建站了本文以mac系统下为例！

- 首先，在github 上 Create a repository ，具体参考：https://pages.github.com/, 创建后就有了blog 的容器，现在该添加内容了。
- 为了能让电脑与github利用git通信，需要安装github客户端https://mac.github.com/
- 安装博客生成系统，本文以 hexo为主，它的优点是快速，效率高，易于使用。安装过程如下http://hexo.io/docs/index.html
- 安装完hexo后，需要初始化，先把之前在github上面创建的 repository,clone到一个目录，然后在终端里面
hexoinit<那个目录地址> cd <那个目录地址>
$ npm install
做完这些，hexo 开发环境就安装好了
- 下面是安装自己喜欢的主题，[主题列表](https://github.com/tommy351/hexo/wiki/Themes)，最简单的安装主题方法是，直接下载主题包，然后复制到你的<那个目录地址>/themes下
- 配置博客信息，也就是编辑_config.yml文件，[具体参数](http://hexo.io/docs/configuration.html)，对于配置，有点需要特别注意的就是下面的字段
<pre><code>//# Deployment
deploy:
type: github
repository: git@github.com:<你的用户名>/<你的用户名>.github.io.git
branch: master </pre></code>
- 配置了简单信息后，就需要把自定义的博客，生成在page上面，打开终端 cd 到<那个目录地址>下
hexoclean(清除一些缓存数据) hexo generate 生成用于部署的博客
`$ hexo deploy (部署到page 上面)`
完成了上面的工作，就能查看自己的博客了，首次部署可能会有十分钟左右的延迟


### 给博客添加插件

评论系统，目前流行的评论系统是 `disqus`国际上流行 和 `多说` 国内流行，本博客选择前者，由于做的是技术博客，hexo内置支持的就是disqus，配置起来很方便，只需要在[这里](http://disqus.com/)注册过，然后在_config.yml里，有个d`isqus_shortname` 字段写上之前注册过的shortname即可。然后重新部署下，就会看到每篇博客下面都自动加上评论部分了。
另外的一些有用的信息

[常用的hexo命令](http://hexo.io/docs/commands.html)

未完待续…
