---
title: Tailwind CSS 初探
date: 2021-02-04
category: comp
tags:
  - frontend
keywords:
  - CSS
---

Tailwind CSS 从真香到放弃的个人经历。
<!-- more -->

## Tailwind CSS 简介

[Tailwind CSS](https://tailwindcss.com/) 是一个 CSS 框架，通过类名实现近似内联样式（inline style）但更方便快捷的网页样式编写体验。一段示例代码：

```html
<header
  className="flex justify-between items-center h-16 md:h-20 text-purple-600"
>
  <!-- ... -->
</header>
```

等于如下的 CSS：

```css
.this-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 4rem;
  --tw-text-opacity: 1;
  color: rgba(124, 58, 237, var(--tw-text-opacity));
}

@media (min-width: 768px) {
  .this-header {
    height: 5rem;
  }
}
```

## 关注点分离

> [!TLDR]
> Tailwind CSS **值得一试**，但它显然也**不是万金油**。

Tailwind CSS 的创始人 Adam Wathan 曾经写过一篇[文章](https://adamwathan.me/css-utility-classes-and-separation-of-concerns/)探讨“语义化 CSS 类名”（semantic classes）以及各种传统 CSS 最佳实践的弊端——这有一定的道理。尽管前端三件套的“关注点分离”思想让编写用户界面的体验得到了跨越式的提升，但这三部分最终还是要协同工作的，因此**耦合是不可避免的**。所以，“关注点分离”其实没有那么绝对。~~既然能接受 JSX，当然也可以把 CSS 混进去了！~~

Tailwind CSS 在官网上直接呼吁：

> 尽可能忍住干呕的冲动来尝试一下，你会**真香**并抛弃其他 CSS 实践的。
>
> "If you can suppress the urge to retch long enough to give it a chance, I really think you'll wonder how you ever worked with CSS any other way."

Tailwind 已经比较完善了，截至写作时有 36k 星星，因此值得抛弃对“关注点分离”的坚守，去亲自尝试一下。但最终会不会真香呢？我尝试了一下，确实香，但还是有些不便之处的。

## 优点

虽然 Tailwind CSS 看起来~~只是把内联样式从 style 属性移到了 class 属性并且换成了各种花式简写~~，但它实际上**比直接内联样式好得多**。有了简写，HTML 标签不会像内联样式那么臃肿，而且用 JS 框架调整也更方便简洁。尤其是简化到极致的断点语法 `{breakpoint-type}:{style}` 和暗黑模式语法 `dark:{style}`，比手写媒体查询要爽得多。

它开箱自带**彻底的样式重置**，因此对样式拥有完全的掌控权。这一点也比较舒服。

Tailwind 覆盖的 CSS 样式非常广，官网的文档也写得非常详细，上手还是比较快的。比如，它对长度值`<length>` 有着丰富的支持：

```css
.whatever {
  /* w-4      */
  width: 1rem;
  /* w-auto   */
  width: auto;
  /* w-px     */
  width: 1px;
  /* w-1/3    */
  width: 33.333333%;
  /* w-screen */
  width: 100vw;
}
```

它也支持按需打包（treeshaking）。

另外，通过 PostCSS 转译，它也支持组件复用：

```css
.btn {
  @apply px-4 py-5 text-white font-semibold;
}
```

## 缺点

尽管 Tailwind 覆盖的属性已经非常广了，但它也不可能穷尽所有的值，因此**有时需要手动填写配置文件**，如果你本身就喜欢玩各种 CSS 花样，大规模填写配置文件是个很痛苦的工程。

比如说，需要使用一些 flex 或者 grid 布局，比如[这篇文章](http://www.ruanyifeng.com/blog/2020/08/five-css-layouts-in-one-line.html)中介绍的三明治布局：

```css
.container {
  display: grid;
  grid-template-rows: auto 1fr auto;
}
```

就需要填配置文件：

```js
module.exports = {
  theme: {
    extend: {
      gridTemplateRows: {
        sandwich: "auto 1fr auto", // grid-rows-sandwich
      },
    },
  },
};
```

这时候就产生了非标准的类名。一旦配置多起来，原本 Tailwind 易修改且不损失太多可读性的优势就荡然无存了。

此外，Tailwind 的模式显然对各式 CSS 选择器的兼容性很差。要使用伪类修饰符 `{pseudo-class}:{style}` ，除了一小部分预置的样式，使用其他样式或不那么常用的伪类（`:first-child`）都需要填配置文件。一些伪类如 `:not` 则不支持，对伪元素的支持更是没有，若要使用，还得再额外写 CSS 文件。

最后，即使有缩写，超长的 HTML class 属性确实有碍观瞻，如果提取成组件写进 CSS 文件，一大长串的`@apply` 可读性或许还不如直接写 CSS。

## 总结

如果你是内联样式的爱好者，那 Tailwind 应该很适合你。如果你是关注点分离原教旨主义者，也可以考虑仔细读一读创始人的文章，试一把 Tailwind 看看。不过，Tailwind 虽然已经比较香了，但确实离官方宣传的万金油还差着不少。
