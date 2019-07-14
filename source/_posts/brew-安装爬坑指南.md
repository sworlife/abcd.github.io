---
title: brew 安装爬坑指南
date: 2019-07-14 16:19:44
tags: 开发工具
categories: 前端技术
comments: true
---

[Homebrew 官网](https://brew.sh/index_zh-cn.html)

安装命令：

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

安装过程很慢，请自备梯子。

好不容易等完进度条，却报错：

```
Homebrew install: Failed during: git fetch origin master:refs/remotes/origin/master -n --depth=1

```
解决方案：[stackoverflow](https://stackoverflow.com/questions/39836190/homebrew-install-failed-during-git-fetch-origin-masterrefs-remotes-origin-mas)

![20190412_stackoverflow](https://user-images.githubusercontent.com/33422051/59659335-93ca8f80-91d8-11e9-8b8e-34fe35193337.jpg)


> 注意事项: 因为之前安装过一次，所以 /usr/local 目录下会创建 Cellar 和 Homebrew 。但重新安装的时候，如果检测到 Cellar 和 Homebrew 中某些文件已存在，仍然会安装失败。因此，需要移除这些目录（如果因权限问题，无法删除目录，可以删除目录里所有文件）
