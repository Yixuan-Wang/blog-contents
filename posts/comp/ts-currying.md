---
title: "Tymnastics (2): 柯里化"
date: 2023-02-21
category: comp
series: [tymnastics, 2]
tags:
  - pl
  - typescript
keywords:
  - type
  - fp
---

用 TypeScript 的类型系统描述一个函数的柯里化过程。

<!-- more -->

## 柯里化

柯里化（[currying](https://en.wikipedia.org/wiki/Currying)）是函数式编程中常见的一种技术。它的定义如下：

1. 一个只接收一个参数或者没有参数的函数 $f$, 它的柯里化得到它自身;
2. 一个函数 $f(x_1, x_2, \dots, x_n)$，它的柯里化得到接收 $x_1$ 作为参数, 返回一个接收 $x_2, \dots, x_n$ 作为参数且返回 $f(x_1, x_2, \dots, x_n)$ 的函数的柯里化结果.

看一个例子：

```typescript
let add         = (a: number, b: number, c: number) => a + b + c;
let curried_add = (a: number) => (b: number) => (c: number) => a + b + c;

console.log(add(1, 2, 3)); // 6
console.log(curried_add(1)(2)(3)); // 6

// 等价于:
let add_1 = curried_add(1);
let add_1_2 = add_1(2);
let result = add_1_2(3);
```

考虑到 Haskell 的类型注解，可以明白柯里化在函数式编程中的重要性。顺便，柯里化（*curry*-ing）得名自逻辑学家 Haskell *Curry*😏。

```haskell
add :: Integer -> Integer -> Integer -> Integer
-- 有三个 Integer 参数且返回一个 Integer 的函数
```

## 柯里化 I

在 TypeScript 中，如何给柯里化作类型注解呢？参看 [Type Challenges](https://github.com/type-challenges/type-challenges) 的[第 17 题](https://github.com/type-challenges/type-challenges/blob/main/questions/00017-hard-currying-1/README.md)。


### 解法 1

根据以上的递归定义，所有的单参数的函数的柯里化是其本身。我们可以以这个条件作为递归的终止条件。于是有：

```typescript
type IsCurried<F> = F extends (..._: any[]) => any
  ? F extends (_: any) => any
    ? true
    : false
  : true;

type Curry<T> = IsCurried<T> extends true
  ? T
  : T extends (p0: infer P0, ...ps: infer Ps) => infer R
  ? (p0: P0) => Curry<(...ps: Ps) => R>
  : never;

declare function currying<F>(fn: F): Curry<F>;
```

首先，我们写出一个类型 `IsCurried<F>`。它首先检查 `F` 是否是函数。若是函数，则返回它是否是单参数的。一个多参数的函数一定不是柯里化过的。特别地，一个无参数的函数的类型等价于 `(_: unknown) => ...`。

其次是主要运算的递归类型 `Curry<T>`。如果 `T` 已经柯里化过了，则直接返回，否则 `T` 是一个多参数的函数，通过 `infer` 关键字对它的类型进行匹配，得到第一个参数 `P0` 、打包的剩余参数 `Ps` 和返回值 `R` 的类型，得到的柯里化的结果应该是 `(_: P0) => Curry<(..._: Ps) => R>`。


### 解法 2

另外一个思路是对参数做文章。观察 Haskell 的函数类型记法，只需要把 `(T, U, V) -> R` 这样的函数转化成 `T -> U -> V -> R` 即可。由此有：


```typescript
type CurriedFunc<P extends any[], R> = P extends []
  ? () => R
  : P extends [infer P0, ...infer Ps]
  ? Ps extends []
    ? (_: P0) => R
    : (_: P0) => CurriedFunc<Ps, R>
  : never;

type Curry<F> = F extends (..._: any[]) => any
  ? CurriedFunc<Parameters<F>, ReturnType<F>>
  : never;

declare function currying<F>(fn: F): Curry<F>;
```

这个思路是在参数列表上进行匹配，而非函数本身，最终从参数列表自内向外构建类型。

## 柯里化 I 的实现

```typescript
function currying<F extends (..._: any[]) => any>(fn: F) {
  if (fn.length <= 1) return fn;
  else {
    return (p0: Parameters<F>[0]) => currying(fn.bind(null, p0));
  }
}
```

实现的思路基本上就是利用 `Function.prototype.bind` 方法依次从左向右绑定参数。


## 柯里化 II

Type Challenges [第 462 题](https://github.com/type-challenges/type-challenges/blob/main/questions/00462-extreme-currying-2/README.md) 提出了一种新的*动态*柯里化，每次不仅能传入一个参数，还可以传入多个，直到在某一次调用中，累计传入了和原函数相同个数的参数，得到返回值。

~~怎么会有这么诡异的需求啊~~🙃？

```typescript
type CurriedFunc<
  Leftover extends any[],
  Ret,
  Pre extends any[] = []
> = Leftover extends []  // (1)
  ? (..._: Pre) => Ret // (1)
  : Leftover extends [infer L0, ...infer Ls] // (2)
  ? Ls extends [] // (3)
    ? (..._: [...Pre, L0]) => Ret // (3)
    : ((..._: [...Pre, L0]) => CurriedFunc<Ls, Ret, []>) // (4)
      & CurriedFunc<Ls, Ret, [...Pre, L0]> // (5)
  //  ^ (6)
  : never;

type DynamiclyCurry<F> = F extends (..._: any[]) => any
  ? CurriedFunc<Parameters<F>, ReturnType<F>>
  : never;

declare function dynamicCurrying<F>(fn: F): DynamiclyCurry<F>;
                              // ^ 在这里加上 (..._: any[]) => any 的约束
                              //   才能通过 Type Challenges 的测试
                              //   但字面量类型 `true` 会被还原成类型 `boolean`
```

延续之前的思路 2。对于一个形如 `T, U, V, ...` 的参数列表，在任意两个类型之间，我们可以选择截断并插入一个 `->` 或不截断。比如，对于 `(T, U, V) -> R` 而言，以下都是合法的动态柯里化结果：
```
(T  ,   U  ,   V) ->  R
(T  ,   U) ->  V  ->  R
 T  -> (U  ,   V) ->  R
 T  ->  U  ->  V  ->  R 
```

所以，把原函数参数列表想象成一个队首在左端的队列，每一个从队首弹出的参数 `T`，有两种选择：压入之前的参数列表后（`(..., T`），或者截断并进入下一层函数（`(..., T) -> (`），或者不截断并留待继续延伸（`(..., T,`）。所以我们的泛型类型设置三个类型参数，`Leftover` 表示原函数参数构成的队列中剩余的参数，`Pre` 表示当前正在构建的参数列表, `Ret` 表示最终的返回值。

所以，首先我们判断 `Leftover` 是否为空，若为空则直接用 `Pre` 和 `Ret` 构建最内层的函数<sup>(1)</sup>。下一步，解包匹配 `Leftover`<sup>(2)</sup>，若只剩最后一项 `L0`，则将 `L0` 附在 `Pre` 后和 `Ret` 构建最内层的函数<sup>(3)</sup>。

进入正题。如果选择截断，则把 `L0` 附在 `Pre` 后构建函数参数（`[...Pre, L0]`），返回值下一层函数（`Pre=[]`）——也就是在其余参数（`Ls`）上的动态柯里化 `CurriedFunc<Ls, Ret, []>`<sup>(4)</sup>。如果选择不截断，则该参数从 `Leftover` 弹出并压入 `Pre`，此时不直接构建任何函数（参数列表还在继续延伸），而是返回进一步动态柯里化的结果，`CurriedFunc<Ls, Ret, [...Pre, L0]>`<sup>(5)</sup>。

这里还有一个问题。得到的不同的柯里化结果，应该用什么逻辑符号相连<sup>(6)</sup>？最开始习惯性地写了析取 `|`，但动态柯里化得到的结果并不是随机选择这些函数类型的其中之一，而是每一个类型所代表的调用链都是合法的。所以这里应该取**合取** `&`。

这个东西的实现有点复杂，因为在 `Function.prototype.bind` 里面使用了剩余参数后，在所有参数都填充了实际值以后并不会调用原函数，而是会返回一个 `() => R` 类型的函数。所以，除了使用 `bind` 方法，每一步动态柯里化返回的结果还需要记录当前已使用的参数个数，所以返回的不再是函数（function），而是 C++ 函子（functor）或函数对象（function object）之类的东西。至于怎么在 JavaScript 里实现 functor，我不会🙃。
