---
abbrlink: 42009ab4
---
下载git

https://git-scm.com/downloads



## 配置用户名和邮箱

~~~shell
git config --global user.name "XXX"
git config --global user.email "XXXXXX@gmail.com"
~~~

注：此用户名和邮箱是 git 提交代码时用来显示你身份和联系方式的，并不是 GitHub 用户名和邮箱。

**查看命令**

~~~shell
git config user.name
git config user.email
~~~



## 配置SSH秘钥

**1、运行 Git Bash**

2、**生成秘钥对**

查看是否存在秘钥，windows秘钥存放位置：

```
C:/Users/用户名/.ssh
```

3、**生成秘钥**

```
ssh-keygen -t rsa -C "caichao198805@gmail.com"

```

一路Enter就行，公钥位置为

```
C:/Users/用户名/.ssh/id_rsa.pub
```

4、添加公钥到github仓库

复制公钥内容

```
cat ~/.ssh/id_rsa.pub
```

登陆GitHub 帐户，点击右上角的头像，然后 

`Settings -> 左栏点击 SSH and GPG keys -> 点击 New SSH key -> 然后你复制上面的公钥内容，粘贴进“Key”文本域内。title域，自己随便起个名字。 -> 点击 Add key。`

测试key是否生效

```
ssh -T git@github.com
```

看见一下就是成功：

`Hi XXX! You've successfully authenticated, but GitHub does not provide shell access.`

# [git设置、查看、取消代理](https://www.cnblogs.com/yongy1030/p/11699086.html)

设置代理：

```
git config --global http.proxy 'socks5://127.0.0.1:1080' 
git config --global https.proxy 'socks5://127.0.0.1:1080'
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