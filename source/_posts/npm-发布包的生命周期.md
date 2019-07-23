---
title: npm 发布包的生命周期
comments: true
date: 2019-07-23 11:03:29
tags: 工程化
categories: 前端技术
---

## 发布

`npm publish` 用来发布包，发布时读取的配置是 `package.json` 文件，比如：name 为包名字，version 为版本号。

## 更新

`npm publish` 同时也用来更新包，但一定要更改 `package.json` 的 version 字段

## 停更或撤销

有时候我们会觉得这个包功能已经完善了，或者我放弃了，再或者我没有精力继续了等，可以使用 `npm deprecate <pkg>[@<version>] <message>`, 这样，npm 会给尝试安装这个包的人发警告。还有一种情况，我想让 npm 上让这个包消失，`npm unpublish --force`可以达成愿望，但必须在发布的24小时以内。并且，即使撤销成功，也再不能使用相同的名字重现发布了。

