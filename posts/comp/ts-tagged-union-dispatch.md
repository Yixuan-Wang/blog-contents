---
title: "Tymnastics (1): 标签联合动态派发"
date: 2022-12-22
category: comp
series: [tymnastics, 1]
tags:
  - pl
  - typescript
keywords:
  - type
---

在 TypeScript 的标签联合（Tagged Union）上类型安全地动态派发。

<!-- more -->

## 什么是 Tymnastics

Tymnastics 即为 **Ty**peScript **M**agics & **Nast**y Tac**tics**😏.

## TL;DR

```typescript
/* 标签联合类型 */
type MyUnion = VariantA | VariantB;

interface VariantA {
  kind: "A";
  valueA: string;
}

interface VariantB {
  kind: "B";
  valueB: number;
}

/* 实现的声明 */
interface Impl<T extends MyUnion> {
  getValue: (variant: T) => string;
}

/* 分量标签 */
type Discriminant<T extends MyUnion> = T["kind"];

/* 派发 */
type Dispatch<T extends MyUnion> = (
  T extends any ? (_: { [_ in Discriminant<T>]: Impl<T> }) => any : never
) extends (_: infer R) => any
  ? R
  : never;

/* 实现 + 派发 */
const implA: Impl<VariantA> = {
  getValue: (variant) => variant.valueA,
};
const dispatch: Dispatch<MyUnion> = {
  A: implA,
  B: {
    getValue: (variant: VariantB) => `${variant.valueB}`,
  },
};

/* 动态派发 */
function dynamic_dispatch(union: MyUnion) {
  return dispatch[union.kind] as Impl<MyUnion>;
}

/* DEMO */
let a: MyUnion = {
  kind: "A",
  valueA: "42",
};

let b: MyUnion = {
  kind: "B",
  valueB: 42,
};

let strings = [a, b].map((union) => dynamic_dispatch(union).getValue(union));
```

## 标签联合和动态派发

