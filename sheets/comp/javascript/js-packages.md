---
title: 'JavaScript 相关工具速查表'
date: 2020-08-24T17:13:00+08:00
category: 信息学
tags:
  - JavaScript
  - 工具
---

JavaScript 相关工具的速查表。

<!-- more -->

<a name="http-server"></a>

## http-server

http-server 顾名思义是一个 HTTP 服务器工具，用于在本地进行调试。

```sh
$ http-server [dist]
```

目标默认为当前文件夹（./）。

开启 HTTPS 需要用到 OpenSSL：

```sh
$ openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout key.pem -out cert.pem
$ http-server -S -C cert.pem
```
