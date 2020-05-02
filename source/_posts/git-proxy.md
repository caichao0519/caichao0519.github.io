---
title: Git代理配置
tags:
  - Git
categories:
  - 软件
  - Git
top: false
abbrlink: aa7e6513
date: 2020-04-29 20:27:53
---

# Git代理设置、查看、取消代理

设置代理：

```
git config --global http.proxy 'http://127.0.0.1:8080' 
git config --global https.proxy 'https://127.0.0.1:8443'
```
<!--more-->

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