在 Rust 中，有一个 [`enum_dispatch`](https://docs.rs/enum_dispatch/latest/enum_dispatch/) crate，在参与派发的类型可枚举的情况下，可以通过过程宏为枚举（`enum`）自动实现要派发的特征（`trait`)，而不需要用到虚表 `dyn`，可以加速动态派发并减少 boilerplate 工作，同时还保证类型安全（当然，这是静态强类型语言）。

TypeScript 提供了类似 Rust 枚举的标签联合类型（tagged union）。通常的实践是，每一个分量（variant）用一个常量标签（tag, discriminant）标注。

```typescript
type Union = VariantA | VariantB; // <- 标签联合

interface VariantA {
  kind: "A"; // <- 标签
  valueA: string; // <- 其他数据
} // <- 分量

// ...
```

TypeScript 的联合，底层仍是 `Object` 类型，分量的类型在编译期被擦除，所以在运行期的动态派发时，需要利用分量的标签。因此，需要一个从分量标签到该分量对应的实现的映射。

```typescript
const dispatch = {
  A: {
    getValue: (variant: VariantA) => variant.valueA,
  },
  B: {
    getValue: (variant: VariantB) => `${variant.valueB}`,
  },
};
```

但是，如何确保这个映射是类型安全的呢？有两个问题需要关注：

1. 确保分量的标签和分量的实现是类型匹配的。也就是确保在 `VariantA` 上调用时会被派发到 `Impl<VariantA>` 的实现。
2. 所有分量是否都有完整且正确的实现？


## 标签映射到实现

标签 $D$ 和实现 $I$ 存在于值空间，而分量 $V$ 只存在于类型空间，前两者都由分量类型确定。因此，为保证映射的正确性，需要遍历联合类型，这在 TypeScript 中应当使用条件类型（conditional type）。联合类型在条件类型中会被展开并逐一计算类型，在计算结果上重新求联合。

```typescript
type UnboxUnion<T> = T extends any ? Something<T> : never;
                   // ^^^^^^^^^^^^^
                   //                 ^^^^^^^^^^^^
type BoxUnion<T> = Something<T>;

// UnboxUnion<A | B> == Something<A> | Something<B>
// BoxUnion<A | B> == Something<A|B>
```

```typescript
type DispatchUnion<T extends MyUnion> = T extends any ? { [_ in Discriminant<T>]: Impl<T> } : never; 
```

其中索引处的 `[_ in Discriminant<T>]` 用于字面量类型转换为值。当联合类型作为 `DispatchUnion` 的类型参数时，每一种类型会分别参与运算（展开？），因为这里泛型 `T` 并没有被包装在其他类型内。


因此确保 `Discriminant<T>` 和 `Impl<T>` 的类型参数始终一致。但得到的结果仍然是 `|` 连接的多个联合类型：

```typescript
type _ = DispatchUnion<MyUnion>; // 等于...
type _ = { A: Impl<VariantA> } | { B: Impl<VariantB> }
                            // ^ 错误的
```

而我们想要的是这些类型的交集类型（intersection）——确保所有分量的实现都是完整且正确的。

## 逆变

如何从联合类型变为交集类型？这里就需要用到逆变（contravariant）。

函数的参数位置就是发生逆变的一个例子。设 `F<T>` 表示 `(_: T): any`，则有如下 Venn 图。

<svg version="1.1" viewBox="0 0 512 512" xmlns="http://www.w3.org/2000/svg"><text x="187.10776" y="129.28905" fill="currentColor" font-family="sans-serif" font-size="40px" style="line-height:1.25" xml:space="preserve"><tspan x="187.10776" y="129.28905"/></text><g transform="translate(.90005 -.60281)"></g><g transform="translate(-.60867 -13.546)"><circle cx="100.76" cy="132.93" r="64.485" fill="none" stroke="currentColor" stroke-width="4"/><circle cx="178.41" cy="132.93" r="64.485" fill="none" stroke="currentColor" stroke-width="4"/><path d="m140.03 81.595a64.485 64.485 0 0 0-25.787 51.377 64.485 64.485 0 0 0 25.662 51.283 64.485 64.485 0 0 0 25.662-51.283 64.485 64.485 0 0 0-25.537-51.377z" fill="currentColor" fill-opacity=".25"/><text x="69.916725" y="141.68504" fill="currentColor" font-family="var(--font-mono)" font-size="24px" style="line-height:1.25" xml:space="preserve"><tspan x="69.916725" y="141.68504" font-family="var(--font-mono)" font-size="24px" text-align="center" text-anchor="middle">T</tspan></text><text x="213.31772" y="141.56505" fill="currentColor" font-family="var(--font-mono)" font-size="24px" style="line-height:1.25" xml:space="preserve"><tspan x="213.31772" y="141.56505" font-family="var(--font-mono)" font-size="24px" text-align="center" text-anchor="middle">U</tspan></text><text x="140.00302" y="238.37758" fill="currentColor" font-family="sans-serif" font-size="24px" style="line-height:1.25" xml:space="preserve"><tspan x="140.00302" y="238.37758" font-family="var(--font-mono)" font-size="24px" text-align="center" text-anchor="middle">T&amp;U</tspan></text></g><g transform="translate(-.60867 -13.029)"><circle cx="334.81" cy="131.21" r="64.485" fill="none" stroke="currentColor" stroke-width="4"/><circle cx="412.46" cy="131.21" r="64.485" fill="none" stroke="currentColor" stroke-width="4"/><path d="m334.27 66.723a64.485 64.485 0 0 0-64.484 64.484 64.485 64.485 0 0 0 64.484 64.486 64.485 64.485 0 0 0 38.824-13.201 64.485 64.485 0 0 0 38.824 13.201 64.485 64.485 0 0 0 64.484-64.486 64.485 64.485 0 0 0-64.484-64.484 64.485 64.485 0 0 0-38.699 13.107 64.485 64.485 0 0 0-38.949-13.107z" fill="currentColor" fill-opacity=".25"/><text x="303.96799" y="139.968" fill="currentColor" font-family="var(--font-mono)" font-size="24px" style="line-height:1.25" xml:space="preserve"><tspan x="303.96799" y="139.968" font-family="var(--font-mono)" font-size="24px" text-align="center" text-anchor="middle">T</tspan></text><text x="447.36899" y="139.84801" fill="currentColor" font-family="var(--font-mono)" font-size="24px" style="line-height:1.25" xml:space="preserve"><tspan x="447.36899" y="139.84801" font-family="var(--font-mono)" font-size="24px" text-align="center" text-anchor="middle">U</tspan></text><text x="374.05429" y="236.66054" fill="currentColor" font-family="sans-serif" font-size="24px" text-align="center" text-anchor="middle" style="line-height:1.25" xml:space="preserve"><tspan x="374.05429" y="236.66054" font-family="var(--font-mono)" font-size="24px" text-align="center" text-anchor="middle">T|U</tspan></text></g><circle cx="102.61" cy="352.21" r="64.485" fill="none" stroke="currentColor" stroke-width="4"/><circle cx="180.26" cy="352.21" r="64.485" fill="none" stroke="currentColor" stroke-width="4"/><path d="m141.88 300.88a64.485 64.485 0 0 0-25.787 51.377 64.485 64.485 0 0 0 25.662 51.283 64.485 64.485 0 0 0 25.662-51.283 64.485 64.485 0 0 0-25.537-51.377z" fill="currentColor" fill-opacity=".25"/><g fill="currentColor" font-size="24px"><text x="75.765312" y="360.97333" font-family="var(--font-mono)" style="line-height:1.25" xml:space="preserve"><tspan x="75.765312" y="360.97333" font-family="var(--font-mono)" font-size="24px" text-align="center" text-anchor="middle">F&lt;T&gt;</tspan></text><text x="207.16631" y="360.97333" font-family="var(--font-mono)" style="line-height:1.25" xml:space="preserve"><tspan x="207.16631" y="360.97333" font-family="var(--font-mono)" font-size="24px" text-align="center" text-anchor="middle">F&lt;U&gt;</tspan></text><text x="141.3116" y="457.66583" font-family="sans-serif" style="line-height:1.25" xml:space="preserve"><tspan x="141.3116" y="457.66583" font-family="var(--font-mono)" font-size="24px" text-align="center" text-anchor="middle">F&lt;T|U&gt;</tspan></text></g><g transform="translate(.90005 -11.226)"><circle cx="330.84" cy="364.64" r="64.485" fill="none" stroke="currentColor" stroke-width="4"/><circle cx="408.49" cy="364.64" r="64.485" fill="none" stroke="currentColor" stroke-width="4"/><path d="m330.3 300.15a64.485 64.485 0 0 0-64.484 64.484 64.485 64.485 0 0 0 64.484 64.486 64.485 64.485 0 0 0 38.824-13.201 64.485 64.485 0 0 0 38.824 13.201 64.485 64.485 0 0 0 64.484-64.486 64.485 64.485 0 0 0-64.484-64.484 64.485 64.485 0 0 0-38.699 13.107 64.485 64.485 0 0 0-38.949-13.107z" fill="currentColor" fill-opacity=".25"/><text x="304.00198" y="373.3992" fill="currentColor" font-family="var(--font-mono)" font-size="24px" style="line-height:1.25" xml:space="preserve"><tspan x="304.00198" y="373.3992" font-family="var(--font-mono)" font-size="24px" text-align="center" text-anchor="middle">F&lt;T&gt;</tspan></text><text x="437.40298" y="373.27921" fill="currentColor" font-family="var(--font-mono)" font-size="24px" style="line-height:1.25" xml:space="preserve"><tspan x="437.40298" y="373.27921" font-family="var(--font-mono)" font-size="24px" text-align="center" text-anchor="middle">F&lt;U&gt;</tspan></text><text x="369.54831" y="470.09174" fill="currentColor" font-family="sans-serif" font-size="24px" text-align="center" text-anchor="middle" style="line-height:1.25" xml:space="preserve"><tspan x="369.54831" y="470.09174" font-family="var(--font-mono)" font-size="24px" text-align="center" text-anchor="middle">F&lt;T&amp;U&gt;</tspan></text></g></svg>

能看出来“逆”，但理解不能？用集合来表示：

$$
\begin{aligned}
(T \times \Omega) \cup (U \times \Omega) &= (T \cup U) \times \Omega \\
(T \times \Omega) \cap (U \times \Omega) &= (T \cap U) \times \Omega
\end{aligned}
$$

而
$$
\begin{aligned}
x \in T \cup U \Rightarrow x \in T \lor x \in U \\
x \in T \cap U \Rightarrow x \in T \land x \in U
\end{aligned}
$$

所以，在我们的类型标注中，$(T \cup U) \times \Omega$ 被记作 `F<T|U>`，而 $(T \cap U) \times \Omega$ 被记作 `F<T&U>`。所以 `F<T>` 和 `F<U>` 的交集是 `F<T|U>`，而并集是 `F<T&U>`。

所以，要想从 `T|U` 获得 `T&U`，你需要：

1. 从 `T|U` 获得 `F<T>|F<U>`
2. 令 `F<T>` 并 `F<U>` 的计算结果为 `F<R>`，推导 `R` 的类型，`R` 是 `T&U`


```typescript
type StepOne<Union> = Union extends any ? F<Union> : never;
type StepTwo<FuncUnion> = (FuncUnion extends any ? FuncUnion : never) extends F<infer R> ? R : never;
```

注意第二步仍然需要上文的展开技巧，因为需要根据 `F<T>` 和 `F<U>` 的并运算的结果做推断，而不是在 `F<T>|F<U>` 类型上做推断。**如果不展开，TypeScript 会把联合类型视作一个整体**。

## 泛型擦除

得到的交集类型，就是我们要的从分量标签到该分量对应的实现的映射的类型。但还有一个小问题：变成了彻底的静态派发。而我们最终想要的是动态派发，即运行时传入一个动态的标签联合类型，派发到正确的实现，而不是传入一个具体的分量，在编译期确定正确的实现。

```typescript
declare let a: VariantA;
declare let b: VariantB;
declare let u: MyUnion;

let implA = dispatch[a.kind]; // Impl<A>
let implB = dispatch[b.kind]; // Impl<B>

let implU = dispatch[u.kind]; // Impl<A> | Impl<B>

// implU.getValue(?) <- 这里的 ? 应该是什么类型？编译期无法确定！
```

所以，我们需要一个 `(_: MyUnion): Impl<MyUnion>` 类型的函数来实现动态派发。具体实现非常简单，对于任何一个 `MyUnion` 类型的值（可能是任何一个分量），取其标签作为下标，我们构造的 `Dispatch<MyUnion>` 将它映射到的值一定是其分量对应的 `Impl`。而我们的静态派发是 `<T extends MyUnion>(_: T): Impl<T>`。所以，我们只需要把所有的泛型 `T` 全部擦除为动态的 `MyUnion` 即可。

```typescript
function dynamic_dispatch(union: MyUnion) {
  return dispatch[union.kind] as Impl<MyUnion>;
}
```

~~好麻烦，还不如用 Rust 🤣~~。以及，由于没有 HKT，这里的 `Discriminant` 和 `Dispatch` 两个泛型每次都需要重新实现一份，~~还不如用 Rust/C++~~，~~你的键盘不是有 Ctrl, C, V 三个键吗~~。

## 参考文献

[TypeScript: Union to intersection type](https://fettblog.eu/typescript-union-to-intersection/).
