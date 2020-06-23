---
title: Squid代理服务器搭建
tags:
  - squid
  - proxy
categories:
  - 软件
  - squid
abbrlink: c4788f70
date: 2020-05-05 17:36:56
top: false
---


# 前言

Squid是一个Web缓存代理，支持处理HTTP，FTP，GOPHER，SSL和WAIS等协议，它通过缓存域重用经常请求的Web页面，减少带宽使用的同时提升响应时间。

也就是说，如果一个人想下载一web页面，他请求Squid为他取得这个页面。Squid随之连接到远程服务器并向这个页面发出请求。然后，Squid将获取到的页面数据返回到客户端机器，而且同时复制一份。当下一次有人需要同一页面时，Squid可以简单地从磁盘中读到它，那样数据迅即就会传输到客户机上。

官站：http://www.squid-cache.org/

<!--more-->

# 主要功能

- 缓存网站内容，以达到分担源站压力加快访问速度的目的。
  - 热点缓存，只缓存访问热度到达设定级别的网站内容。
  - 合并回源，多个相同的请求只回源一次。

- ACL访问控制，可针对源IP、目的地IP、域名、URL、访问时间、单一最大连接数限制访问行为。或通过外部程序验证访问者（proxy_auth）。

- 主要支持协议：HTTP、HTTPS、FTP

- 网页内容篡改，可根据需求篡改网站内容。

- 网站头部篡改，可根据需求篡改请求头部。

- 可针对不同的域名或url配置不同的缓存规则。

# 应用场景

- 正向代理（本地网关）
  - 正向代理的典型用途是为在防火墙内的局域网客户端提供访问Internet的途径。
  - 正向代理还可以使用缓存特性减少网络使用率。
  - 正向代理允许客户端通过它访问任意网站并且隐藏客户端自身，因此你必须采取安全措施以确保仅为经过授权的客户端提供服务。
- 透明代理（cdn，架设于网络运营商主干机房）
  - 提高各个地区访问者的访问速度。
  - 减少源站压力。
  - 减少网络运营商的网间结算费用。
  - 节省网络运营商带宽资源。
- 反向代理（网站前端）
  - 降低源站服务器的负载。
  - 隐藏源站真实ip。

# 安装服务

Debian安装

```bash
apt install squid -y
```

Centos安装

```bash
yum install squid -y
```

# 服务配置

安装完成后需要修改配置文件，文件默认路径：`/etc/squid/squid.conf`，查看原始配置

```bash
grep -v "^#" /etc/squid/squid.conf |grep -v "^$"
```

可以将原文件备份，去掉注释和空行后输出到配置文件中

```bash
mv /etc/squid/squid.conf /etc/squid/squid.conf.bak
grep -v "^#" /etc/squid/squid.conf.bak |grep -v "^$" >  /etc/squid/squid.conf
```

## 端口及https协议

注释http端口，配置https，直接使用http也行，证书自动注册可以参考{% post_link acme-install acme.sh申请免费证书 %}，本人是复用了caddy申请的证书。

```bash
# http_port 3128
https_port 8443 tls-cert=/path/to/ssl/domain.com.cert tls-key=/path/to/domain.com.key
```

吐槽：squid3.x和squid4.x配置有个微小的修改，3.x中直接配置`cert=xxx.crt key=xxx.key`，squid4就变成`tls-cert=xxx.tls-crt key=xxx.key`了，怎么都喜欢不向下兼容呢。

## 用户认证配置

生成密码文件，htpasswd在apache里面，也可以不安装，用工具在线生成，将结果写入文件就行。

```bash
apt-get install apache2-utils
htpasswd /etc/squid/squid_passwd username
```

`username`替换成想要的用户名，如果要多个用户则运行多次该命令。

修改`/etc/squid/squid.conf`，在`http_access deny all`之前添加用户认证配置。

```
auth_param basic program /usr/lib64/squid/basic_ncsa_auth /etc/squid/squid_passwd
auth_param basic children 5
auth_param basic realm Squid proxy-caching web server
auth_param basic credentialsttl 30 minutes
auth_param basic casesensitive on
acl ncsa_users proxy_auth REQUIRED
http_access allow ncsa_users
```

`auth_param basic program`：指定密码文件和用来验证密码的程序，验证程序`basic_ncsa_auth`的路径在不同32和64位系统中路径是不同的，可以使用命令来检测

```
# Debian
dpkg -L squid | grep ncsa_auth
# Centos
rpm -ql squid | grep ncsa_auth
```

`auth_param basic children` ：鉴权进程的数量，即最多同时在线用户数

`auth_param basic realm`：输入用户名和密码时用户看到的提示信息

`auth_param basic credentialsttl`：用户名的缓存时间，即同一个用户多久会调用一次`basic_ncsa_auth`

