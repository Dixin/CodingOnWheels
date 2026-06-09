---
title: "Lambda Calculus via C# (19) Church Encoding and More"
published: 2018-11-19
description: "So far a ton has been encoded. Here is a summary."
image: ""
tags: [".NET", ".NET Core", ".NET Standard", "C#", "LINQ"]
category: "C#"
draft: false
lang: "en"
---

> [!TIP]
> [Functional Programming and LINQ via C#](/posts/linq-via-csharp) Series
>
> [Lambda Calculus via C#](/archive/?tag=Lambda%20Calculus) Series

So far a ton has been encoded. Here is a summary.

## Summary of church encoding

### [Boolean](/posts/lambda-calculus-via-c-sharp-4-encoding-church-booleans)

```csharp
True := λt.λf.t
False := λt.λf.f
```

### [Boolean logic](/posts/lambda-calculus-via-c-sharp-5-boolean-logic)

```csharp
And :=  λa.λb.a b False
Or :=  λa.λb.a True b
Not := λb.b False True
Xor := λa.λb.a (b False True) (b True False)
```

### [If logic](/posts/lambda-calculus-via-c-sharp-6-if-logic-and-reduction-strategies)

```csharp
If := λc.λt.λf.c t f (λx.x)
```

### [Numeral](/posts/lambda-calculus-via-c-sharp-7-encoding-church-numerals)

```csharp
0 := λfx.x                  ≡ λf.λx.x                   ≡ λf.λx.f0 x
1 := λfx.f x                ≡ λf.λx.f x                 ≡ λf.λx.f1 x
2 := λfx.f (f x)            ≡ λf.λx.(f ∘ f) x           ≡ λf.λx.f2 x
3 := λfx.f (f (f x))        ≡ λf.λx.(f ∘ f ∘ f) x       ≡ λf.λx.f3 x
...
n := λfx.f (f ... (f x)...) ≡ λf.λx.(f ∘ f ∘ ... ∘ f) x ≡ λf.λx.fn x
```

### [Arithmetic](/posts/lambda-calculus-via-c-sharp-9-wrapping-church-numerals-and-arithmetic)

```csharp
Increase := λn.λf.λx.f (n f x)
Increase2 := λn.λf.f ∘ (n f)

Add := λa.λb.λf.λx.a f (b f x)
Add2 := λa.λb.λf.fa ∘ fb ≡ λa.λb.λf.(a f) ∘ (b f)
Add3 := λa.λb.a Increase b

Decrease := λn.λf.λx.n (λg.λh.h (g f)) (λu.x) (λu.u)
Decrease2 := λn.Item1 (n (Shift Increase) (CreateTuple 0 0))

Subtract := λa.λb.b Decrease a

Multiply := λa.λb.a (λx.Add b x) 0

_DivideBy := λa.λb.If (IsGreaterOrEqual a b) (λx.Add One (_DivideBy (Subtract a b) b)) (λx.Zero)
DivideByIgnoreZero = λa.λb.If (IsZero b) (λx.0) (λx._DivideBy a b)

Pow := λm.λ e.e (λx.Multiply m x) 1
```

A better [DivideBy will be re-implemented after introducing Y combinator](/posts/lambda-calculus-via-c-sharp-23-y-combinator-and-divide):

```csharp
DivideBy := Y (λf.λa.λb.If (IsGreaterOrEqual a b) (λx.Add One (f (Subtract a b) b)) (λx.Zero))
          ≡ (λf.(λx.f (x x)) (λx.f (x x))) (λf.λa.λb.If (IsGreaterOrEqual a b) (λx.Add One (f (Subtract a b) b)) (λx.Zero))
```

So DivideByIgnoreZero can by redefined using DivideBy instead of `_DivideBy`:

```csharp
DivideByIgnoreZero = λa.λb.If (IsZero b) (λx.0) (λx.DivideBy a b)
```

### [Predicate](/posts/lambda-calculus-via-c-sharp-11-predicates-and-divide)

```csharp
IsZero := λn.n (λx.False) True
```

### [Comparison](/posts/lambda-calculus-via-c-sharp-12-church-numeral-comparison-operators)

```csharp
IsLessOrEqual := λa.λb.IsZero (Subtract a b)
IsGreaterOrEqual := λa.λb.IsZero (Subtract b a)

IsEqual := λa.λb.And (IsLessOrEqual a b) (IsGreaterOrEqual a b)

IsLess := λa.λb.Not (IsGreaterOrEqual a b)
IsGreater := λa.λb.Not (IsLessOrEqual a b)
IsNotEqual := λa.λb.Not (IsEqual a b)
```

### [Pair (2-tuple)](/posts/lambda-calculus-via-c-sharp-13-encoding-church-pairs-2-tuples-and-generic-church-booleans)

```csharp
CreateTuple := λx.λy.λf.f x y
Tuple := λf.f x y

Item1 := λt.t True
Item2 := λt.t False

Shift := λf.λt.CreateTuple (Item2 t) (f (Item1 t))
Swap := λt.CreateTuple (Item2 t) (Item1 t)
```

### List

### [1 pair for each node, and null](/posts/lambda-calculus-via-c-sharp-15-encoding-church-list-with-church-pair-and-null)

```csharp
CreateListNode := CreateTuple ≡ λv.λn.λf.f v n

Value := Item1 ≡ λl.l (λv.λn.v)
Next := Item2 ≡ λl.l (λv.λn.n)

Null := False
IsNull := λl.l (λv.λn.λx.False) True

Index := λl.λi.i Next l
```

### [2 pairs for each node, and null](/posts/lambda-calculus-via-c-sharp-16-encoding-church-list-with-2-church-pairs-as-a-node)

```csharp
CreateListNode2 := λv.λn.CreateTuple False (CreateTuple v n)

Value2 := λl.Item1 (Item2 l)
Next2 := λl.If (IsNull2 l) (λx.l) (λx.(Item2 (Item2 l)))

Null2 := λf.True
IsNull2 := λl.(Item1 l)

Index2 := λl.λi.i Next2 l
```

### [Fold (aggregate) function for each node, and null](/posts/lambda-calculus-via-c-sharp-17-encoding-church-list-with-fold-aggregate-function)

```csharp
CreateListNode3 := λv.λn.λf.λx.f v (n f x)

Value3 := λl.λx.l (λv.λy.v) x
Next3 := λl.Item2  (l (λv.λt.ShiftTuple (CreateListNode3 v)) (CreateTuple Null3 Null3))

Null3 := λf.λx.x
IsNull3 := λl.l (λv.λx.False) True

Index3 := λl.λi.i Next3 l
```

### [Signed number](/posts/lambda-calculus-via-c-sharp-18-encoding-signed-number)

```csharp
Signed := Tuple
ToSigned := λn.CreateTuple n 0
Negate := Swap

Positive := Item1
Negative := Item2

FormatWithZero := λs.If (IsEqual sp  sn) (λx.ToSigned 0) (λx.If (IsGreater sp sn) (λy.ToSigned (Subtract sp sn)) (λy.Negate (ToSigned (Subtract sn sp))))
```

### Arithmetic

```csharp
AddSigned := λa.λb.FormatWithZero (CreateTuple (Add ap bp) (Add an bn))

SubtractSigned := λa.λb.FormatWithZero (CreateTuple (Add ap bn) (Add an bp))

MultiplySigned := λa.λb.FormatWithZero (CreateTuple (Add (Multiply ap bp) (Multiply an bn)) (Add (Multiply ap bn) (Multiply an bp)))

DivideBySigned := λa.λb.FormatWithZero (CreateTuple (Add (DivideByIgnoreZero ap bp) + (DivideByIgnoreZero an bn)) (Add (DivideByIgnoreZero ap bn) (DivideByIgnoreZero an bp))))
```

## Encode, encode, and encode

### From signed number to complex integer and rational number

With signed number, [complex integer](http://en.wikipedia.org/wiki/Gaussian_integer) can be [encoded](http://cs.stackexchange.com/questions/2272/representing-negative-and-complex-numbers-using-lambda-calculus) by a Church pair of signed numbers: $(s_{real}, s_{imaginary})$, which represents complex integer $z = s_{real} + s_{imaginary} * i$.

With signed number, [rational number](http://en.wikipedia.org/wiki/Rational_number) can also be encoded by a Church pair of a signed number and a Church numeral: $(s_{numerator}, n_{denominator})$, which represents rational number:

$$q = \frac{s_{numerator}}{1 + n_{denominator}}$$

[Dyadic rational number](http://en.wikipedia.org/wiki/Dyadic_rational) can be encoded by $(s_{numerator}, n_{exponent})$ as well, which represents:

$$d = \frac{s_{numerator}}{2 ^ {n_{exponent}}}$$

### From rational number to real number and complex number

Then with rational number, a [real number](http://en.wikipedia.org/wiki/Real_number) r can be [encoded](https://users.dimi.uniud.it/~pietro.digianantonio/papers/copy_pdf/thesis.pdf) in many different ways:

-   r can be represented by a sequence of Church pair of 2 rational numbers $p_{0} = (q_{0}, q_{0}')$, $p_{1} = (q_{1}, q_{1}')$, $p_{2} = (q_{2}, q_{2}')$, …, such that:
    -   $p_{n}$ represents a rational interval, since $q_{n}$ and $q_{n}'$ are both rational numbers.
    -   $p_{n + 1} \subseteq p_{n}$
    -   $\lim \limits_{n \to \infty} q_{n}' - q_{n} = 0$
    -   $r = \bigcap_{n ∈ N}p_{n}$
-   r can be represented by a [Cauchy sequence](http://en.wikipedia.org/wiki/Cauchy_sequence) of rational numbers q0, q1, q2, …, and a function f of type `Func<_Numeral, _Numeral>`, defining the convergence rate of the Cauchy sequence such that:
    -   $\forall i.j.k. | q_{f(i) + j} - q_{f(i) + k} | ≤ 2 ^ {-i}$
    -   $r = \lim \limits_{n \to \infty} q_{n}$
-   r can be represented by a Cauchy sequence of rational numbers q0, q1, q2, … with a fixed convergence rate, such that:
    -   $\forall i.j. | q_{i} - q_{i + j} | ≤ \frac{1}{i}$
    -   $r = \lim \limits_{n \to \infty} q_{n}$

etc.. An example in [Haskell](http://en.wikipedia.org/wiki/Haskell_\(programming_language\)) can be found [on Github](https://github.com/andrejbauer/marshall/blob/master/etc/haskell/Reals.hs).

With real number, [complex number](http://en.wikipedia.org/wiki/Complex_number) can be naturally encoded by a Church pair of 2 real numbers $(r_{real}, r_{imaginary})$, which represents complex number $z = r_{real} + r_{imaginary} * i$.

### And much more

Church pair can encode more complex data structures, like tree.

Church List can encode string.

Church Tuple and Church List can encode more complex algebra types.

…

Don’t worry. The encoding stops here. All the above data types and functions demonstrate that any data type or calculation may be encoded in lambda calculus. This is the [Church-Turing thesis](http://en.wikipedia.org/wiki/Church-Turing_thesis).
