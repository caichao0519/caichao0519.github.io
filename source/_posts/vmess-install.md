---
title: WebSocket+TLS+Caddy+V2ray+CDN
tags:
  - v2ray
  - vps
  - bbr
  - nginx
  - caddy
  - cloudflare
  - cdn
categories:
  - VPS
abbrlink: 869e664c
date: 2020-05-03 00:32:15
top: false
---


# 前言

之前的VPS要到期了，正好迁移下VPS，顺带整理下资料。

实现方式：WebSocket+TLS+Caddy/Nginx+V2ray+CDN

系统环境：Debian 10

加速：BBR

# 更新系统

**更新系统源和软件包**

```bash
apt-get update && apt-get upgrade
```

<!--more-->

# V2ray

官站：https://www.v2ray.com/

白话文教程：https://toutyrater.github.io/

## 脚本安装

**下载安装**

```bash
bash <(curl -L -s https://install.direct/go.sh)
```

**位置查看**

```bash
whereis v2ray
```


## 编辑配置

```bash
nano /etc/v2ray/config.json
```

- `"port": 10086` 换成自己的端口，未被占用就行
- ID需要换成自己的可以使用生成器：[UUID生成器](https://www.uuidgenerator.net/version1)
- `"path": "/blog/CC"`换成自己的路径，随便填写，与caddy配置匹配就行

```shell
{
        "log": {
                "access": "/var/log/v2ray/access.log",
                "error": "/var/log/v2ray/error.log",
                "loglevel": "warning"
        },
        "inbound": {
                "port": 10086,
                "listen": "127.0.0.1",
                "protocol": "vmess",
                "allocate": {
                        "strategy": "always"
                },
                "settings": {
                        "clients": [{
                                "id": "********-c7b0-4642-bd23-********",
                                "level": 0,
                                "alterId": 32
                        }]
                },
                "streamSettings": {
                        "network": "ws",
                        "security": "auto",
                        "wsSettings": {
                                "path": "/****/***"
                        }
                }
        },
        "outbound": {
                "protocol": "freedom",
                "settings": {}
        },
        "outboundDetour": [{
                "protocol": "blackhole",
                "settings": {},
                "tag": "blocked"
        }],
        "routing": {
                "strategy": "rules",
                "settings": {
                        "rules": [{
                                "type": "field",
                                "ip": [
                                        "0.0.0.0/8",
                                        "10.0.0.0/8",
                                        "100.64.0.0/10",
                                        "127.0.0.0/8",
                                        "169.254.0.0/16",
                                        "172.16.0.0/12",
                                        "192.0.0.0/24",
                                        "192.0.2.0/24",
                                        "192.168.0.0/16",
                                        "198.18.0.0/15",
                                        "198.51.100.0/24",
                                        "203.0.113.0/24",
                                        "::1/128",
                                        "fc00::/7",
                                        "fe80::/10"
                                ],
                                "outboundTag": "blocked"
                        }]
                }
        }
}
```

**验证配置**

```bash
/usr/bin/v2ray/v2ray -test -config=/etc/v2ray/config.json
```

返回`Configuration OK.`就算成功。

## 启动程序

**开机自启**

```bash
systemctl enbale v2ray
```

**启动服务**

```bash
systemctl start v2ray
```

**查看运行状态**

```bash
systemctl status v2ray
```



# Caddy

官站：https://caddyserver.com/

文档：https://caddyserver.com/docs/

最坑的是Caddy升级了，V2版本不向下兼容，Caddyfile V1的配置不兼容了。只能按照官方文档调整。

## 软件安装

**使用源安装**

```bash
echo "deb [trusted=yes] https://apt.fury.io/caddy/ /" \
    | tee -a /etc/apt/sources.list.d/caddy-fury.list
apt update
apt install caddy
```

## 编辑配置

```
nano /etc/caddy/Caddyfile
```

Caddy的配置非常简单，且可以自动申请证书，这也是换用Caddy的原因。


```shell
domain.com
{
  tls 123456@gmail.com
  log /var/log/caddy.log
  root * /var/www/html
  reverse_proxy /blog/CC localhost:10086 {
    header_up -Origin
  }
}
```

## 启动程序

**开机自启**

```bash
systemctl enbale caddy
```

**启动服务**

```bash
systemctl start caddy
```

**查看运行状态**

```bash
systemctl status caddy
```

# BBR加速

BBR有好几个版本，原版、魔改版、BBR Plus版，原版的加速效果是最差的，不过，最后决定安装原版。

**查看内核版本**

开起BBR需要内核4.9以上

```shell
uname -r
```

**添加变量**

```shell
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
```

**命令生效**

```shell
sysctl -p
```

**验证命令是否生效**

```shell
#输入命令
sysctl net.ipv4.tcp_available_congestion_control
#显示结果
net.ipv4.tcp_available_congestion_control = bbr cubic reno
```

```shell
#输入命令
sysctl net.ipv4.tcp_congestion_control
#显示结果
net.ipv4.tcp_congestion_control = bbr
```

**验证BBR是否启动**

```shell
#输入命令
lsmod | grep bbr
#显示结果
tcp_bbr          20480  14
```



# CDN

在外面套一层CDN，即使直连被QQ了也能救回来。使用的是Cloudflare的（别的CDN自然也可以），Cloudflare在国内的速度只能说堪堪能用吧，当然这个要看个人需求。

## 注册域名

要使用CDN，先要有个域名，现在国内外域名价格都差不多，但是国内注册域名要实名验证。

国内域名注册商：[DNSPOD（腾讯）](https://www.dnspod.cn/)，[万网（阿里）](https://wanwang.aliyun.com/)

国外域名注册商：[GoDaddy](https://sg.godaddy.com/)，[NameSilo](https://www.namesilo.com/)

## Cloudflare

**注册Cloudflare**

Cloudflare注册非常简单，有个邮箱就行了，[Cloudflare注册地址](https://dash.cloudflare.com/)。

**域名管理**

登录Cloudflare，会让add site，填入刚注册的域名即可。

![添加域名](https://pic.cc2048.top:8443/i/2020/05/02/12wcifl.png)

填完域名后会提示修改Nameservers，需要到域名注册商处将Nameservers改为Cloudflare的Nameservers。



![DNS添加](https://pic.cc2048.top:8443/i/2020/05/02/12vyc12.png)

**配置SSL/TLS**

配置SSL/TLS，选择Full（strict），选择Flexble会导致黄色云朵点亮的时候，请求失败。

![SSL/TLS](https://pic.cc2048.top:8443/i/2020/05/02/12vs1gz.png)



# 客户端

官方文档里面又有下载地址：https://www.v2ray.com/awesome/tools.html

IOS：Shadowrocket ，收费的，且国区账号下不到了，需要注册美区账号，比较麻烦的

Android：V2RayNG [Play Store ](https://play.google.com/store/apps/details?id=com.v2ray.ang)  |  [GitHub](https://github.com/2dust/v2rayNG)

Windows：Windows 只推荐[V2RayN ](https://github.com/2dust/v2rayN)，其他用了几个都没这个好用。

Linux：Linux的客户端就服务端，只是配置文件需要修改下，还有一个小问题，客户端在安装前，理论上是不能访问外站的，但是使用`bash <(curl -L -s https://install.direct/go.sh)`运行脚本过程中又需要访问外站。先有鸡还是先有蛋:(，所以需要使用go.sh本地安装。

- 访问https://install.direct/go.sh，将go.sh下载到本地
- 去GitHub上下载v2ray的[releases](https://github.com/v2ray/v2ray-core/releases)安装包，将go.sh和安装包放在同一个目录下
- 执行本地安装命令`bash go.sh --loacl ./v2ray-linux-64.zip`

记录一下客户端配置

~~~shell
{
  "log": {
    "access": "",
    "error": "",
    "loglevel": "warning"
  },
  "inbounds": [
    {
      "port": 10808,
      "listen": "127.0.0.1",
      "protocol": "socks",
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls"
        ]
      },
      "settings": {
        "auth": "noauth",
        "udp": true,
        "ip": null,
        "clients": null
      },
      "streamSettings": null
    }
  ],
  "outbounds": [
    {
      "tag": "proxy",
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "domain.com",
            "port": 443,
            "users": [
              {
                "id": "*******-c7b0-4642-bd23-*******",
                "alterId": 32,
                "email": "t@t.tt",
                "security": "auto"
              }
            ]
          }
        ],
        "servers": null,
        "response": null
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "tlsSettings": {
          "allowInsecure": true,
          "serverName": null
        },
        "tcpSettings": null,
        "kcpSettings": null,
        "wsSettings": {
          "connectionReuse": true,
          "path": "/****/**",
          "headers": null
        },
        "httpSettings": null,
        "quicSettings": null
      },
      "mux": {
        "enabled": true
      }
    },
    {
      "tag": "direct",
      "protocol": "freedom",
      "settings": {
        "vnext": null,
        "servers": null,
        "response": null
      },
      "streamSettings": null,
      "mux": null
    },
    {
      "tag": "block",
      "protocol": "blackhole",
      "settings": {
        "vnext": null,
        "servers": null,
        "response": {
          "type": "http"
        }
      },
      "streamSettings": null,
      "mux": null
    }
  ],
  "dns": null,
  "routing": {
    "domainStrategy": "IPIfNonMatch",
    "rules": []
  }
}
~~~

# Nginx

Nginx 和Caddy选择一个就行了，但是使用nginx的话还要配合acme申请证书。可以参考[[acme-install|acme.sh申请免费证书]]
{% post_link acme-install acme.sh申请免费证书 %}

如果配合Cloudflare的话，直接使用Cloudflare的免费证书也行，但是这个就只有Cloudflare中启动 Proxy才有用，但是没有acme方便，所以不做详细介绍。

## 安装配置

正常安装即可

~~~shell
apt install nginx -y
~~~

增加v2ray的nginx配置文件

~~~
nano /etc/nginx/conf/v2ray.conf
~~~

这里除了配置了v2ray还匹配了域名，非指定域名返回500错误

```bash
upstream v2ray {
    server 127.0.0.1:10086;
}

server {
    listen 443 default_server;
    server_name _;
    ssl on;
    ssl_certificate       /ssl/cloudflare/2225470_xx.domain.com_public.crt;
    ssl_certificate_key   /ssl/cloudflare/2225470_xx.domain.com.key;
    ssl_protocols         TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers           HIGH:!aNULL:!MD5;
    return 500;
}
server {
    listen 443 ssl;
    server_name xx.domain.com;
    ssl on;
    ssl_certificate       /ssl/cloudflare/2225470_xx.domain.com_public.crt;
    ssl_certificate_key   /ssl/cloudflare/2225470_xx.domain.com.key;
    ssl_protocols         TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers           HIGH:!aNULL:!MD5;
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;
    location / {
        try_files $uri $uri/ =404;
    }

    location /blog/CC {
        proxy_redirect off;
        proxy_pass http://v2ray;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        proxy_read_timeout 300s;
    }
}
```

## 注意

在Ubutnu上Nginx有一个Bug，`nginx.service: Failed to read PID from file /run/nginx.pid: Invalid argument`，具体信息：https://bugs.launchpad.net/ubuntu/+source/nginx/+bug/1581864

使用root用户执行以下命令即可。

```bash
mkdir /etc/systemd/system/nginx.service.d
printf "[Service]\nExecStartPost=/bin/sleep 0.1\n" > /etc/systemd/system/nginx.service.d/override.conf
systemctl daemon-reload
systemctl restart nginx 
```