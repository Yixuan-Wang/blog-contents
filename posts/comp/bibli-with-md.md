---
title: 在 Markdown 中引用参考文献
date: 2020-12-30T23:05:00+08:00
category: comp
tags:
  - softwares
keywords:
  - markdown
  - bibliography
---

借助 Pandoc，在 Markdown 中引用参考文献非常方便。

<!-- more -->

## 为什么不用 DOCX

> [!TLDR]
> 因为无法避免**干扰性的界面交互**，同时**调整样式**十分困难 😭。

DOCX 这里是广义的，代指 Word、WPS、Pages 等文档编辑软件使用的富文本格式。DOCX 编辑器通常是所见即所得的，也就是“写的是什么样，显示的就是什么样，打印出来就是什么样”。调整样式、参考文献只需要通过操作编辑器界面上的各种按钮。但是，在写作的过程中，不断在**内容和界面之间切换注意力**会干扰到思路。引用本是文章内容的一部分，但在所见即所得编辑器中，它只能通过操作界面来添加</span>，单击引文面板—插入引文—你需要的参考文献。整个过程耗时虽不长，但仍属一种非必要的、令人厌烦的中断。

此外，DOCX 自带的引用及参考文献样式也很少，不能满足需求。自带的引文是著者年份的格式，而没有编号格式（似乎可以用交叉引用解决，但这样就不能用自带的文献管理器）。如果想标注引文的页码，还要手动设置为上标格式，又被迫要操作界面上的按钮。最后插入的“GB7714 标准”参考文献列表，根本不符合最新的 GB7714 标准要求，又要大费周章地修改。

## 为什么不用 LaTeX

> [!TLDR]
> LaTeX 引用很方便，但它的**学习、配置成本**比较高 🤔。

LaTeX 是基于 TeX 的排版系统，在学术上的应用非常之广泛，或许说它是学术界的标准工具也不为过。它引用参考文献（当然，交叉引用等同理）非常简便，配合 **BibTeX**，再加上 cite 和 gbt7714 宏包，在论文尾部加入`\bibliography{文献数据文件}`，可以自动生成参考文献列表；行文中插入一句 `\cite[页码]{文献名}` 指令就可以添加带页码的引用——而且全部符合 GB7714 标准（当然也可以配置其他标准）。引用得以自然地融入了行文过程，输入几个字符的指令带来的干扰比操作用户界面要小得多，速度也快得多。

但是，LaTeX 的指令多且复杂，学习曲线陡峭，配置环境、导言区也比较复杂。个人在实践中常常想不起来某个样式的语法是什么，某个功能是哪一个宏包里提供的，文档必须放在手边。另外， LaTeX 对中文以及整套 Unicode 的支持也需要一番<ruby>配置<rt>zhē teng</rt></ruby>。LaTeX 引用虽然方便，但还是存在不方便之处的。

## Pandoc Markdown 的引用语法

