---
title: Hexo命令汇总
tags:
  - hexo
categories:
  - 软件
  - Hexo

date: 2020-03-13 11:32:24
top:80
---



# 前言
汇总一下Hexo的使用命令，方便查找。
# 常用命令
hexo n "my-first-blog" == hexo  new "my-first-blog" #新建文章
hexo g == hexo generate #生成静态页面,生成public文件夹
hexo s == hexo server  #以静态模式启动页面预览 http://localhost:4000
hexo c == hexo clean #清除缓存 ,网页正常情况下可以忽略此条命令,执行该指令后,会删掉站点根目录下的public文件夹
hexo d == hexo deploy #部署到github pages

hexo p == hexo publish
<!--more-->

# 其他命令
hexo n page “tags” == hexo new page “tags” #新建页面

