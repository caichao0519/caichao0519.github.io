---
title: Hexo命令汇总
tags:
  - hexo
categories:
  - 软件
  - Hexo
top: 80
abbrlink: 8bb4c5fa
date: 2020-03-13 11:32:24
---



# 前言
hexo的命令并不算多，常用命令更少，基本使用一遍就能记得。在此记录一下，方便查找。

# 常用命令
新建文章
`hexo n "my-first-blog"` 

清除缓存、静态文件并生成网站静态文件
`hexo clean && hexo g` 
`hexo g -f`

生成网站静态文件并部署到git
`hexo g -d`  

生成网站静态文件并再本地启动预览
`hexo g && hexo s`

<!--more-->

# 命令说明
```
hexo n "my-first-blog"
```

新建一篇标题为 ***my-first-blog*** 的文章，文件生成在`source/_posts下面`;
也可以直接在目录下新建***my-first-blog.md***文件，不过要手动加 Front-matter ;
`hexo n`是`hexo new`的简写，命令效果一样；
此命令同时等同于`hexo n post "my-first-blog"`，使用`hexo n`命令的格式为`hexo new [layout] <title>`，不带`[layout]`时会依据 `_config.yml`中的`default_layout:`配置填充。

```
hexo n page tags
```

新增一个名为tags的页面，在`source`目录下生成`tags/index.md`文件；
同样可以直接手动新建，也是要手动加 Front-matter ；

```
hexo n draft tags
```

新增一个名为tags的草稿，在`source/_drafts`目录下生成`tags.md`文件；
同样可以直接手动新建，也是要手动加 Front-matter ；
在生成静态页面时，草稿文件不会被渲染。

```
hexo p tags
```

发表草稿tags到正文，将tags.md文件从`source/_drafts`移动到`source/_posts下面`;
手动移动的效果相同，使用命令的好处是，会自动将_posts的模板套用上来；
`hexo p`是`hexo publish`的简写，命令效果一样。


```
hexo g 
```

生成网站静态文件到public目录；
`hexo g -f`强制重新生成文件， 效果类似于`hexo c && hexo g`；
`hexo g`是`hexo generate`的简写，命令效果一样。

```
hexo clean
```

清除缓存文件 `db.json`和已生成的静态文件 `public`；
发现对站点更改的内容无法生效时，使用此命令可以解决。

```
hexo s
```

启动服务器。默认情况下，访问网址为： http://localhost:4000/；
主要用于预览主题，在本地调试站点配置等操作；
`hexo s`是`hexo server`的简写，命令效果一样。

```
hexo d
```

部署网站到制动的仓库，需要配合`hexo-deployer-git插件使用`，安装方式:`npm install hexo-deployer-git --save`；
需要在站点配置文件`_config.yml`中配置仓库信息，可配置多个如做双线的时候，配置git和coding的仓库同时同步；
`hexo d`是`hexo deploy`的简写，命令效果一样。

```
 hexo v
```

查看hexo的版本号，一般在新安装及升级后校验用；
`hexo v`是`hexo version`的简写，命令效果一样。

还有一些其他的命令，但是基本不会用到，有兴趣的可以参见[官方文档](https://hexo.io/zh-cn/docs/commands)	