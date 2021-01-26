---
title: Git常用操作
tags:
  - Git
categories:
  - 软件
  - Git
top: 88
abbrlink: aa7e6513
date: 2020-04-29 20:27:53
---



# 指定分支上传

查看本地分支情况

```
git branch -a
```

添加所有改变的文件

```
git add .
```

提交代码到本地仓库

```
git commit -m “注释”
```

push到指定分支

```
git push origin hexo
```

<!--more-->

# 恢复本地误删文件

查看本地改动的记录

```
git status
```

指向被删除的文件或文件夹

```
git reset HEAD [被删除的文件或文件夹]
```

恢复被删除的文件或文件夹

```
git checkout [被删除的文件或文件夹]
```

# Git代理设置、查看、取消代理

设置代理：

```
git config --global http.proxy 'http://127.0.0.1:8080' 
git config --global https.proxy 'https://127.0.0.1:8443'
```


查看代理：

```
git config --global --get http.proxy
git config --global --get https.proxy
```
取消代理：

```
git config --global --unset http.proxy
git config --global --unset https.proxy
```