title: 建立自己的github 博客
date: 2014-10-31 16:11:23
tags:
---

> 对于写博客的好处这里不太多说，我个人的初衷是总结技术，研究技术，分享技术。推荐一篇总结如何写博客好的文章 http://rock3.info/blog/2013/11/26/如何写一篇好的技术博客

之前也尝试过自己购买空间和域名建站的方案，一方面要花钱，价格低得访问速度不行，价格高的也没必要。无意间看到 github page:

- github page 使用的是生成的是静态网页，不能使用php，数据库，添加动态功能需要使用外部服务，比如添加评论等
- github page 使用免费，流量免费，能使用git 的版本管理功能
- github page 基于Jekyll引擎，它是一种简单的、适用于博客的、静态网站生成引擎
- github page 可以自定义主题，自定义域名，添加各种功能，适合简历个人博客。

---

> 大家知道，github 是国外的网站，访问速度实在不敢恭维，好在国内也有类似 github 这样的代码托管网站 `gitcafe`, 访问速度不错，而且也支持自定义域名，于是乎，就在`gitcafe`上建立了新博客。(由于之前在github上写的博客数不多，索性直接新建了，对于在github上面写博客多的人，建议把博客迁移到`gitcafe`)，点这里查看[迁移过程](blog.devtang.com/blog/2014/06/02/use-gitcafe-to-host-blog)。

### 配置博客环境
#### 1、在[hexo官网上](http://hexo.io)上 安装'hexo'，按照顺序安装
    $ npm install hexo -g
    $ hexo init blog
    $ cd blog
    $ npm install
    $ hexo server
如果提示你`-bash: npm: command not found`，推荐安装`node.js`，它里面包含`npm`包，在[http://nodejs.org](http://nodejs.org) 上面安装`node.js`。然后继续安装上面的步骤安装
#### 2、添加自己喜欢的主题
经过`1`，你的博客就在本地初始化完成了，下面在[主题列表](https://github.com/tommy351/hexo/wiki/Themes)里面选择喜欢的主题，然后`clone`到本地
      
      $ cd ~/blog (你的博客路径，一般为~/blog)
      $ git clone <repository> themes/<theme-name>
     
#### 3、在`gitcafe`上面建立`gitcafe page`（博客项目）
建立`gitcafe page`的过程请查看：[https://gitcafe.com/GitCafe/Help/wiki/Pages-相关帮助#wiki](https://gitcafe.com/GitCafe/Help/wiki/Pages-相关帮助#wiki)

#### 4、设置本地博客的配置文件

##### 4.1、博客的配置文件，路径为~/blog/_config.yml，主要是设置主题，博客名字，同步的平台信息等，常用的如下，点[这里](http://hexo.io/docs/configuration.html)查看全部的配置信息
- `theme`属性，用来设置博客主题，添加你下载的主题名字即可
- `deploy`属性，用于设置同步平台的信息，我的配置信息如下，相应的修改用户名即可
                
         deploy:
           type: git
           message: 
           repo:
           gitcafe: https://gitcafe.com/wangyangyang/wangyangyang.git,gitcafe-pages

##### 4.2、主题的配置文件，路径为~/blog/themes/jacman/_config.yml，`jacman`的配置文件中文说明在[这里](https://github.com/wuchong/jacman/wiki/How-To-Use-Jacman-(中文))
#### 5、发布本地博客到`gitcafe.com`
      $ cd ~/blog
      $ sudo hexo clean //清楚缓存
      $ sudo hexo g     //生成博客的静态html文件
      $ sudo hexo d     //发布到`gitcafe.com`
   
---
### 使用GitHub来管理博客源文件

 请移步:[http://wuchong.me/blog/2014/01/17/use-github-to-manage-hexo-source/](http://wuchong.me/blog/2014/01/17/use-github-to-manage-hexo-source/)  
 
---
### 给博客添加插件

#### 评论系统
目前流行的评论系统是 `disqus`国际上流行 和 `多说` 国内流行，本博客选择前者，由于做的是技术博客，hexo内置支持的就是disqus，配置起来很方便，只需要在[这里](http://disqus.com/)注册过，然后在_config.yml里，有个d`isqus_shortname` 字段写上之前注册过的shortname即可。然后重新部署下，就会看到每篇博客下面都自动加上评论部分了。

---
### 建立过程中可能遇到的错误：

- `Ooooops, page can't be found`，解决方法，需要建立 gitcafe-pages的分支，步骤请查看 [https://gitcafe.com/GitCafe/Help/wiki/Pages-相关帮助#wiki](https://gitcafe.com/GitCafe/Help/wiki/Pages-相关帮助#wiki)

---
### 添加站长统计工具
jacman 主题默认支持`谷歌分析`统计工具，需要在`~/blog/themes/jacman/_config.yml`配置文件里面加入相关配置，我的配置如下(`id`要在[https://www.google.com/analytics/web/?hl=zh-CN](https://www.google.com/analytics/web/?hl=zh-CN)上面申请)
       
      google_analytics:
        enable: true
        id: UA-1766729-8 ## e.g. UA-1766729-8 your google analytics ID.
        site: auto ## e.g. yangjian.me your google analytics site or set the value as auto.
        
填写完了重新发布下博客，然后在[https://www.google.com/analytics/web/?hl=zh-CN](https://www.google.com/analytics/web/?hl=zh-CN)就可以查看浏览量了

---
### 搜索引擎的提交入口
自定义的博客，如果不让各个搜索引擎的记录，那很难被搜索到
在[http://jingyan.baidu.com/article/d5a880eb70424413f147cce4.html](http://jingyan.baidu.com/article/d5a880eb70424413f147cce4.html)里面有各大搜索引擎入口
常用的如下：

- 360搜索引擎登录入口：[http://info.so.360.cn/site_submit.html](http://info.so.360.cn/site_submit.html)
- 百度搜索网站登录口：[http://www.baidu.com/search/url_submit.html](http://www.baidu.com/search/url_submit.html)
- 百度单个网页提交入口：[http://zhanzhang.baidu.com/sitesubmit](http://zhanzhang.baidu.com/sitesubmit)
- Google网站登录口：[https://www.google.com/webmasters/tools/submit-url](https://www.google.com/webmasters/tools/submit-url)
- bing(必应)网页提交登录入口：[http://www.bing.com/toolbox/submit-site-url](http://www.bing.com/toolbox/submit-site-url)
- SOSO搜搜网站收录提交入口: [http://www.soso.com/help/usb/urlsubmit.shtml](http://www.soso.com/help/usb/urlsubmit.shtml)
- 雅虎中国网站登录口：[http://sitemap.cn.yahoo.com/](http://sitemap.cn.yahoo.com/)
- 百度博客提交: [http://utility.baidu.com/blogsearch/submit.php](http://utility.baidu.com/blogsearch/submit.php)
- 博客大全提交：[http://lusongsong.com/daohang/login.asp](http://lusongsong.com/daohang/login.asp)
- Google博客提交：[http://blogsearch.google.com/ping](http://blogsearch.google.com/ping)
- 必应 Bing博客提交：[http://www.bing.com/toolbox/submit-site-url](http://www.bing.com/toolbox/submit-site-url)
- 搜狗(SoGou)博客提交：[http://www.sogou.com/feedback/blogfeedback.php](http://www.sogou.com/feedback/blogfeedback.php)


---
### 有用的连接
[常用的hexo命令](http://hexo.io/docs/commands.html)