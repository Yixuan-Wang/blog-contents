---
title: 页面高级内容测试
date: 2022-01-29
category: misc
tags:
  - test
---

页面高级内容渲染效果的测试。

<!-- more -->

## 扩展 Markdown 语法及样式系统

:::::::details[太长不想看]{open}

### 详情

```markdown
:::::details[点击以展开] 
这里有些隐藏的内容。

可以有多段。

:::details[<strong>再点击！</strong>]
还可以嵌套。
:::
:::::
```

:::::details[点击以展开] 
这里有些隐藏的内容。

可以有多段。

:::details[<strong>再点击！</strong>]
还可以嵌套。
:::
:::::

### 卡片、空块和行内元素

```markdown
:::div{.card .text-back .bg-front}
这是一张卡片

圆角的。默认反色。有一些内容。
:::

:::div{.some-class}
而这是一个无聊的空 `<div>`。
:::
```

:::div{.card .text-back .bg-front}
这是一张卡片

圆角的。默认反色。有一些内容。
:::

:::div{.some-class}
而这是一个无聊的空 `<div>`。
:::

```markdown
:span[平平无奇。]{.some-class}
```

:span[平平无奇。]{.some-class}


### UnoCSS

```markdown
This is :span[red]{.text-red-500}, :span[light]{.font-light}, :span[Small-caps]{.small-caps}.
While this is :span[red and light and small-caps]{.text-red-500 .font-light .small-caps}.

This is :span[**bold and red**]{.text-red-500}. While this is :span[`monospace and red`]{.text-red-500}. 

This one is :a[a link forced to be red]{href="#详情" class="!text-red-500"}.

Custom values are :span[**fine**]{.px-1 class="bg-[#114514]" .text-white}.
```

This is :span[red]{.text-red-500}, :span[light]{.font-light}, :span[Small-caps]{.small-caps}.
While this is :span[red and light and small-caps]{.text-red-500 .font-light .small-caps}.

This is :span[**bold and red**]{.text-red-500}. While this is :span[`monospace and red`]{.text-red-500}. 

This one is :a[a link forced to be red]{href="#详情" class="!text-red-500"}.

Custom values are :span[**fine**]{.px-1 class="bg-[#114514]" .text-white}.

```markdown
:::div{ .card .bg-one-back .text-front }
:h4[块也可以]{ class="!mb-0" }

有一些内容。
:::
```

:::div{ .card .bg-one-back .text-front }
:h4[块也可以]{ class="!mb-0" }

有一些内容。
:::

:::::::

## Ruby

```markdown
[夢](-ゆめ)に[僕](-ぼく)らで[帆](-ほ)を[張](-は)って　
[来](-きた)るべき[日](-ひ)のために[夜](-よる)を[越](-こ)え
```

[夢](-ゆめ)に[僕](-ぼく)らで[帆](-ほ)を[張](-は)って　
[来](-きた)るべき[日](-ひ)のために[夜](-よる)を[越](-こ)え


## 莱比锡标注法

```html
<component is="leipzig-glossing">
  <p>Russian</p>
  <p align lang="ru">My s Marko poexa-l-i avtobus-om v Peredelkino. </p>
  <p align gloss>1PL COM Marko go-PST-PL bus-INS ALL Peredelkino</p>
  <p>'Marko and I went to Perdelkino by bus.'</p>
</component>
```

<component is="leipzig-glossing">
  <p>Russian</p>
  <p align lang="ru">My s Marko poexa-l-i avtobus-om v Peredelkino. </p>
  <p align gloss>1PL COM Marko go-PST-PL bus-INS ALL Peredelkino</p>
  <p>'Marko and I went to Perdelkino by bus.'</p>
</component>
