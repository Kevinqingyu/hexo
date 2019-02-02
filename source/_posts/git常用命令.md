---
title: git常用命令
date: 2018-12-18 14:19:30
tags: [Git]
---



### 拉取远端分支
```shell
git fetch
```
<!--more-->
### 删除本地分支
```shell
git branch -D 分支名
```

### 删除远端分支
```shell
git push origin --delete 分支名
```

### 切换分支
```shell
git checkout branch
```

### 回到某次提交
```shell
git reset --hard HEAD^
```

### 合并commit到本分支
```shell
git merge --commit
```

### 合并分支
```shell
git merge branch
```

### 暂存未提交更改
```shell
git stash
```

### 从暂存区取出修改
```shell
git stash apply
```

### 查看当前修改
```shell
git stash
```

### 查看提交记录
```shell
git log
```
