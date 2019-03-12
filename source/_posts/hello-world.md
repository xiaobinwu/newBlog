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
### 更换_config.yml 的`theme`配置
``` bash
# theme: landscape
theme: cactus

```
### 进入`themes/cactus/_config.yml`,
``` bash
# 有'dark', 'light', 'classic'，'white'四种可选
colorscheme: dark

```

### 在`themes/cactus/_config.yml`,配置

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
