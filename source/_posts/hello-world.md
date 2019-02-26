---
title: Hexo Blog Guide
date: 2016/4/21 11:32:21
---

## Ready

### install Hexo
``` bash
$ npm install -g hexo-cli
```
### init Floder
``` bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
$ hexo s
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
$ hexo g
```

More info: [Generating](https://hexo.io/docs/generating.html)



### Deploy to remote sites

``` bash
$ hexo deploy
$ hexo d
```

More info: [Deployment](https://hexo.io/docs/deployment.html)


## 自动部署hexo博客到阿里云服务器

### 创建仓库(远端服务器创建git仓库)
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