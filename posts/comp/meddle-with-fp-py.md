---
title: Meddling with Functional Programming in Python
date: 2023-10-26
category: comp
tags:
  - pl
  - python
keywords:
  - fp
---

Python is not a functional programming language.

<!-- more -->

But I do want it to be one!

## Why Functional?

Why functional? Because functional codes can be more intuitive to write and easier to read, and it might enable better performance in some cases (like you don't need to `append` to a list and pay for the possible copy).

Of course, sometimes imperative codes are better. But in Python you can always be imperative anyway.

Python can do functional stuff. However, as it is Object Oriented down to the core, and the typical Pythonic code doesn't involve much functional programming, common functional patterns require tedious boilerplates to implement.

## Problems of Pythonic

There are plenty of functional packages out there, and even some functional tools inside the standard library. However, they just don't quite fit my preference.

Standard [`functools`](https://docs.python.org/3/library/functools.html) and [`toolz`](https://github.com/pytoolz/toolz) provides some higher-level functional tools, but it is too Pythonic. Why it is not good to be Pythonic? Python encourages data to flow from right to left, from inside to outside! For Haskell, you have `.` and `$` to compose functions, but you don't have that in your pocket writing Python.

```python
a = list(map(func, filter(pred, iterable)))
#                               ^^^^^^^^ data enters here
#                  <----- through filter
#        <-- through map
#   <--- collect into a list   

# First you type your data
iterable

# And then you realize you need to wrap it inside a filter
filter(pred, iterable # So you move left to write the filter
) # And you move right to write the right paren!

# And then you realize you need to wrap it inside a map
# Oh my. Another round.
# ...
```

You might say we have list comprehensions. Looks better, but now we jump back and forth!

```python
a = [func(x) for x in iterable if pred(x)]
#                     ^^^^^^^^ data enters here
#               through filter --------->
#            <---------- then jump back here
#   <--- through map
# <----- collect into a list
```

Well, in Haskell the data also comes after the function. But its unique syntax and operator `.` and `$` will at least save you from wrapping everything in a hell of parentheses. If you want it from left to right, no problem, Haskell has `&` for you.

```haskell
-- Haskell
a = map func . filter pred $ iterable
--                           ^^^^^^^^ data enters here
--             <----- through filter
--  <-- through map

a = iterable & filter pred & map func
--  ^^^^^^^^ data enters here
--             -----> through filter
--                           --> through map
```

In Rust, you can chain methods on iterators. But Python doesn't give you anything on its iterators.

```rust
// Rust
let a = iterable
        .filter(pred)
        .map(func)
        .collect::<Vec<_>>();
```

That's also possible in JavaScript `Array`s, and in the future will be also available in the new `Iterator`s.

```javascript
// JavaScript
const a = iterable
          .filter(pred)
          .map(func);
```

## Problems of Static Type Checking

[`PyFunctional`](https://github.com/EntilZha/PyFunctional) provides idiomatic chainable functional operations, but it is too dynamic. It is **not typed**. `Unknown` will pollute the whole logic chain, and functional programming without static type checking is more error-prone, especially when composing functions.

Recent advancement on `typing` library, like `ParamSpec`, really makes a typed functional library possible.