`auth_param basic casesensitive`：用户名是否匹配大小写

`acl ncsa_users proxy_auth REQUIRED`：定义一条名为ncsa_users的用户组，所有鉴权成功的用户都归入ncsa_users组

`http_access allow ncsa_users`：允许ncsa_users用户组使用代理



## 用户行为限制

限制用户最高下载10m的文件，且`mp3、exe、zip、rar、apk、mp4、msi`等文件不能下载。

```
reply_body_max_size 10 MB
acl download urlpath_regex –i \.mp3$\.exe$\.zip$\.rar$\.apk$\.mp4$\.msi$
http_access deny download
```

禁止多线程流量

```
acl rangeget req_header Range .*
http_access deny rangeget
```

隐藏客户端IP

```
request_header_access Via deny all
request_header_access X-Forwarded-For deny all
request_header_access From deny all
```

测试是否隐藏客户端IP（不显示客户端IP即成功)[测试地址](http://httpbin.org/ip )

# 启动服务

```
#重启服务
systemctl restart squid
#开机自启
systemctl enable squid
```

在使用过程中，发现运行时间一长squid需要重启才能正常运行，也懒得去找原因，就配置了每日重启

运行命令

```
crontal -e
```

写入计划任务

```
0 0 * * * /usr/bin/systemctl restart squid
```

重启cron

```
systemctl restart cron
```

# 参考配置

```bash
#
# Recommended minimum configuration:
#

# Example rule allowing access from your local networks.
# Adapt to list your (internal) IP networks from where browsing
# should be allowed
acl localnet src 10.0.0.0/8     # RFC1918 possible internal network
acl localnet src 172.16.0.0/12  # RFC1918 possible internal network
acl localnet src 192.168.0.0/16 # RFC1918 possible internal network
acl localnet src fc00::/7       # RFC 4193 local private network range
acl localnet src fe80::/10      # RFC 4291 link-local (directly plugged) machines

acl SSL_ports port 443
acl Safe_ports port 80          # http
acl Safe_ports port 21          # ftp
acl Safe_ports port 443         # https
acl Safe_ports port 70          # gopher
acl Safe_ports port 210         # wais
acl Safe_ports port 1025-65535  # unregistered ports
acl Safe_ports port 280         # http-mgmt
acl Safe_ports port 488         # gss-http
acl Safe_ports port 591         # filemaker
acl Safe_ports port 777         # multiling http
acl CONNECT method CONNECT
acl download urlpath_regex –i \.mp3$\.exe$\.zip$\.rar$\.apk$\.mp4$\.msi$
acl rangeget req_header Range .*
reply_body_max_size 10 MB
http_access deny rangeget
http_access deny download
#
# Recommended minimum Access Permission configuration:
#
# Deny requests to certain unsafe ports
http_access deny !Safe_ports

# Deny CONNECT to other than secure SSL ports
http_access deny CONNECT !SSL_ports

# Only allow cachemgr access from localhost
http_access allow localhost manager
http_access deny manager

# We strongly recommend the following be uncommented to protect innocent
# web applications running on the proxy server who think the only
# one who can access services on "localhost" is a local user
#http_access deny to_localhost

#
# INSERT YOUR OWN RULE(S) HERE TO ALLOW ACCESS FROM YOUR CLIENTS
#

auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/squid_passwd
auth_param basic children 5
auth_param basic realm Squid proxy-caching web server
auth_param basic credentialsttl 30 minutes
auth_param basic casesensitive on
acl ncsa_users proxy_auth REQUIRED
http_access allow ncsa_users
#http_access deny ncsa_users

# Example rule allowing access from your local networks.
# Adapt localnet in the ACL section to list your (internal) IP networks
# from where browsing should be allowed
http_access allow localnet
http_access allow localhost

# And finally deny all other access to this proxy
http_access deny all
#http_access allow all

# Squid normally listens to port 3128
https_port 8443 tls-cert=/path/to/ssl/domain.com.cert tls-key=/path/to/domain.com.key

# Uncomment and adjust the following to add a disk cache directory.
#cache_dir ufs /var/spool/squid 100 16 256
#cache_dir ufs /var/spool/squid 1024 16 256

# Leave coredumps in the first cache dir
coredump_dir /var/spool/squid

#
# Add any of your own refresh_pattern entries above these.
#
refresh_pattern ^ftp:           1440    20%     10080
refresh_pattern ^gopher:        1440    0%      1440
refresh_pattern -i (/cgi-bin/|\?) 0     0%      0
refresh_pattern .               0       20%     4320
# 
request_header_access Via deny all
request_header_access X-Forwarded-For deny all
request_header_access From deny all

```



# 参考资料

[万字长文带你了解最常用的开源 Squid 代理服务器](https://jishuin.proginn.com/p/6430.html)

