---
title: 利用githooks让博客实现自动部署
date: 2017-10-27 16:00:02
tags: [nginx, git]
---

为了不让这个vps太过于浪费，不让自己名字命名的域名浪费了，所以决定搭建一个博客在这里。整理一下搭建的过程当做记录。

## 准备阶段

一台服务器，一台可以连接服务器的电脑
<!--more-->
## 开始

这里采用了hexo作为静态页面生成器，hexo的使用教程请点击这里[hexo使用文档](https://hexo.io/zh-cn/index.html)

## 服务器nginx配置

```
server {
  root /home/wwwroot/hexo;  #网站根目录,用来存储网站文件
  index index.html index.htm;
  server_name www.qingyu.ren;   #你的域名
  location / {
    try_files $uri $uri/ /index.html;
  }
}

```

## Git hooks post-receive 配置

```
#!/bin/bash -l
GIT_REPO=/home/git/hexo.git # git 仓库
TMP_GIT_CLONE=/git/tmp/hexo_tmp
PUBLIC_WWW=/home/wwwroot/hexo  # 网站存放目录
rm -rf ${TMP_GIT_CLONE}
git clone $GIT_REPO $TMP_GIT_CLONE
rm -rf ${PUBLIC_WWW}/*
cp -rf ${TMP_GIT_CLONE}/* ${PUBLIC_WWW}
```

这个文件要加上可执行权限
```bash
chmod +x post-receive
```
改变hexo.git的所属用户
```bash
sudo chown -R git:git blog.git
```
## 将本地ssh公钥上传到服务器
将 本机`~/.ssh/id_rsa.pub`里的内容复制到 服务器 `~/.ssh/authorized_keys`里面
这样就可以不用密码登录了
## 本地项目配置 _config.yml

```
deploy:
  type: git
  message: update
  repo: ssh://git@你的域名:端口/~/hexo.git
  branch: master
```

这样在写完blog 的时候 执行hexo d 就会自动部署了