原始的 Markdown 标准（以及许多分支）是**不支持参考文献引用**的（仅支持简单的脚注），但是 [Pandoc](https://pandoc.org/) 的 Markdown 标准支持一套强大的参考文献引用扩展语法。大量论文都要求 DOCX 格式，可以用顺手的 Markdown 编辑器把文章写好，然后用 Pandoc CLI 转换成 DOCX。

[Pandoc 的用户手册](https://pandoc.org/MANUAL.html)很详细，但比较长，这里指路到参考文献相关的[命令行选项](https://pandoc.org/MANUAL.html#citation-rendering)和[语法](https://pandoc.org/MANUAL.html#citations)。关于 Pandoc 可参考[速查表](../sheets/pandoc)。

### 命令行选项

按 \*nix 下 Shell 的语法（Powershell 除了换行是一样的）：

```sh
$ pandoc '[导入]' \
    -C \
    --bibliography '[文献数据]' \
    --csl '[CSL]' \
    -o '[导出]'
```

文献数据用 **BibTeX** 就可以（百度学术、谷歌学术、图书馆的搜索结果都提供 BibTeX）。CSL 是指**引用样式标记语言**（Citation Style Language）。

**2020-12-30 补充**：必须加上 `-C` 选项（或 `--citeproc`）**才能开启引用功能**。在之前的版本中这一功能可能默认启用了，在 Windows 上版本 2.11.3.1 下必须加上这一选项才能启用。

### CSL

CSL 是基于 XML 的，语法很易懂，指路[文档](https://docs.citationstyles.org/en/1.0.1/specification.html)。通过它可以设置引用、参考文献的样式。在[官方 GitHub 库](https://github.com/citation-style-language/styles)里有许多社区贡献的样式文件，GB7714 的有[著者加日期](https://github.com/citation-style-language/styles/blob/1a87f4b4748d0a9a5653aa3b3491d772745c247e/china-national-standard-gb-t-7714-2015-author-date.csl)（著者-出版年制）、[尾注编号](https://github.com/citation-style-language/styles/blob/8628afb2f4b4db13e317cca45b4c384eaae9d83c/china-national-standard-gb-t-7714-2015-numeric.csl)、[脚注编号](https://github.com/citation-style-language/styles/blob/2032fef01713812c184739c030bb18b8c89a89de/china-national-standard-gb-t-7714-2015-note.csl)（顺序编号制），但页码显示部分不合标准，前两种的修复方案如下。

对于前两种格式文件，需要把第 25 行中间 `<single>`、`<multiple>`标签注释掉：

```xml
<term name="page" form="short">
	<single>p.</single>
	<multiple>pp.</multiple>
</term>
```

对于著者加日期 CSL，还需要把第 189 行及之后整个 `<citation>` 标签改成：

```xml
<citation et-al-min="2" et-al-use-first="1" disambiguate-add-year-suffix="true" disambiguate-add-names="true" disambiguate-add-givenname="true" collapse="year">
  <sort>
      <key macro="author-intext"/>
      <key macro="issue-date-year" sort="ascending"/>
  </sort>
  <layout delimiter="; ">
    <group  prefix="(" suffix=")"  delimiter=", ">
      <text macro="author-intext"/>
      <text macro="issue-date-year"/>
    </group>
    <group>
        <label variable="locator" form="short"/>
        <text vertical-align="sup" variable="locator"/>
    </group>
  </layout>
</citation>
```

尾注编号 CSL 则在 176 行，需要改成：

```xml
<citation collapse="citation-number" after-collapse-delimiter=",">
  <sort>
    <key variable="citation-number" sort="ascending"/>
  </sort>
  <layout vertical-align="sup" delimiter=",">
    <text variable="citation-number" prefix="[" suffix="]"/>
    <group>
      <label variable="locator" suffix=". " form="short" strip-periods="true"/>
      <text variable="locator"/>
    </group>
  </layout>
</citation>
```

这样引用的页码显示就符合标准了。

### @ 语法

Pandoc 引入了一个表示引用参考文献的符号，`@`。

#### 行内引用

对于著者加日期，著者在行内，日期在括号里；对于尾注编号，没有上标，不带方括号；对于脚注编号，行内引用和正常相同，显示上标。如：

- 无名氏 (2020)
- 1

语法如下：

```markdown
@文献名
一段话之后跟一个空格 @文献名
```

#### 引用

对于著者加日期，著者和日期都在括号内，逗号分隔；对于尾注编号，显示为上标，带方括号；对于脚注编号，显示为上标，不带方括号。前两种经修正，页码会显示为上标。如：

- (无名氏, 2020)<sup>15, 20–21</sup>
- (无名氏, 2020)
- <sup>[1]15, 20-21</sup>
- <sup>[1]</sup>
- <sup>1</sup>

语法如下：

```markdown
[@文献名]
前面不带空格也可以[@文献名]
[@文献名 {页码}]
[@文献名 {页码,半角逗号分隔,还可以用-横杠}]
```

#### 不显示著者

著者加日期的格式还可以不显示著者，只显示日期（页码照常），如：(2020)。语法如下：

```markdown
-@文献名
[-@文献名]
```

#### 转义

以下情况不会处理成为引用：

```markdown
but-i-am-email@whatever.com
1@不是引用
中文字符@不是引用
+\@不是引用
这里有个空格 \@不是引用
[\@不是引用]
```

### 参考文献列表

参考文献列表会自动附在文章的结尾（所以记得在结尾留一个二级标题 `## 参考文献` ）。可以将以下片段插入你希望生成参考文献列表的位置，就不会再附加在结尾处：

```markdown
::: {#refs}
这里也可以插入一个段落显示在列表前面，但好像没什么用
:::
```

参考文献列表（以及参考文献列表语法中插入的段落）在 DOCX 中会使用“书目”样式。

## 配合 BibTeX

如果需要自己手写 BibTeX，以下是书籍、期刊文章、书籍内文章的格式参考。文献名是一个标签，常用著者加年份后两位表示，但可以随意起名字（中文也可）。当然这里的文献种类和参数没有列全。

```latex
@book{文献名,
  title={书名},
  author={作者 and 作者2 and 作者3},
  year={2000},
  publisher={出版社},
  address={地址},
  edition={版本}
}
@article{文献名,
  title={文章名},
  author={作者},
  year={2000},
  journal={期刊名},
  number={1},#卷数
  pages={1-5},#页码范围
}
@inbook{文献名,
  title={文章名},
  author={作者},
  year={2000},
  pages={1-5},
  booktitle={书名},
  publisher={出版社},
  address={地址},
  edition={版本}
}
```
