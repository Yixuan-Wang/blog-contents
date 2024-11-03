---
title: 测速 PyO3 实现的 Python 类
date: 2024-11-02T02:00:00-05:00
category: comp
tags:
  - python
  - rust
keywords:
  - pyo3
---

使用 PyO3 实现的 Python 类型会比 Python `class` 实现的更快吗？是的。

<!-- more -->

## PyO3

[PyO3](https://pyo3.rs) 提供了 Rust 与 CPython 之间的绑定，可以用于在 Rust 中编写原生的 CPython 扩展。包括 [`pydantic`](https://github.com/pydantic/pydantic-core)，[`tokenizers`](https://github.com/huggingface/tokenizers) 和 [`polars`](https://github.com/pola-rs/polars) 在内的多个包都使用了 PyO3。FFI [^1] 操作一定会有额外开销。那么，~~直接用 Python `class` 实现的类究竟有多慢呢~~用 PyO3 实现一个原生类型是否会比 Python 类更快吗？Python 类会创建 `__dict__` 以支持运行时的 monkey patching，调用类的成员函数时会有查询字典的开销，这个开销相比 FFI 调用又如何呢？

[^1]: Foreign Function Interface

> [!TLDR]
>
> *Python 确实有那么慢*。PyO3 实现的类可能确实比 Python 类更快。

## 实验

实验的动机是，[`Yixuan-Wang/apfel`](https://github.com/Yixuan-Wang/apfel) 尝试引入一个函数对象辅助类，用于给 Python 函数引入新运算符，但最核心的功能是需要重载 `()` 运算符，使之能像原来的函数一样被使用。大致的最小可用 Python 代码如下：

```python
class Func[**P, R]:
  f: Callable[P, R]
  
  def __init__(self, f: Callable[P, R]):
    self.f = f
  
  def __call__(self, *args: P.args, **kwargs: P.kwargs) -> R:
    return self.f(*args, **kwargs)
```

然后可以实现如下效果：

```python
@Func
def hello(name: str) -> str:
  return f"Hello, {name}!"

hello("world") # == "Hello, world!"
type(hello)    # is Func
```

之后，我们在 PyO3 中按照类似的组合思路（而非继承原生 `function` 对象）实现一个原生的类，它和 Python `class` 关键字定义的类功能几乎相同，主要区别是无法动态增删改查属性。

```rust
#[pyclass]
pub struct Func{
    f: Py<PyAny>,
}

#[pymethods]
impl Func {
    #[new]
    fn new(f: Py<PyAny>) -> Self {
        Func { f }
    }

    #[getter]
    fn f(&self) -> &Py<PyAny> {
        &self.f
    }

    #[pyo3(signature = (*args, **kwargs))]
    fn __call__(
        &self,
        py: Python<'_>,
        args: &Bound<'_, PyTuple>,
        kwargs: Option<&Bound<'_, PyDict>>,
    ) -> PyResult<Py<PyAny>> {
        self.f.call_bound(py, args, kwargs)
    }
}
```



## 结果

> [!TIP]
>
> ::p[PyO3 大勝，CPython 慘敗！這種速度，使人汗顏！如此速度，如何計算？我問了一下，我們 CPython 優化遠不如其他動態語言負責任，這個事情我們現在要正視，PSF 自己提出如何交待。]{lang=zh-Hant}

使用 `timeit` 标准库模块，循环执行 1e6 次函数调用，取 5 次平均值计算时间。

| 实现                            | 速度 (s) |
| ------------------------------- | -------- |
| 原函数                          | 0.068    |
| PyO3                            | 0.133    |
| Python `class`                  | 0.187    |
| Python `class` with `__slots__` | 0.187    |

PyO3 的 Rust 代码仍然交由 Python 调用了原函数，这部分开销是基准的。重载运算符在 PyO3 中引入了一倍于函数调用的开销，而 Python 类则引入了近乎两倍。即使引入了 `__slots__` 阻止创建 `__dict__` 也没有改善，毕竟重载需要查询的 `__call__` 是类的成员函数而不是字段。

那么，创建对象的开销呢？

| 实现                            | 速度 (s) |
| ------------------------------- | -------- |
| PyO3                            | 0.130    |
| Python `class`                  | 0.171    |
| Python `class` with `__slots__` | 0.165    |

仍然是 PyO3 更快。PyO3 只需要从 Python 读取一个指针存下即可，没有太多转换成 Rust 数据类型的开销，~~而我们 `class` 要考虑的可就多了~~。`__slots__` 能让 Python 类实例化过程快一些，但不多。

结论是，PyO3 实现的类（如果没有复杂的数据拷贝操作）确实可能比 Python 类要快。这当然不代表所有情况下使用机器原生扩展都更快——大致上来说，操作 Python 标准库定义的原生类型速度还是可观的。
