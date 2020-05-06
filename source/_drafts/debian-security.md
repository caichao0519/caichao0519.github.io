---
layout: darft
title: debian-security
date: 2020-05-04 23:51:44
tags:
---

# 修改SSH端口





systemctl restart ssh

systemctl status ssh

# 防火墙配置

ufw allow 22/tcp

ufw allow 80/tcp

ufw allow 443/tcp

ufw allow 10048/tcp

ufw enable

ufw status



ufw  status numbered