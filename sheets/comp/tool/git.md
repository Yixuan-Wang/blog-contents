---
title: 'Git 速查表'
date: 2020-07-09T22:01:19+08:00
category: 信息学
tags:
  - 工具
keywords:
  - Git
---

Git 是一个分布式版本控制程序。

<!-- more -->

## 删除

删除分支：

```sh
$ git branch -d '[分支名]'
$ git branch -D '[分支名]' # 强制删除
$ git branch -r -d '[远程仓库名]/[分支名]' # 删除本地监控的远程分支
$ git push '[远程仓库名]' -d '[分支名]' # 删除远程分支
```
