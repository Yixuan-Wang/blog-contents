---
title: "Make Python Functional Again: 函数扩展"
date: 2024-08-29
category: comp
tags:
  - pl
  - python
  - haskell
series: [py-fp, 1]
keywords:
  - fp
  - haskell
math: true
---

构建一个更函数式的 Python 的尝试：扩展 Python 函数。

<!-- more -->

## 动机

整个系列围绕这一个核心问题——*Python 声称支持多种编程范式，为什么还需要一个更函数式的 Python*？

本篇的切入点是：

> [!IMPORTANT]
>
> Python 缺乏*组合函数*的语言级支持。

在一个值 $x$ 上连续应用一连串函数 $f_i$：
$$
(f_n \circ f_{n-1} \circ \dots f_1) (x),
$$

是一个很常见的需求。一些例子：

- 对于一个 DataFrame，先筛选出符合条件 $p$ 的行，再按列 $a$ 分组，每一组取列 $b$ 求平均值……
- 对于一段数据， 输入 Transformer 架构的神经网络的编码器，先经过嵌入矩阵，加上位置编码，再传入自注意力机制……
- 对于一段代码，一个 parser combinator 实现的前端在 token 序列中提取出一个标识符，返回一个 AST 结点和指向序列中下一个解析位置的指针……

这些需求当然可以通过多步过程式语句来表达，但除去创建中间量的效率疑虑，给中间量起名也是一件难事。像 Pyright 这样的类型检查器可能还会阻止你将不同类型的值绑定到一个不再使用的变量名（一个动态语言的静态类型检查器竟然会不支持这？）。

我们尝试构造一种更漂亮的方法来解决这一问题。

## 在 Python 中不适宜的思路

以下的部分来扩展自[前一篇系列文章](/post/meddle-with-fp-py)。

### 链式调用

即使不是函数式语言，没有组合函数的一等支持，最常见的解决方法是特殊处理函数的首个参数——也就是面向对象范式里的**方法**，搭配上方法调用运算符 `.` ，把 `a.f(...)` 用作 `f(a, ...)` 的语法糖 。一个类定义的方法如果返回对象本身，就可以做**链式调用**了！但如果我们要调用的操作不是类的成员呢？面向对象范式中给一个对象定义的操作是有限的、不可扩展的，所以很多传统的编程语言并不能很好地处理这个问题。

新一些的语言支持在类的定义外扩展一个类的功能。比如 C# 和 Kotlin 的扩展函数（extension functions），Go 的 receiver function 和 Rust 的 `trait`，都可以在类外定义某个对象上的操作。这里~~甚至以最简洁优美的~~ Go 为例：

```go
// https://go.dev/tour/methods/4

type Vertex struct {
	X, Y float64
}

func (v *Vertex) Scale(b float64) *Vertex {
	v.X = v.X * b
	v.Y = v.Y * b
	return v
}

func (v *Vertex) Move(x float64, y float64) *Vertex {
	v.X = v.X + x
	v.Y = v.Y + y
	return v
}

v := Vertex{3, 4}
v.Scale(0.5).Move(1, 2) 
```

