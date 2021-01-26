---
tags:
  - caddy
  - forwardproxy
categories:
  - 软件
title: Caddy正向代理
top: false
abbrlink: 95d3d41d
date: 2021-01-26 15:48:00
---


# 前言
之前正向代理一直选择的是squid+acme的方式，但是后来发现squid配置的证书链不生效，也试了其他几个证书都是这个问题，导致有时候手机上面没法使用，所以打算换成caddy，而且使用caddy就不用安装acme了，caddy可以自动申请证书。

# 安装
caddy1已经被官方放弃了，现在只能使用caddy2，而且v1和v2不兼容。使用caddy最重要的原因还是应为配置简单，出问题的话直接检查caddy就行了。

caddy想要使用正向代理功能需要安装http.forwardproxy插件，而该插件是非标插件，官方提供的下载中没有此插件，只能手动编译。

<!--more-->

## 编译caddy

编译带插件的二进制文件并非必要操作，可以直接使用已经编译好的二进制文件。

### go安装

下载go安装包并解压

```SH
wget https://golang.org/dl/go1.15.7.linux-amd64.tar.gz
tar -zxvf go1.15.7.linux-amd64.tar.gz
```

将解压文件移动到任意目录

```shell
mkdir -p /opt/devtools
mv go /opt/devtools/go
```

调整变量配置，将以下变量加入到 shell 初始化配置中

```
cat >>  ~/.profile
export PATH=$PATH:/opt/devtools/go/bin
export PATH=$PATH:$HOME/.cargo/bin
export GOROOT=opt/devtools/go
export GOBIN=$GOROOT/bin
export PATH=$PATH:$GOBIN
```

运行命令使配置生效，或退出重新登录

```
source ~/.profile
```

### xcaddy安装 

xcaddy需要从git下载，要先安装git 

```
apt  install -y git
```

安装xcaddy工具

```
go get -u github.com/caddyserver/xcaddy/cmd/xcaddy
```

### 编译插件

在caddy中打入forwardproxy插件

```
xcaddy build \
    --with github.com/caddyserver/forwardproxy@caddy2
```

完成后会生成caddy的二进制文件，可使用以下命令验证插件是否安装成功

```
./caddy list-modules
```

## 安装caddy

官方安装[文档](https://caddyserver.com/docs/install#debian-ubuntu-raspbian)

```shell
apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/cfg/gpg/gpg.155B6D79CA56EA34.key' | sudo apt-key add -
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/cfg/setup/config.deb.txt?distro=debian&version=any-version' | sudo tee -a /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

替换二进制文件

```shell
rm -f /usr/bin/caddy
mv ./caddy /usr/bin/caddy
```

# 配置

v2可以直接写json作为配置，但还是Caddyfile的配置特别简单，所以还是用Caddyfile，配置文件在/etc/caddy/Caddyfile。

```shell
cat > /etc/caddy/Caddyfile
:443, xxx.domain.com
route {
  forward_proxy {
    basic_auth user pass
    ports     443
    hide_ip
    hide_via
    probe_resistance
  }
}
```

可以通过命令将Caddyfile转成json文件,`caddy adapt /ect/caddy/Caddyfile`

```json
{
    "apps":{
        "http":{
            "servers":{
                "srv0":{
                    "listen":[
                        ":443"
                    ],
                    "routes":[
                        {
                            "handle":[
                                {
                                    "handler":"subroute",
                                    "routes":[
                                        {
                                            "handle":[
                                                {
                                                    "allowed_ports":[
                                                        443
                                                    ],
                                                    "auth_pass_deprecated":"pass",
                                                    "auth_user_deprecated":"user",
                                                    "handler":"forward_proxy",
                                                    "hide_ip":true,
                                                    "hide_via":true,
                                                    "probe_resistance":{

                                                    }
                                                }
                                            ]
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                }
            }
        },
        "tls":{
            "certificates":{
                "automate":[
                    "xxx.domain.com"
                ]
            }
        }
    }
}
```

# 总结

最终发现这个不适合我现在的场景，forwordproxy的插件只支持`:443, xxx.domain.com`这种格式进行匹配，用多域的语法就没法代理，而且只支持443端口，后来想了下，或许直接用json可以？但是不想试了，这个暂时弃用了， 后面考虑考虑用Nginx试试。