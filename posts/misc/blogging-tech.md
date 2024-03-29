---
title: 搭建博客的技术性思考
date: 2020-10-11
updated: 2021-02-14
category: comp
tags:
  - frontend
outdated: true
---

搭建静态博客时考虑的技术的的问题。

<!-- more -->

> [!UPDATE]
> 2021-01-09：又重写了博客。本文已经**非常过时**。

> [!UPDATE]
> 2021-02-14：更换为 Gatsby 生成器后，本文内容已经**过时**。

纯前端的静态网站加载速度应当都是快于 WordPress 等后端渲染方案的，而且还能节省下服务器的费用。所以这里主要谈静态的博客。

## 托管

由于 _某些_ 原因，在 ECS 上托管博客并不是个稳妥的选择。因此还是选择静态网站托管商。GitHub Pages 显然是**最方便直接**的选择，因为：

1. GitHub 是一家较为可信的公司（不容易跑路，且服务质量高）
2. GitHub Pages 免费，且功能完整

但是出于 _某些_ 原因，其访问不稳定。因此有必要考虑一个备选方案。

Vercel 专职提供静态网站托管，而且支持从 GitHub 仓库直接部署。但 Vercel 访问速度不够快。

Gitee 也提供静态网站托管服务（需要实名认证），但免费版不支持随 git 提交自动部署，只能手动部署。而且 Gitee 不支持托管某些格式的文件（比如 .txt），此外，有时还会有奇怪的文字编码问题。它的**最大优势就是速度快**。另在 Gitee 上部署一份镜像是可行的方案。

## 部署

> [!UPDATE]
> 2021-02-15：换用 Gatsby 以后不再使用**绝对链接**，替换步骤可以省略了😊。

把部署成品所在的文件夹设为 git 仓库，再上传到 GitHub。我使用的生成器没有提供部署到远程的功能，所以手写了一个脚本，把部署和 git 推送到远程服务器两个步骤绑在一起。

GitHub Action 在提交后触发，制作镜像（很不优雅）：

1. 检出仓库 `master` 分支的最新一次提交到 `master` 文件夹。注意，此时在 CD 服务器上的仓库里只有这一次提交，没有 `master` 分支以往的提交和其他分支，但远程服务器（`remote`）则绑定好了 GitHub 仓库的 `master` 分支。
2. 检出仓库 `gitee` 分支的最新一次提交到 `gitee` 文件夹，这时分支的远程服务器就是 `gitee` 分支。
3. 土味替换 🤦‍：把 `gitee/.git` 文件夹取出来，删掉 `master/.git` 文件夹，再删掉 `gitee` 文件夹（如果直接覆盖，可能有一部分存在于旧版本而不存在于新版本的文件被保留下来，因此必须清空），再把 `master` 文件夹拷贝到 `gitee` 文件夹（其实就是重命名），最后把之前取出来的 `gitee/.git` 放回去。
4. 部署成品里面用于页面间导航的 URL 设成了绝对路径，要把所有 URL 中的 `{username}.github.io/blog/` 替换成 `{username}.gitee.io/blog/`。
5. 提交到 `gitee` 分支。
6. 使用第三方 Action 将库镜像到 Gitee，并触发 Gitee Pages 部署 `gitee` 分支。

同时，GitHub Pages 会自动部署 `master` 分支。

## 静态网站生成器

静态网站生成器多种多样，选用哪一个比较见仁见智，因此放在最后。对于个人来说，我目前使用的是 [**Hugo**](https://gohugo.io)。

不过它也有十分显著的缺点：

1. 接口设计复杂，引入了不少术语，经常需要翻阅文档。
2. 文档内容比较简略，有时一笔带过。
3. 如果需要修改主题，需要一定的 Golang 基础。Golang 的模板语法个人认为有些诡异。
4. 社区不够活跃，生态不够完整。虽然主题较多，但是插件比较少，网上的教程也不多（尤其缺乏中文教程）。
5. Markdown 渲染能力差。Golang 自带的 Markdown 渲染器虽然快，但功能不全，且不够稳定，最后我换用了外置 Pandoc 进行渲染。

因此我选中并维持在 Hugo 主要是看中了它的速度。不过，随着折腾和自定义的欲望增加，我也考虑过迁移，~~只不过出于爱护肝而选择了继续苟在 Hugo~~（已经迁移了 ::真香::）。

[**NexT 主题**](https://theme-next.js.org)基本上由开发者<ruby>包办<rt>bào gān</rt></ruby>了一切，包括评论系统、数学公式，等等，都已经内置好了，教程也十分详细，唯一的缺点就是用的人实在是太多了，可能不够彰显个性。我曾经尝试过 NexT 主题，在规模很小的时候，和 Hugo 速度差异并不显著。

**Vuepress** 是 Vue 官方推出的生成器，主要用于制作 Vue 文档，不过也可以生成博客。我曾经尝试过，似乎在性能方面问题更严重（和 Hugo 采用类似主题而仅有少量测试内容的情况下，启动耗时 7 秒），而且热加载之前似乎存在一些问题（这次测试似乎正常，速度也是 0.1 毫秒量级），因此放弃。

如果想要在博客中如果需要超高的自由度~~且不再需要肝~~，Gatsby / Gridsome 应该是首选。

至于 GitHub Pages 自带的 Jekyll，Jekyll 是基于 Ruby 语言的，Ruby 语言似乎在走下坡路，因而感觉 Jekyll 的前途也有些灰暗。而基于 Python 的 Pelican 和 MkDocs 我没有试过，不过据 OI Wiki 团队说 [MkDocs 自由度不够，在考虑换用 Gatsby](https://github.com/OI-wiki/OI-wiki/issues/1956)。

其他的生成器我没有了解过，在这里就不讨论了。
