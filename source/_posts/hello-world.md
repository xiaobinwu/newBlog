---
title: 教你10分钟搭建酷炫的个人博客
date: 2016/4/21 11:32:21
---

## 以个人博客为例

## 准备工作

### 安装
``` bash
$ npm install -g hexo-cli
```
### 初始化
``` bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```

### 创建新文章

``` bash
$ hexo new "My New Post"
```

### 运行开发环境

``` bash
$ hexo server
$ hexo s
```

### 构建

``` bash
$ hexo generate
$ hexo g
```

### 部署

``` bash
$ hexo deploy
$ hexo d
```

详细准备工作，可以查阅[hexo官网](https://hexo.io/zh-cn/)


## 安装主题[cactus](https://github.com/probberechts/hexo-theme-cactus)，一个很程序员的主题，推荐！

### 克隆仓库，并且将源文件复制到博客项目中`themes`目录下
``` bash
git clone https://github.com/probberechts/hexo-theme-cactus.git themes/cactus

```

### `themes/cactus/_config.yml`相关配置,详细解释可以看官方文档
``` bash
# 首页Projects的url
projects_url: https://github.com/xiaobinwu
# 设置页面方向
direction: ltr
# 首页导航
nav:
  home: /
  about: /about/
  articles: /archives/
  categories: /categories/
  search: /search/
# 社交链接
social_links:
  github: https://github.com/xiaobinwu
  mail: mailto:xiaobin_wu@yahoo.com
# 开启标签快捷方式
tags_overview: true
# 首页 Writing的展示条数
posts_overview:
  show_all_posts: false
  post_count: 5
  sort_updated: false
# 排列方式
archive:
  sort_updated: false
post:
  show_updated: false
# logo设置
logo:
  enabled: true
  width: 50
  height: 50
  url: /images/logo.png
  gravatar: false
# ico设置
favicon:
  desktop:
    url: /images/favicon.ico
    gravatar: false
  android:
    url: /images/favicon-192x192.png
    gravatar: false
  apple:
    url: /images/apple-touch-icon.png
    gravatar: false
# 高亮语法
highlight: kimbie.dark
# 博客主题色，dark, light, classic, white
colorscheme: dark
page_width: 48
rss: atom.xml
open_graph:
  fb_app_id:
  fb_admins:
  twitter_id:
  google_plus:
disqus:
  enabled: false
  shortname: cactus-1

google_analytics:
  enabled: false
  id: UA-86660611-1

baidu_analytics:
  enabled: false
  id: 2e6da3c375c8a87f5b664cea6d4cb29c

gravatar:
  email: xiaobin_wu@yahoo.com

valine:
  enable: true
  app_id: xxxxxx
  app_key: xxxxxxx
```

## Valine评论系统
### `themes/cactus/_config.yml`配置Valine，需要申请app_id，app_key
``` bash
valine:
  enable: true
  app_id: xxxx
  app_key: xxxx
```
### `themes/cactus/layout/_partial/comments.ejs`,写入
``` bash
<% if(theme.valine.enable) { %>
    <script src="//cdn1.lncld.net/static/js/3.0.4/av-min.js"></script>
    <script src='//unpkg.com/valine/dist/Valine.min.js'></script>
    <div id="vcomments" class="blog-post-comments"></div>
    <script>
        new Valine({
            el: '#vcomments',
            visitor: true,
            appId: '<%=theme.valine.app_id %>',
            appKey: '<%=theme.valine.app_key %>',
            placeholder: 'ヾﾉ≧∀≦)o来啊，快活啊!',
            avatar: 'robohash'
        })
    </script>
<% } %>
```

## 自动部署hexo博客到阿里云服务器

### 创建仓库(远端服务器创建git仓库),可以使用ssh登入云服务器
``` bash
mkdir blog.git && cd blog.git
git init --bare
```

### Hexo配置
``` bash
deploy:
  type: git
  message: update
  repo: root@xx.xxx.xx.xxx:/www/wwwroot/blog.git,master
```

### 插件安装
``` bash
npm install hexo-deployer-git --save
```

### 自动部署

#### 进入到git仓库hooks目录，并创建钩子post-receive
``` bash
cd /www/wwwroot/blog.git/hooks
touch post-receive
vim post-receive
```
#### 输入下面脚本：
``` bash 
#!/bin/bash -l
GIT_REPO=/www/wwwroot/blog.git
TMP_GIT_CLONE=/www/wwwroot/tmp/blog
PUBLIC_WWW=/www/wwwroot/blog
rm -rf ${TMP_GIT_CLONE}
mkdir ${TMP_GIT_CLONE}
git clone $GIT_REPO $TMP_GIT_CLONE
rm -rf ${PUBLIC_WWW}/*
cp -rf ${TMP_GIT_CLONE}/* ${PUBLIC_WWW}
```

### 更改权限
``` bash
chmod +x post-receive
chmod 777 -R /www/wwwroot/blog
chmod 777 -R /www/wwwroot/tmp/blog
```
### 最后部署
``` bash
$ hexo g -d
```
