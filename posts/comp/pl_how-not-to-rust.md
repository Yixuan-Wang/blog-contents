---
title: How NOT to Rust 讲座笔记
date: 2022-07-10
category: comp
tags:
  - pl
  - rust
keywords:
  - salon
---

TUNA 讲座 How NOT to Rust 的简单笔记，作为种草 Rust 的纪念。

<!-- more -->

原文及视频载 TUNA 博客：[金枪鱼之夜：How NOT to Rust - 语言发展中的失误和补救之选集 | 清华大学 TUNA 协会](https://tuna.moe/event/2021/how-not-to-rust/)。


## 长度为泛型的栈上分配数组

🟢 解决 

需要常量泛型（const generics, [RFCS #2000](https://rust-lang.github.io/rfcs/2000-const-generics.html)）的支持。21 年冬，在 1.51 中[常量泛型终于千呼万唤始出来](https://blog.rust-lang.org/2021/02/26/const-generics-mvp-beta.html)，成功填坑，标准库的方法也不再限制栈上数组长度小于等于 32 了（之前的数组真的有办法用吗？）。

```rust
enum SingleOrArray<T, const N: usize> {
  Single(T),
  Array([T; N]), // <- const generics
}
```

在 C++ 中，这个常量泛型，~~你曾见过的~~，唤作模板非类型形参（[template non-type arguments](https://en.cppreference.com/w/cpp/language/template_parameters#Template_non-type_arguments)）：

```cpp
template<typename T, size_t N>
using single_or_array = std::variant<T, std::array<T, N>>;
```

## 特化

🟠 搁置

C++ 对特化（specialization）的支持很好。除了模板的[特化](https://en.cppreference.com/w/cpp/language/template_specialization)和[偏特化](https://en.cppreference.com/w/cpp/language/partial_specialization)，特别地，C++ 20 的 `concept` 也规定了约束的偏序（[partial ordering of constraints](https://en.cppreference.com/w/cpp/language/constraints#Partial_ordering_of_constraints)），从而支持以下特化魔法：

```cpp
template<typename T>
concept CanA = requires(T t) { t.a(); };

template<typename T>
concept CanAB = requires(T t) { t.b(); } && CanA<T>;
// CanAB is more SPECIFIC than CanA

struct A { void a() {} };
struct AB: A { void b() {} };

template<CanA T>
void print(T t) { std::cout << "CanA" << std::endl; }

template<CanAB T>
void print(T t) { std::cout << "CanAB" << std::endl; }


int main() {
    auto ab = AB();
    print(ab); // CanAB
}
```

但 Rust 的 `trait` 不允许。

```rust
trait Common {}
trait CanA {}
trait CanB {}

impl<T: CanA> Common for T {}
impl<T: CanA + CanB> Common for T {} // E0119: Conflicting implementations
```

尽管可以用 `#![feature(specialization)]` 启用 [RFCS #1210](https://rust-lang.github.io/rfcs/1210-impl-specialization.html) 的特化，但它是非可靠的（unsound），可能引发 UB。这个 RFCS 已经不再继续推进了，可能还是要等待新的 `trait` 解析引擎 `chalk`。

## 可变参数泛型

🔴 无解

在 C++ 中，你可以用 `...` 为模板和函数指定可变的参数列表（[parameter pack](https://en.cppreference.com/w/cpp/language/parameter_pack)），这是一种可变参数泛型（variadic generics）。比如，以下是一个接受不定类型（只需满足特定约束）且不定长度的参数的函数：

```cpp
template<typename T>
concept printable = requires(T t) { std::cout << t; };

template<printable... Ts>
void print_all(Ts... args) {
    ((std::cout << args << ' '), ...) << std::endl;
}

int main() {
    print_all(1, 2.3, 'a', "string");
}
```

但在 Rust 里做不到，只能用宏，但宏不可能像泛型一样展开出无限多个声明。所以在标准库里……

```rust
impl<T0, T1, T2, T3, T4, T5, T6, T7, T8, T9, T10, T11> Debug for (T0, T1, T2, T3, T4, T5, T6, T7, T8, T9, T10, T11)
where
    T0: Debug,
    T1: Debug,
    T2: Debug,
    T3: Debug,
    T4: Debug,
    T5: Debug,
    T6: Debug,
    T7: Debug,
    T8: Debug,
    T9: Debug,
    T10: Debug,
    T11: Debug + ?Sized, // OMG
```

只有力气枚举到 12 个 🤯。其他时候，你只能自己定义 `struct(...)` 并且用 `#[derive]` 演绎，或者手写一个实现。在常量泛型稳定前，数组也是类似的下场（但更惨，枚举了 32 个）。既然常量泛型经过五年长跑终于稳定了，那么可变参数泛型是不是未来可期？非也，它的 [RFCS #376](https://github.com/rust-lang/rfcs/issues/376) 已经卡在草案阶段五年了。

## `Default` 和空数组

🟠 搁置

既然常量泛型稳定了，为什么为数组实现的 `Default` 还是长度为 32 的枚举呢？因为：

```rust
impl<T> Default for [T; 0]
impl<T: Default, const N: usize> Default for [T; N]
```

空数组的元素不是 `Default` 也没问题！所以仍然需要特化才能解决……

## `box` 和 `Box`

🔵 不稳定

Rust 的堆上分配原本由特殊的语法 [`~`](https://rust-lang.github.io/rfcs/0059-remove-tilde.html) 实现，但后来被包装到了 `Box` 结构体，内部实现仍使用特殊语法 [`box`](https://doc.rust-lang.org/unstable-book/language-features/box-syntax.html)。[某教程](https://course.rs/advance/smart-pointer/box.html)会说，使用了 `Box::new()` 就能把巨大的东西扔到堆上了，但……

```rust
#[test]
fn test() {
    let large = Box::new([0; 1000000]); // fatal runtime error: stack overflow
}
```

并不是这样。Rust 没有像 C++ 一样的复制（或移动）擦除保证（copy ellision），自从 C++ 17 开始，包括字面量在内的纯值（prvalue）在用来初始化变量时保证不会调用复制构造函数——但 Rust 根本就*没有*构造函数，`new` 不过是一个普普通通的关联方法。所以，如果没有优化，实参会被分配到栈上，再移动到堆上。因此栈还是爆了。若想要一发上堆，你得打开 Nightly 用 `box` 语法，~~或者开 `unsafe` 手动扔进 BSS 段~~。

## `MaybeUninit`

🟢 解决 

Rust 编译器执行优化时永远假定**值是非空的**。所以，结构体和枚举的内存布局可能按照不存在空值的情况被优化。但如果用 `unsafe` 等手段产生了空值，则可能由于空值的内存表示和合法值冲突，导致 UB。因此，Rust 1.36 增加了一个联合体 `MaybeUninit` 显式地关闭这种内存优化。

```rust
assert_eq!(size_of::<Option<bool>>(), 1);
assert_eq!(size_of::<Option<MaybeUninit<bool>>>(), 2);
```

## “内存泄漏是安全的”

⚫ 特性

Rust 最初认为 `drop` 解构可以保证被调用，所以 Rust 的安全保证包括不会发生内存泄露。但他们忘记了可以用引用计数指针 `Rc` 和内部可变 `RefCell` 拉一个*环*。

```rust
fn safe_forget<T>(data: T) {
    use std::rc::Rc;
    use std::cell::RefCell;

    struct Leak<T> {
        cycle: RefCell<Option<Rc<Rc<Leak<T>>>>>,
        data: T,
    }

    let e = Rc::new(Leak {
        cycle: RefCell::new(None),
        data: data,
    });
    *e.cycle.borrow_mut() = Some(Rc::new(e.clone())); // Create a cycle
}
```

这一在 [std::thread::JoinGuard (and scoped) are unsound because of reference cycles · Issue #24292 · rust-lang/rust (github.com)](https://github.com/rust-lang/rust/issues/24292) 爆出的设计灾难被称作“天机泄露”（Leakpocalypse），最终，内存泄露——以及手动泄露内存 `mem::forget` 都成了安全的，`Rc` 增加了溢出检查，迭代器调用 `drain` 可能导致内存泄露，同时依赖 `drop` 保证作为线程守卫的 `thread::scoped` 也被移除。

不过，最终 [RFCS #3151](https://rust-lang.github.io/rfcs/3151-scoped-threads.html) 决定把 `crossbeam` 库中使用闭包的带作用域线程加回标准库。

## Turbofish

⚫ 特性

Rust 希望能像 C++ 一样在泛型函数调用时使用 `id<T>(...)` 这样的文法而不是 `id::<T>()`。但：

```rust
let (the, guardian, stands, resolute) = ("the", "Turbofish", "remains", "undefeated");
let _: (bool, bool) = (the<guardian, stands>(resolute));
```

打破了这一幻想。为这个语法赐名 Turbofish 的 Rust 团队成员 Anna Harren 后来因病去世，[Turbofish](https://turbo.fish/) 这个语法和名字也成为了她的纪念，留在了 Rust 里。



这场讲座大概是我种草 Rust 的起点，今天有幸见到了主讲本尊，于是决定把这篇干货再回顾了一遍。主讲最后说：

> 虽然 Rust 有这样那样的问题，但是和别的语言比还是很好的！



最后，我们想说：

> C++ 明天更好。