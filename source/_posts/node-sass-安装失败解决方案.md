---
title: node-sass 安装失败解决方案
date: 2019-07-14 16:25:29
tags: 工程化
categories: 前端技术
comments: true
---

```
npm install sass-loader node-sass -D
// -S --save
// -D --save-dev
```

因网络原因，node-sass 中的二进制文件很容易下载失败。
可通过 `.npmrc` 进行配置指定下载链接：
```
// .npmrc
sass_binary_site = https://npm.taobao.org/mirrors/node-sass/
```