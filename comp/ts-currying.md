---
title: "Tymnastics (2): 柯里化"
date: 2023-01-12
category: comp
series: [tymnastics, 2]
draft: true
tags:
  - pl
  - typescript
keywords:
  - type
  - fp
---

用 TypeScript 的类型系统描述一个函数的柯里化过程。

<!-- more -->

柯里化（[currying](https://en.wikipedia.org/wiki/Currying)）是函数式编程中常见的一种技术。简单来说，它将一个多参数的函数转换为一个*接收原有的第一个参数*，返回*一个接收余下所有参数作为参数，而返回结果不变的函数*的函数。


```typescript
type Cu<P extends any[], R> = P extends []
  ? (() => R)
  : (
    P extends [infer X, ...infer Xs]
    ? (
      Xs extends []
      ? ((_: X) => R)
      : ((_: X) => Cu<Xs, R>)
    )
    : never
  );
 
type Curried<F> = F extends (..._: any[]) => any ? Cu<Parameters<F>, ReturnType<F>> : never;

declare function Currying<F>(fn: F): Curried<F>
```