Python 是支持 [monkey patching](https://en.wikipedia.org/wiki/Monkey_patch) 的——在运行时动态向类上增加成员，这其实非常灵活：

```python
class A:
    def __init__(self, x, y) -> None:
        self.x, self.y = x, y

def scale(self, factor):
    self.x *= factor
    self.y *= factor
    return self

def move(self, dx, dy):
    self.x += dx
    self.y += dy
    return self

A.scale = scale
A.move = move

a = A(1, 2)
a.scale(2).move(3, 4)
```

但静态类型检查器不能很好地支持 monkey patching。有一个解决方案——手写 `.pyi` 的类型注解文件，但这未免有些繁重。

```python
class A:
    def __init__(self, x: float, y: float): ...
    def scale(self, factor: float) -> Self: ...
    def move(self, dx: float, dy: float) -> Self: ...
```

### 函数组合运算

一些函数式的语言支持直接组合函数，比如 Haskell 的 `$` 和 `.` 运算符。先组合函数再调用的风格被称作 tacit programming 或者 point-free programming。

```haskell
move . scale $ vertex
```

在 Python 里面要想表达组合函数的逻辑：

```python
# https://en.wikipedia.org/wiki/Tacit_programming
from functools import partial, reduce
def compose(*fns):
    return partial(reduce, lambda v, fn: fn(v), fns)

example = compose(foo, bar, baz)
```

不仅需要自己定义辅助函数，静态类型检查器当然也会直接罢工。

JavaScript 提案的 `|>` ，Julia 里的 `|>` 还有 R 的 `%>%` 等链式调用运算符也起到了类似的效果。但 Python 社区对于加入新运算符有着奇怪的偏好。`//`, `@`, `:=` 闯关成功，但用于判空的 `?.` 和 `??` 却被拒绝，不太可能看到任何链式调用的运算符加入 Python。

## 函数组合运算符

这么来看，~~只能自己造轮子了~~。我们可以[设想一个 `FunctionObject` 类](#函数对象类)作为 Python 原生函数类型的扩展。类似 C++ 里面重载 `()` 运算符的操作，Python 也可以重载 `__call__` 魔术方法使得一个类的对象成为可调用对象（`Callable`）。同时再重载其他运算符，即可获得一个更为强大的函数类型。这样做的主要好处是，链式调用和扩展没有静态类型解析支持，*但运算符重载有*。此外，链式调用只能调用绑定在该类上的方法，而运算符可以接收任何对象——因此可以针对任何函数链式使用：

```python
ds.read(path_to_dataset)
	| filter_valid
  | select_col_result
  | map_to_exponential
```

之前和 Teddy Huang 讨论，他提出了一个[想法和实现](https://teddyhuang-00.github.io/zh/posts/DevLog/hacks-to-python.html#pipeline-operator)。我们最初使用了 `|` 这个在 Shell 的管道运算符作为链式调用的运算符。顺着这个思路——前文提到了 Haskell 的 `$` 和 `.` 运算符，Haskell 其实有多种用于函数组合的机制。能否将这些机制通过运算符重载迁移到非常不同的 Python 中呢？

但首先，Python 没有自定义运算符的功能，现有的运算符也十分受限：

- 一些 Python 运算符有默认行为（比如 `in` 返回的结果一定会被强制转换成 `bool`），重载后可能有意想不到的结果。
- 有些 Python 运算符有语法上的特殊处理（比如 `a > b > c` 等价于 `a > (b := b) and b > c`），链式调用会有意外行为。

这些有特殊行为的运算符恰好都是比较运算符（comparison operator）。此外还有一些不可重载的运算符，比如布尔运算符。最终剩下的二元运算符及优先级是：

```python
# ↑ 高
**
+, -, ~        # 一元运算符
*, @, /, //, %
+, -
<<, >>
&
^
|
# ↓ 低
```

所有这些运算符除了 `**` 都是左结合的。

```python
f ** g ** h
**(f, **(g, h)) # 右结合

f @ g @ h
@(@(f, g), h)   # 左结合
```

Haskell 使用最高优先级、左结合的空格 ` ` 表示函数调用。这和 Python 的 `()` 运算符类似（当然，`()` 优先级低于字面量表达式），但 Python 的 `()` 不是二元运算符。而 Haskell 增加的额外的调用运算符 `$` （application operator）优先级为最低的 0 级，结合性为右结合，可以协助节省修改优先级的括号。因而以下表达式都是等价的：

```haskell
 move . scale $ vertex  -- `.` 优先级高于 `$`
(move . scale)  vertex  -- `.` 优先级低于 ` `
 move $ scale $ vertex  -- 右结合
 move $ scale   vertex  -- `$` 优先级低于 ` `
 move  (scale   vertex)
```

`.` 是函数组合运算符（composition operator），和数学记号 $\circ$ 形似且都是右结合，优先级是除函数调用外最高的 9。

此外，还有反向调用运算符 `&`（reverse application operator），参数在左函数在右，优先级为次低的 1，因此可以和 `$` 混用。因而以下表达式都是等价的：

```haskell
vertex &  scale  & move    -- 左结合
move   $  vertex & scale   -- `&` 优先级高于 `$`
vertex &  move   . scale   -- `&` 优先级低于 `.`
move     (vertex & scale)
move     (scale    vertex)
```

综合来看在 Python 中：

- `**` 优先级最高，且为右结合，对应函数组合运算符 `.`
- `@` 优先级次高，左结合，使用较少，且让人联想到装饰器 `@` 语法 ，可以作为高优先级的函数调用运算符 ` `
- `&` 优先级为第三低，且为左结合，对应反向函数调用运算符 `&`
- `|` 优先级为最低，对应调用运算符 `$`，但为左结合

一个重要的缺陷是 `|` 不是右结合，因此无法复刻 `$` 取代嵌套括号的作用，不过可以用 `**` 代替。此外，`@` 的优先级比函数组合 `**` 低。

那么，以上的 Haskell 表达式就可以换成：

```python
move ** scale | vertex
move ** scale @ vertex
move |  scale @ vertex
move @ (scale @ vertex)

vertex &  scale  &  move
move   |  vertex &  scale
vertex &  move   ** scale
move   @ (vertex &  scale)
```

但 `&` 有一个隐藏的问题——我们将在下文提到。

## 偏函数

Haskell 的另外一个特性是自动柯里化（currying），搭配上高阶函数诸如 `flip`，很容易创建偏函数。

Python 提供了 `functools.partial`，标准库提供了原生实现，可以创建非常高性能的偏函数。我们可以给我们的函数对象增加一个方法 `bind`：

 ```python
 f.bind()         == f
 f.bind(1)        == partial(f, 1)
 f.bind(a=1, b=2) == partial(f, a=1, b=2)
 f.bind(1,   b=2) == partial(f, 1,   b=2)
 ```

对于只有位置参数（positional argument）或者只有关键字参数（keyword argument）的情况，我们可以借用格式化字符串运算符 `%` 作为 `bind` 的运算符形式。这里的 `%` 的优先级仅次于 `**`。

```python
f % ()               == f
f % (1,)             == partial(f, 1)
f % {"a": 1, "b": 2} == partial(f, a=1, b=2)
```

那么柯里化呢？注意到我们在前一节引入的运算符都是二元运算符，因此对于没有自动柯里化的 Python 来说有些多余。我们可以略微更改 `@` 的行为：假如函数剩余的位置参数多于一个，则 `@` 先对函数执行柯里化；假如剩余的位置参数恰为 1 个，则即时调用函数，以右操作数为参数；假如剩余的位置参数为 0 个，则丢弃右操作数，直接调用函数。

```python
def add(a, b):
    return a + b

add @ 5            ==  lambda x: 5 + x
add @ 5 @ 5        == (lambda x: 5 + x)(5)
```

## 函数对象类

前文我们假定我们的 `f` 都是一个自定义的 `FunctionObject` 类型的实例。那么，如何实现这个 `FunctionObject` 类型呢？

首先，我们可以不引入一个新类型吗？Python 中使用 `def` 声明的函数都是内置的 `function` 类型——这个内置类型是不支持通过 monkey patching 重载运算符的！所以新的 `FunctionObject` 类型是必须的。

```python
def f(a): return a + 1
setattr(f, "__or__", f.__call__)
# f | 1
#   ^ TypeError: unsupported operand type(s) for |: 'function' and 'int'
```

要想让我们的新类 `FunctionObject` 尽可能快，我们可以通过指定 `__slots__` 关闭它的 monkey patching 功能，把 `__call__` 魔术方法设置为在构造函数中才动态地绑定：

```python
class FunctionObject:
    __slots__ = ("__call__",) # 不把 __call__ 设为 method_descriptor 的话这个类就无法重载

    def __init__(self, f):
      self.__call__ = f
```

我们静态重载的运算符不需要加入 `__slots__`。示例：

```python
class FunctionObject：
    ...

    def __or__(self, rhs):
      return self.__call__(rhs)
```

但 `&` 有一个问题：这里，我们的函数永远出现在操作符右侧，而 Python 会先尝试调用左操作数的魔术方法 `__and__`。我们可以通过重载 `__rand__` 实现这个逻辑，但假如左操作数有 blanket implementation，即对于任何类型的右操作数均不返回 `NotImplemented`，这个操作符就无法使用了，比如 `numpy` 的数组会把右侧操作数的 `__rand__` 广播到全部元素上。这个问题我们留待实现函子功能的时候再解决。

## 函数对象类的类型注解

这个函数对象类的类型注解可就有点头疼了。我们设计了柯里化一类的功能，这在截至 Python 3.13 的类型系统里完全无法表示！怎么办？手动展开！

```python
# PYI

class FunctionObject<N>[P1, ...PN, R](FunctionObject):
    __call__: Callable[[P1, ...PN], R]
    
    # 以 bind 为例
    @overload
    def bind(self, arg1: P1) -> FunctionObject<N-1>[P2, ...PN, R]: ...
    @overload
    def bind(self, arg1: P1, arg2: P2) -> FunctionObject<N-2>[P3, ...PN, R]: ...
    ...
```

至于未定义的部分，比如在一个接受 2 个参数的函数上调用 `|`，我们可以把参数的类型注为 `Never`。

另外，我们再为有关键字参数的函数单独设计一些类型：

```python
# PYI

class FunctionObjectP<N>[P1, ...PN, R, **P](FunctionObject):
    __call__: Callable[Concatenate[P1, ...PN, P], R]
  
    @overload
    def bind(self, arg1: P1) -> FunctionObjectP<N-1>[P2, ...PN, R, P]: ...
    @overload
    def bind(self, arg1: P1, **kwargs: P.kwargs) -> FunctionObjectP<N-1>[P2, ...PN, R, P]: ...
    ...
  
    @overload
    def bind(self, **kwargs: P.kwargs) -> FunctionObjectP<N>[P1, ...PN, R, P]: ...
```

手动生成这样的注解放入 `.pyi` 即可。这些类都是静态检查时才存在的傀儡，运行时不存在。在 `.pyi` 文件中，我们同时重载 `FunctionObject` 的 `__new__` 到各个傀儡子类即可：

```python
# PYI

class FunctionObject: 
    @overload
    def __new__[R](cls, f: Callable[[], R]) -> FunctionObjectA0[R]: ...
    @overload
    def __new__[P1, R](cls, f: Callable[[P1], R]) -> FunctionObjectA1[P1, R]: ...
    @overload
    def __new__[P1, P2, R](cls, f: Callable[[P1, P2], R]) -> FunctionObjectA2[P1, P2, R]: ...
    ...
```

Pylance 可以识别 `__call__` 的签名给出相应的签名提示，但显示的类型是我们的傀儡类，原来的文档字符串也不会被渲染。怎么办？我们的 `FunctionObject` 虽然不是 `function` 的子类却有着和 `function` 完全相同的公开接口，因此我们可以使用如下技巧：

```python
@FunctionObject
def func(f):
    return FunctionObject(f)

# PYI
def func[F: Callable](f: F) -> F: ...
```

我们不使用 `FunctionObject` 作为公开接口，而是新建一个辅助函数 `func`，并且在 `.pyi` 文件中声明它不改变函数的类型，运行时却暗渡陈仓把它套上 `FunctionObject`。同时我们再创建一个 `reveal_func` 的辅助函数，拥有和 `FunctionObject.__new__` 相同的重载，但运行时只是一个恒等映射。这样我们就可以做到按需开关类型提示：在需要使用我们自定义的运算符时再开启类型提示，在定义函数的时候则保留原来的签名和文档。

## 上游和下游

我们的 `FunctionObject` 扩展了 Python 内置的 `function`，但带来的是函数定义处和执行处的额外开销。所以，这个类不适用于在上游的**库**（library）中使用，而适用于下游的**应用**（application），比如你在实验时跑的 Jupyter Notebook——省去 `()` 包裹的麻烦在这类~~使用不了 Vim 的~~场景下特别有用。在关键路径里使用这些 `FunctionObject`（比如 PyTorch `nn.Module` 的 forward pass）也应当避免。
