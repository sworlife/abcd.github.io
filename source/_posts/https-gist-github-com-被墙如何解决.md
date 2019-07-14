---
title: 'https://gist.github.com 被墙如何解决'
date: 2019-07-15 06:31:29
tags: 开发工具
categories: 前端技术
comments: true
---

## 前言
被墙往往有两种级别（**封域名**、**封IP**），很幸运 https://gist.github.com 是封域名级别的。因此，我们才有可能通过修改本地 hosts 文件，来避免通过网关来进行域名解析，直接从本地获取 IP 地址进行访问。其技术原理可参加[浏览器DNS查询流程](https://sworlife.github.io/2019/07/15/浏览器DNS查询)

## 修改 hosts 文件

1. 获取 https://gist.github.com 的IP地址
这个很不好意思说: 百度吧!
IP: 192.30.253.118

2. hosts 文件路径

MAC: /etc/hosts

Windows: c/Windows/System32/drivers/etc/hosts

3. 在hosts 文件末尾添加

`192.30.253.118  gist.github.com`