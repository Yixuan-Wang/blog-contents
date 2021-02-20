---
title: 'Pandoc 速查表'
date: 2020-12-30T23:05:00+08:00
category: 信息学
tags:
  - 工具
keywords:
  - 文本处理
---

Pandoc 是一个文本格式转换工具。

<!-- more -->

## 链接

[官网](https://pandoc.org/)  
[基础指南](https://pandoc.org/getting-started.html)，面对不会命令行的人士  
[用户手册](https://pandoc.org/MANUAL.html)

## 支持的格式

以下列出部分常用的格式。完整列表见官网。

- Markdown
- LaTeX
- Microsoft Word(.docx)
- HTML
- EPUB
- OPML
- Jupyter Notebook
- reStructuredText

以上格式支持导入和导出。

- reveal.js
- Microsoft Powerpoint(.pptx)
- LaTeX Beamer
- PDF

以上格式仅支持导出。

## 基本用法

Pandoc 可以根据扩展名自动检测导入导出类型。

使用 `-o` 或 `--output="foo"` 导出文件。如果文件设为 `-` 则直接打印到控制台。

```sh
$ pandoc '[导入]' \
    -o '[导出]'
```

使用 `-f` 指定导入格式，`-t` 指定导出格式。

```sh
$ pandoc '[导入]' \
    -f '[导入格式]' \
    -t '[导出格式]'
```

<!--
```shell
pandoc test.md --bibliography my.bib --csl china-national-standard-gb-t-7714-2015-author-date.csl -o test.docx
```
-->

## 参考文献

```sh
$ pandoc '[导入]' \
    -C \
    --bibliography '[文献数据]' \
    --csl '[CSL]' \
    -o '[导出]'
```
