---
title: 莱比锡标注法
date: 2021-08-03T23:14:00+08:00
category: 语言
tags:
  - 理论语言学
  - 语法
  - 语义
keywords:
  - 马普所
---

莱比锡标注法（Leipzig Glossing Rules）是以语素为单位，标注语料语法、语义的体系。本文简单翻译其规则，方便查阅。

<!-- more -->

## 简介

[莱比锡标注法](http://www.eva.mpg.de/lingua/pdf/Glossing-Rules.pdf)是由马普所进化人类学所的 Bernard 
Comrie 和 Martin Haspelmath 与莱比锡大学的 Balthasar Bickel 共同编写，副标题 Conventions for interlinear morpheme-by-morpheme glosses（逐语素行间标注惯例）既说明了它的基本格式，也表明了它在语言学研究中的重要地位。

本文基于 2015/05/31 的修订稿。原稿参考文献如下：

```
@article{comrie2008leipzig,
  title   = {The Leipzig Glossing Rules: Conventions for interlinear morpheme-by-morpheme glosses},
  author  = {Comrie, Bernard and Haspelmath, Martin and Bickel, Balthasar},
  journal = {Department of Linguistics of the Max Planck Institute for Evolutionary Anthropology \& the Department of Linguistics of the University of Leipzig},
  volume  = {28},
  pages   = {2010},
  year    = {2008}
}
```

## 规则索引

莱比锡标注法主要有如下十条规则：

1. [逐词对齐](#规则-1)：语料和元语言逐词对齐。
2. [逐语素对齐](#规则-2)：语料切分语素后，与元语言逐语素对齐，用连字符连接。
3. [语法标注](#规则-3)：语法标注使用缩写、大写字母表示。
4. [标注连写](#规则-4)：同一个语素对应多个标注，用点连接。
5. [人称和数的连写](#规则-5)：人称和数之间的连写省略。
6. [零形式标注](#规则-6)：零形式承载的语法义包裹在中括号里。
7. [固有语法范畴标注](#规则-7)：语素固有的语法范畴包裹在小括号里。
8. [非连续结构标注](#规则-8)：非连续的结构，每个片段简单重复标注。
9. [中缀标注](#规则-9)：中缀的标注包裹在尖括号里。
10. [重复成分标注](#规则-10)：重复的成分与词根之间用波浪号代替连字符连接。

## 符号

| 符号 | 含义               |
| ---- | ------------------ |
| `-`  | 连接两个语素       |
| `.` | 连接两个语义或语法标注 |
| `=` | 连接词根和黏着语素 |
| `_` | 连接标注中元语言的多个语素 |
| `;` | 连接形式上不可分的语义或语法标注 |
| `:` | 连接形式上可分但省略切分的语义或语法标注 |
| `\` | 连接词根和其上音变构词的标注 |
| `>` | 连接主体和客体论元的语法标注 |
| `~` | 连接重复成分和词根 |
| `[]` | 零形式 |
| `()` | 固有语法范畴 |
| `<>` | 中缀 |


## 规则

### 规则 1

语料和元语言**逐词对齐**。第一行是语料，第二行是用元语言描述的语义标注及语法标注。个人认为，语料 *按音位转写成国际音标* 比较好，能更准确地表示语素（尤其是日语、印地语这样文字系统不是全音素文字的，至少需要转写，才方便在词内部切分语素）。


<div class="leipzig">
  <span class="l-title">
    Indonesian (Sneddon 1996:237)
  </span>
  <span class="l-align">
    <span>
      <span>Mereka</span>
      <span>3<span class="l-abbr">pl</span></span>
    </span>
    <span>
      <span>di</span>
      <span>in</span>
    </span>
    <span>
      <span>Jakarta</span>
      <span>Jakarta</span>
    </span>
    <span>
      <span>sekarang.</span>
      <span>now</span>
    </span>
  </span>
  <span class="l-trans">
    'They are in Jakarta now.'
  <span>
</div>

### 规则 2

**逐语素对齐**是指将语料切分到语素，用连字符 `-` 相连。


<div class="leipzig">
  <span class="l-title">
    Lezgian (Haspelmath 1993:207)
  </span>
  <span class="l-align">
    <span>
      <span>Gila</span>
      <span>now</span>
    </span>
    <span>
      <span>abur<strong>-</strong>u<strong>-</strong>n</span>
      <span>they<strong>-</strong><span class="l-abbr">OBL</span><strong>-</strong><span class="l-abbr">GEN</span></span>
    </span>
    <span>
      <span>ferma</span>
      <span>farm</span>
    </span>
    <span>
      <span>hamišaluǧ</span>
      <span>forever</span>
    </span>
    <span>
      <span>güǧüna</span>
      <span>behind</span>
    </span>
    <span>
      <span>amuq’<strong>-</strong>da<strong>-</strong>č.</span>
      <span>stay<strong>-</strong><span class="l-abbr">FUT</span><strong>-</strong><span class="l-abbr">NEG</span></span>
    </span>
  </span>
  <span class="l-trans">
    'Now their farm will not stay behind forever.'
  <span>
</div>


特别地，若为*黏着语素*（clitic），则用 `=` 相连。


<div class="leipzig">
  <span class="l-title">
    West Greenlandic (Fortescue 1984:127)
  </span>
  <span class="l-align">
    <span>
      <span>palasi<strong>=</strong>lu</span>
      <span>priest<strong>=</strong>and</span>
    </span>
    <span>
      <span>niuirtur<strong>=</strong>lu</span>
      <span>shopkeeper<strong>=</strong>and</span>
    </span>
  </span>
  <span class="l-trans">
    'both the priest and the shopkeeper'
  </span>
</div>


（规则 2-A）若语料中两语素在语音或韵律上分属不同片段，但同属一个词，可以在语料行添加半角空格（个人认为如果应用国际音标记录语料，则可避免应用这条规则，两行的空格数将保持相同）。


<div class="leipzig">
  <span class="l-title">
    Hakha Lai
  </span>
  <span class="l-align">
    <span>
      <span>a-nii&nbsp;-láay</span>
      <span>3<span class="l-abbr">SG</span>-laugh-<span class="l-abbr">FUT</span></span>
    </span>
  </span>
  <span class="l-trans">
    'He/She will laugh'
  </span>
</div>

### 规则 3

表语法义的语素，用语法范畴的缩写表示，使用大写字母，一般是**小型大写字母**。根据标注的实际情况，也可以直接选用元语言的对应词汇。

### 规则 4

若一个语素包含多重词汇或语法意义，不同意义的标注之间**用点号 `.` 连接**。顺序自便。


<div class="leipzig">
  <span class="l-title">
    French
  </span>
  <span class="l-align">
    <span>
      <span>aux</span>
      <span>to<strong>.</strong><span class="l-abbr">ART</span><strong>.</strong><span class="l-abbr">PL</span></span>
    </span>
    <span>
      <span>chevaux</span>
      <span>horse<strong>.</strong><span class="l-abbr">PL</span></span>
    </span>
  </span>
  <span class="l-trans">
    'to the horses'
  </span>
</div>


（规则 4-A）若是出于元语言的限制，调查的语言中某个语素必须由多个元语言语素表示，可用下划线 `_` 替代点号。  
（规则 4-B）若多重的词汇或语法意义在形式上不可分，但又属于不同范畴（比如，多种语法功能的屈折词缀），可用分号 `;` 替代点号。  
（规则 4-C）若多重意义在形式上还可切分（即，未切分到语素层面），但为简便起见不再切分，可用冒号 `:` 替代点号。  
（规则 4-D）若存在音变构词（morphophonological change），可用反斜杠 `\` 替代发生音变的语素中的点号。  
（规则 4-E）若同时存在主体和客体的语法范畴标记，可用大于号 `>` 简写。  


<div class="leipzig">
  <span class="l-title">
    4-A: Turkish
  </span>
  <span class="l-align">
    <span>
      <span>çık-mak</span>
      <span>come<strong>_</strong>out-<span class="l-abbr">INF</span></span>
    </span>
  </span>
  <span class="l-trans">
    'to come out'
  </span>
</div>

<div class="leipzig">
  <span class="l-title">
    4-B: Latin
  </span>
  <span class="l-align">
    <span>
      <span>insul-arum</span>
      <span>island-<span class="l-abbr">GEN</span><strong>;</strong><span class="l-abbr">PL</span></span>
    </span>
  </span>
  <span class="l-trans">
    'of the islands'
  </span>
</div>

<div class="leipzig">
  <span class="l-title">
    4-C: Hittite (Lehmann 1982:211)
  </span>
  <span class="l-align">
    <span>
      <span>n=an</span>
      <span><span class="l-abbr">CONN</span>=him</span>
    </span>
    <span>
      <span>apedani</span>
      <span>that<strong>:</strong><span class="l-abbr">DAT</span>;<span class="l-abbr">SG</span></span>
    </span>
    <span>
      <span>mehuni</span>
      <span>time<strong>:</strong><span class="l-abbr">DAT</span>;<span class="l-abbr">SG</span></span>
    </span>
    <span>
      <span>essandu.</span>
      <span>eat<strong>:</strong>they<strong>:</strong>shall</span>
    </span>
  </span>
  <span class="l-trans">
    'They shall celebrate him on that date.'
  </span>
  <span>
    (<span class="l-abbr">CONN</span> = connective)
  </span>
</div>

<div class="leipzig">
  <span class="l-title">
    4-D: German
  </span>
  <span class="l-align">
    <span>
      <span>unser-n</span>
      <span>our-<span class="l-abbr">DAT</span>.<span class="l-abbr">PL</span></span>
    </span>
    <span>
      <span>V<strong>ä</strong>ter-n</span>
      <span>father<strong>\</strong><span class="l-abbr">PL</span>-<span class="l-abbr">DAT</span>.<span class="l-abbr">PL</span></span>
    </span>
  </span>
  <span class="l-trans">
    'to our fathers'
  </span>
</div>

<div class="leipzig">
  <span class="l-title">
    4-E: Jaminjung (Schultze-Berndt 2000:92)
  </span>
  <span class="l-align">
    <span>
      <span>nanggayan</span>
      <span>who</span>
    </span>
    <span>
      <span>guny-bi-yarluga?</span>
      <span>2<span class="l-abbr">DU</span><strong>&gt;</strong>3<span class="l-abbr">SG</span>-<span class="l-abbr">FUT</span>-poke</span>
    </span>
  </span>
  <span class="l-trans">
    'Who do you two want to spear?'
  </span>
  <span>
    (2<span class="l-abbr">DU</span>&gt;3<span class="l-abbr">SG</span> = 2<span class="l-abbr">DU</span>.<span class="l-abbr">A</span>.3<span class="l-abbr">SG</span>.<span class="l-abbr">P</span>)
  </span>
</div>

### 规则 5

人称和数的语法标记连写时，点号省略。  
（规则 5-A）对于人称、数、性等语法范畴绑定紧密的语言，其间点号皆可省略，标注也可用小写。

<div class="leipzig">
  <span class="l-title">
    Italian
  </span>
  <span class="l-align">
    <span>
      <span>and-iamo</span>
      <span>go-<span class="l-abbr">PRS</span>.<strong>1<span class="l-abbr">PL</span></strong></span>
    </span>
  </span>
  <span class="l-trans">
    'we go'
  </span>
</div>

### 规则 6

通过**零形式**表示的语法范畴，用中括号 `[]` 包裹，或在语料行中插入空记号 `Ø`。


<div class="leipzig">
  <span class="l-title">
    Latin
  </span>
  <span class="l-align">
    <span>
      <span>puer</span>
      <span>boy<strong>[</strong><span class="l-abbr">NOM</span>.<span class="l-abbr">SG</span><strong>]</strong></span>
    </span>
    <span>
      <span>puer-<strong>Ø</strong></span>
      <span>boy-<span class="l-abbr">NOM</span>.<span class="l-abbr">SG</span></span>
    </span>
  </span>
  <span class="l-trans">
    'boy'
  </span>
</div>

### 规则 7

某个语素**固有**的语法范畴（比如性），用小括号 `()` 包裹。

<div class="leipzig">
  <span class="l-title">
    Hunzib (van den Berg 1995:46)
  </span>
  <span class="l-align">
    <span>
      <span>oz#-di-g</span>
      <span>boy-<span class="l-abbr">OBL</span>-<span class="l-abbr">AD</span></span>
    </span>
    <span>
      <span>xõxe</span>
      <span>tree<strong>(</strong><span class="l-abbr">G</span>4<strong>)</strong></span>
    </span>
    <span>
      <span>m-uq'e-r</span>
      <span><span class="l-abbr">G</span>4-bend-<span class="l-abbr">PRET</span></span>
    </span>
  </span>
  <span class="l-trans">
    'Because of the boy the tree bent.'
  </span>
  <span>
    (<span class="l-abbr">G</span>4 = 4th gender, <span class="l-abbr">AD</span> = adessive, <span class="l-abbr">PRET</span> = preterite)
  </span>
</div>


### 规则 8

**二部**（bipartite，或者多部分） 的**不连续**的结构（比如阿拉伯语的词根），简单重复标记即可，或使用 `STEM` 代替词汇语义标记，`CIRC`(circumflex) 代替语法标记。


<div class="leipzig">
  <span class="l-title">
    Lakhota
  </span>
  <span class="l-align">
    <span>
      <span><strong>na</strong>-wíčha-wa-<strong>xʔu̧</strong></span>
      <span><strong>hear</strong>-3<span class="l-abbr">PL</span>.<span class="l-abbr">UND</span>-1<span class="l-abbr">SG</span>.<span class="l-abbr">ACT</span>-<strong>hear</strong></span>
    </span>
    <span>
      <span><strong>na</strong>-wíčha-wa-<strong>xʔu̧</strong></span>
      <span><strong>hear</strong>-3<span class="l-abbr">PL</span>.<span class="l-abbr">UND</span>-1<span class="l-abbr">SG</span>.<span class="l-abbr">ACT</span>-<strong class="l-abbr">STEM</strong></span>
    </span>
  </span>
  <span class="l-trans">
    'I hear them'
  </span>
  <span>
    (<span class="l-abbr">UND</span> = undergoer, <span class="l-abbr">ACT</span> = actor)
  </span>
</div>

<div class="leipzig">
  <span class="l-title">
    German
  </span>
  <span class="l-align">
    <span>
      <span><strong>ge</strong>-seh-<strong>en</strong></span>
      <span><strong class="l-abbr">PTCP</strong>-see-<strong class="l-abbr">PTCP</strong></span>
    </span>
    <span>
      <span><strong>ge</strong>-seh-<strong>en</strong></span>
      <span><strong class="l-abbr">PTCP</strong>-see-<strong class="l-abbr">CIRC</strong></span>
    </span>
  </span>
  <span class="l-trans">
    'seen'
  </span>
</div>

### 规则 9

**中缀**在语料和标注中同时用尖括号 `<>` 包裹。左结合（left-peripheral）则在标注中出现在词干左侧，右结合则在右。

<div class="leipzig">
  <span class="l-title">
    Tagalog
  </span>
  <span class="l-align">
    <span>
      <span>b<strong>&lt;</strong>um<strong>&gt;</strong>ili</span>
      <span><strong>&lt;</strong><span class="l-abbr">ACTFOC</span><strong>&gt;</strong>buy</span>
    </span>
  </span>
  <span class="l-trans">
    'buy'
  </span>
</div>

<div class="leipzig">
  <span class="l-title">
    Latin
  </span>
  <span class="l-align">
    <span>
      <span>reli<strong>&lt;</strong>n<strong>&gt;</strong>qu-ere</span>
      <span>leave<strong>&lt;</strong><span class="l-abbr">PRS</span><strong>&gt;</strong>-<span class="l-abbr">INF</span></span>
    </span>
  </span>
  <span class="l-trans">
    'to leave'
  </span>
</div>

### 规则 10

**重复**的成分与本体间用波浪符 `~` 代替连字符。

<div class="leipzig">
  <span class="l-title">
    Tagalog
  </span>
  <span class="l-align">
    <span>
      <span>bi<strong>~</strong>bili</span>
      <span><span class="l-abbr">IPFV</span><strong>~</strong>buy</span>
    </span>
  </span>
  <span class="l-trans">
    'is buying'
  </span>
</div>

## 扩展链接

[Wikipedia - List of Glossing Abbreviation](https://en.wikipedia.org/wiki/List_of_glossing_abbreviations) 介绍了 Wikimedia 使用的扩展语法，并提供了一个详尽的语法范畴缩写速查表。

也可参考[知乎文章 - 怎么用莱比锡缩写系统描述语言？](https://zhuanlan.zhihu.com/p/32020966)。

## 排版

### Web

本博客中使用的布局如下：

```html
<div class="leipzig">
  <span class="l-title">
    Lezgian (Haspelmath 1993:207)
  </span>
  <span class="l-align">
    <span>
      <span>Gila</span>
      <span>now</span>
    </span>
    <span>
      <span>abur<strong>-</strong>u<strong>-</strong>n</span>
      <span>they<strong>-</strong><span class="l-abbr">OBL</span><strong>-</strong><span class="l-abbr">GEN</span></span>
    </span>
    <span>
      <span>ferma</span>
      <span>farm</span>
    </span>
    <span>
      <span>hamišaluǧ</span>
      <span>forever</span>
    </span>
    <span>
      <span>güǧüna</span>
      <span>behind</span>
    </span>
    <span>
      <span>amuq’<strong>-</strong>da<strong>-</strong>č.</span>
      <span>stay<strong>-</strong><span class="l-abbr">FUT</span><strong>-</strong><span class="l-abbr">NEG</span></span>
    </span>
  </span>
  <span class="l-trans">
    'Now their farm will not stay behind forever.'
  <span>
</div>
```

```css
.leipzig {
  display: flex;
  flex-direction: column;
}

.leipzig .l-align {
  display: flex;
  flex-wrap: wrap;
}

.leipzig .l-align > * {
  display: grid;
}

.leipzig .l-title,
.leipzig .l-trans {
  display: block;
  width: 100%;
}

.leipzig .l-abbr {
  font-variant: all-small-caps;
}
```

可以用以下的 Python 脚本协助切分语料。

```py
import re
import pyperclip

source = input().split()
meta = input().split()

meta = [re.sub('([A-Z]+)', '<span class="l-abbr">\g<1></span>', word) for word in meta]

template_str = '''    <span>
      <span>%s</span>
      <span>%s</span>
    </span>'''

res = '''<div class="leipzig">
  <span class="l-title">
  </span>
  <span class="l-align">
%s
  </span>
  <span class="l-trans">
  </span>
</div>
''' % '\n'.join([template_str % pair for pair in zip(source, meta)])

print(res)
pyperclip.copy(res)
```


## $\LaTeX$

$\LaTeX$ 排版要简便许多。使用 `gb4e` 宏包可以轻松获得漂亮的标注。

```latex
\begin{exe}
\ex{
\gll Gila abur-u-n ferma hamišaluǧ güǧüna amuq^{\prime}-da-č. \\
now they-\textsc{obl}-\textsc{gen} farm forever behind stay-\textsc{fut}-\textsc{neg} \\
\trans `Now their farm will not stay behind forever.'}
\end{exe}
```