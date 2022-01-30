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

::::::: + [太长不想看]

## 详情

```markdown
::::: + [点击以展开] 
这里有些隐藏的内容。

可以有多段。

::: + [<strong>再点击！</strong>]
还可以嵌套。
:::
:::::
```

::::: + [点击以展开] 
这里有些隐藏的内容。

可以有多段。

::: + [<strong>再点击！</strong>]
还可以嵌套。
:::
:::::

## 卡片、空块和行内元素

```markdown
::: ~
这是一张卡片

圆角的。默认反色。有一些内容。
:::

::: % {.some-class}
而这是一个无聊的空 `<div>`。
:::
```

::: ~
这是一张卡片

圆角的。默认反色。有一些内容。
:::

::: % {.some-class}
而这是一个无聊的空 `<div>`。
:::

```markdown
[平平无奇。]{.some-class}
```

[平平无奇。]{.some-class}


## UnoCSS

```markdown
This is [red]{.text-red-500}, [light]{.font-light}, [Small-caps]{.small-caps}.
While this is [red and light and small-caps]{.text-red-500 .font-light .small-caps}.

This is **bold and red**{.text-red-500}. While this is `monospace and red`{.text-red-500}. 

This one is [a link forced to be red](#详情){.!text-red-500}

Custom values are **fine**{.px-1 .bg-[#114514] .text-white}.
```

This is [red]{.text-red-500}, [light]{.font-light}, [Small-caps]{.small-caps}.
While this is [red and light and small-caps]{.text-red-500 .font-light .small-caps}.

This is **bold and red**{.text-red-500}. While this is `monospace and red`{.text-red-500}. 

This one is [a link forced to be red](#详情){.!text-red-500}

Custom values are **fine**{.px-1 .bg-[#114514] .text-white}.

```markdown
::: ~ { .bg-acc .text-bgd .!mt-2 }
### 块也可以 { .!mb-0 }

有一些内容。
:::
```

::: ~ { .bg-acc .text-bgd .!mt-2 }
### 块也可以 { .!mb-0 }

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
<leipzig-gloss def="#.:#" lang="en,ru,en,en" data="
Russian
My s Marko poexa-l-i avtobus-om v Peredelkino. 
1PL COM Marko go-PST-PL bus-INS ALL Peredelkino
'Marko and I went to Perdelkino by bus.'
" />
```

<leipzig-gloss def="#.:#" lang="en,ru,en,en" data="
Russian
My s Marko poexa-l-i avtobus-om v Peredelkino. 
1PL COM Marko go-PST-PL bus-INS ALL Peredelkino
'Marko and I went to Perdelkino by bus.'
" />
