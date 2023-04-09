---
title: Curry-Howard Correspondence in Coq
date: 2023-04-05
category: comp
tags:
  - pl
  - logic
lang: en
keywords:
  - type-theory
  - coq
---

What is Curry-Howard Correspondence and how does the type system of Coq implement it.

<!-- more -->

## Curry-Howard Correspondence

Curry-Howard Correspondence draws relationship between the type theory (computer program) and the logics (proof). In short, ['a proof is a program, and the formula it proves is the type of the program'](https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence#:~:text=a%20proof%20is%20a%20program%2C%20and%20the%20formula%20it%20proves%20is%20the%20type%20for%20the%20program).

One can easily observe the similarity between formalizations in various fields. Take currying as an example:
$$
\begin{aligned}
((T \times U) \to V) &\Rightarrow (T \to U \to V) \\
((T \land U) \to V) &\Rightarrow (T \to U \to V) \\
\end{aligned}
$$
The first line is the formalization of currying in type theory, where $\times$ produces product / pair / tuple type, or `(T, U)`, and $\to$ denotes a function. The second line is the logic formalization, where $\land$ is the logical and operator and $\to$ is the material implication operator. The correspondence is obvious.

The $\to$ operation behaves like exponential. So a further and more unified abstraction could be:
$$
C^{BA} = {C ^ {B}} ^ {A}
$$

| Logic                                | Program                                                      |
| ------------------------------------ | ------------------------------------------------------------ |
| $\land$ (conjunction)                | $\times$ (product type, `tuple`s)                            |
| $\lor$ (disjunction)                 | $+$ (sum type, `enum`s)                                      |
| $\to$ (implication)                  | $\to$ (`function`s)                                          |
| $\forall$ (universal qualification)  | $\Pi$ (generalized product type)                             |
| $\exist$ (existential qualification) | $\Sigma$ (generalized sum type)                              |
| $\mathrm{T}$ (true)                  | ([unit type](https://en.wikipedia.org/wiki/Unit_type))       |
| $\mathrm{F}$ (false)                 | $\bot$ ([bottom type](https://en.wikipedia.org/wiki/Unit_type)) |

Generalized product and sum types sound a little bit odd. They are **dependent types**, which allows you to build types out of values. **Generalized product type**s are the type of functions that take *values* and return *types* that varies according to the value, while **generalized sum type**s are the type of tuples that the *type* of the second element varies according to the *value* of the first element (maybe you can treat the first element as an enum discriminant). With an extra order / generalization, the product types behave like functions (exponential-like), and the sum types behave like product ones.

## Coq's Type System

As a logic programming language, Coq (or Galina, to be exact) does incorporate logical propositions into its type system.

```
(value) -> (type) -> `Type`
                       ↑
(proof) -> (prop) -> `Prop`
```

In the graph above, `()` denotes a kind in general, and `->`  denotes 'has type of'. If something has type of `T`, then it is annotated with `:` and `T` on the righthand side in Coq.  `Type` and `Prop` are two specific types. 

## Types and Values

This is a definition of a type, `nat`.

```coq
Inductive nat : Type (*1*) := 
  | O (*2*)
  | S (*3*) (n : nat (*4*) ).
```

As a type, `nat` has the type of `Type`<sup>(1)</sup>.  This `nat` type has two contructors, `O`<sup>(2)</sup> and `S`<sup>(3)</sup>. The `O` constructor take no arguments, so a `O` constructor *constructs* a value of type `nat` directly, while the `S` constructor takes a `nat` as argument, but the return type does not depend on that argument, just a `nat`. So you can annotate it with `:` as follows:

```coq
Inductive nat : Type := 
  | O : nat
  | S (n : nat) : nat.
```

## Generics

A generic type, `list`, can be definined in a similar fashion.

```coq
Inductive list (X: Type) (*1*) : Type :=
  | nil : list X
  | cons (head: X) (tail: list X) : list X.
```

The generic type parameter is promoted to the lefthand side of `:`<sup>(1)</sup>, which turns the `list` identifier into a `Type -> Type` function. This parameter `X` is now shared between all constructors of `list`. You have to construct a `list` like:

```coq
Compute (cons nat 1 (nil nat)). (* [1] *)
```

## Propositions and Proofs

Then let's head over to the logics and proofs part. This is the definition of the `True` proposition. 

```coq
Inductive True : Prop (*1*) :=
  | I : True (*2*).
```

Just like the `nat` case, `True` has the type of `Prop`<sup>(1)</sup>, the type of all propositions, and it has one constructor `I` that constructs a `True` proposition with no parameters<sup>(2)</sup>. **Constructors of propositions build proofs**.

## Binding: Parameters and Indices

A more complex example is the `ev` property, a `Prop` dependent on a natural number.

```coq
Inductive ev : nat -> Prop (*1*) :=
  | ev_0 : ev 0 (*2*)
  | ev_SS (n : nat) (H : ev n (*3*)) : ev (S (S n)) (*4*).
```

Beware the type annotation of `ev`. Instead of using `(n: nat) : Prop`, which moves the natural number to the parameter side, the `nat` appears on the right side (the **index side**), `nat -> Prop`<sup>(1)</sup>. And the `ev_0` constructor returns the dependent type `ev 0`<sup>(2)</sup>, which is a narrowed type from `Prop`. 

What will happen if you use `(n: nat) : Prop`? Then `n` will be *bound*, instead of free. Check the following example:

```coq
Inductive ev (n: nat) : Prop :=
	| ev_0 : ev ... (*1*)
	(* ... *)
```

Now each `ev` binds its own, *specific* `n` provided within parameters, and that `n` can be a value other than 0. So you cannot call `ev 0` at mark `(*1*)`, because this inner `ev` call now only accepts this specific `n` as argument, or you will get a type mismatch. In contrary, `: nat -> Prop` introduces an arbitrary natural number as parameter, so with any specific `n0`, `ev n0` is compatible with this arbitrary `ev n`. The semantics of a `nat` parameter on `ev` is weird: we will have an infinite list of incompatible properties, even_2, even_4 and so on.

Thus **constructors can further narrow the type definition index side**. According to the type system, `ev_0` is a **proof**, just like `I`, and `ev 0` is a **proposition**, just like `True`. The`ev_SS` constructor has the type of `forall n: nat, ev n -> ev (S (S n))`. It takes a natural number and *a proof that `n` is even* <sup>(3)</sup> and returns a proof stating the proposition that `S (S n)` is even<sup>(4)</sup>.

We are not defining any concrete calculations (or **functions**) for the proof. Instead, we only describe the **relation**. Everything just happens in the type system. Notice that we are leveraging **dependent types** to narrow the type.

## Binding: Scope

In the `ev` example, we move `nat` from the parameter side to the index side to make it a **free** variable instead of a **bound** one. A  parameter defined on the type itself<sup>(1)</sup> has a scope that covers the whole `Inductive` definition. In Coq, you can introduce bound variables in inner scopes<sup>(2)</sup>. Take the definition of existence as an example: 

```coq
Inductive ex (A : Type) (P : A -> Prop) : Prop :=
	| ex_intro : forall x : A, P x -> ex A P.
```

In the `ex_intro` constructor, we introduced a binded variable `x` whose type is `A`. Its scope is limited to the constructor, the exist proposition itself does not depend on the value of `x`, but the proof does. 

The following definition is equivalent. It moves `x` from the index side to the parameter side. As the correspondence suggests, the universal qualification `forall (x: A), ...` is the same thing as the generalized product type `(x: A) -> ...`.

```coq
Inductive ex (A : Type) (P : A -> Prop) : Prop :=
	| ex_intro (x: A) : P x -> ex A P.
```

The following theorem in first order logic corresponds with the Coq definition of `ex`. $\mathcal{B}$ is a predicate whence $x$ doesn't occur freely. In the case of `ex_intro`, $\mathcal{B}$ is `ex A P`, while $\mathcal{A}$ is `P x`. The lefthand side is exactly our `ex_intro` constructor.

$$
\vdash (\forall x)(\mathcal{A}(x)\to \mathcal{B}) \leftrightarrow (\exist x)\mathcal{A}(x)\to \mathcal{B}
$$
