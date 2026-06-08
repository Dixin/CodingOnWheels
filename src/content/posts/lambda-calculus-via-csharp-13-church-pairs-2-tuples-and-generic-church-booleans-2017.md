---
title: "Lambda Calculus via C# (13) Encoding Church Pairs (2-Tuples) and Generic Church Booleans"
published: 2018-11-13
description: "is the Church encoding of the  type, aka 2-"
image: ""
tags: [".NET", ".NET Core", ".NET Standard", "C#", "LINQ"]
category: ".NET"
draft: false
lang: ""
---

> [!TIP]
> [Functional Programming and LINQ via C#](/posts/linq-via-csharp) Series
>
> [Lambda Calculus via C#](/archive/?tag=Lambda%20Calculus) Series
>
> This post is updated, [here is the latest version](/posts/lambda-calculus-via-csharp-4-tuple-and-signed-numeral).

[Church pair](http://en.wikipedia.org/wiki/Church_encoding#Church_pairs) is the Church encoding of the [pair](http://en.wikipedia.org/wiki/Cons) type, aka 2-[tuple](http://en.wikipedia.org/wiki/Tuple). Unlike the [Tuple<T1, T2>](https://msdn.microsoft.com/en-us/library/dd268536.aspx) class in .NET, in lambda calculus Church pair will be represented by lambda expression. To avoid 2 naming systems, here in all the code, Church pair will be called tuple.

## Church pair (2-tuple)

A Church pair can be constructed with 2 values x y:

```csharp
CreateTuple := λx.λy.λf.f x y
```

And it return a tuple - another lambda expression (λf.f x y). So tuple is a higher order function that takes a function and apply it with x and y.

```csharp
Tuple := λf.f x y
```

Notice:

-   tuple is a closure of x and y
-   f is supposed to be in the format of λx.λy.E

So, to get the first item x, a f like λx.λy.x can be applied to a tuple.

```csharp
Item1 := λt.t (λx.λy.x)
```

Item1 takes a tuple as parameter, applies it with a (λx.λy.x), and returns the first item x. This is how Item1 works:

```csharp
Item1 (CreateTuple x y)
≡ Item1 (λf.f x y)
≡ (λt.t (λx.λy.x)) (λf.f x y)
≡ (λf.f x y) (λx.λy.x)
≡ (λx.λy.x) x y
≡ (λy.x) y
≡ x
```

So to get the second item y, a tuple can be applied with a f of λx.λy.y:

```csharp
Item2 := λt.t (λx.λy.y)
```

And just like Item1:

```csharp
Item2 (CreateTuple x y)
≡ Item2 (λf.f x y)
≡ (λt.t (λx.λy.y)) (λf.f x y)
≡ (λf.f x y) (λx.λy.y)
≡ (λx.λy.y) x y
≡ (λy.y) y
≡ y
```

Based on above definitions, here is the C# implementation:

```csharp
// Tuple = f => f(item1)(item1)
public delegate object Tuple<out T1, out T2>(Func<T1, Func<T2, object>> f);
// Tuple is an alias of Func<Func<T1, Func<T2, object>>, object>

public static class ChurchTuple
{
    // CreateTuple = item1 => item2 => f => f(item1)(item2)
    public static Func<T2, Tuple<T1, T2>> Create<T1, T2>
        (T1 item1) => item2 => f => f(item1)(item2);

    // Item1 => tuple => tuple(x => y => x)
    public static T1 Item1<T1, T2>
        (this Tuple<T1, T2> tuple) => (T1)tuple(x => y => x);

    // Item2 => tuple => tuple(x => y => y)
    public static T2 Item2<T1, T2>
        (this Tuple<T1, T2> tuple) => (T2)tuple(x => y => y);
}
```

Tuple’s Item1 is of type T1, Item2 is of type T2. And, f is λx.λy.E, so its type is Func<T1, Func<T2, object>>. Again, just like the object in Church Boolean Func<object, Func<object, object>>, object here does not mean System.Object is introduced. It just mean λx.λy.E can return any type. For example:

-   in function Item1, f is λx.λy.x or x => y => x, so f returns a T1
-   in function Item2, f is λx.λy.y or x => y => y, so f returns a T2

## Generic Church Booleans

If observing above definition:

```csharp
Item1 := λt.t (λx.λy.x)
Item2 := λt.t (λx.λy.y)
```

In Item1 f is actually True, and in Item2 f becomes False. So above definition can be simplified to:

```csharp
Item1 := λt.t True
Item2 := λt.t False
```

In C# more work need to be done for this substitution. As fore mentioned, f is Func<T1, Func<T2, object>> but currently implemented [Church Boolean](/posts/lambda-calculus-via-c-sharp-4-encoding-church-booleans) is Func<object, Func<object, object>>. So a more specific Church Boolean is needed.

```csharp
// Curried from: object Boolean(TTrue @true, TFalse @TFalse)
public delegate Func<TFalse, object> Boolean<in TTrue, in TFalse>(TTrue @true);
// Boolean is alias of Func<TTrue, Func<TFalse, object>>

public static partial class ChurchBoolean
{
    // True = @true => @false => @true
    public static Func<TFalse, object> True<TTrue, TFalse>
        (TTrue @true) => @false => @true;

    // False = @true => @false => @false
    public static Func<TFalse, object> False<TTrue, TFalse>
        (TTrue @true) => @false => @false;
}
```

With this generic version of Church Booleans, above Church tuple can be re-implemented:

```csharp
public delegate object Tuple<out T1, out T2>(Boolean<T1, T2> f);

public static partial class ChurchTuple
{
    // CreateTuple = item1 => item2 => f => f(item1)(item2)
    public static Func<T2, Tuple<T1, T2>> Create<T1, T2>
        (T1 item1) => item2 => f => f(item1)(item2);

    // Item1 = tuple => tuple(x => y => x)
    public static T1 Item1<T1, T2>
        (this Tuple<T1, T2> tuple) => (T1)tuple(ChurchBoolean.True<T1, T2>);

    // Item2 = tuple => tuple(x => y => y)
    public static T2 Item2<T1, T2>
        (this Tuple<T1, T2> tuple) => (T2)tuple(ChurchBoolean.False<T1, T2>);
}
```

### Back to Church Boolean - why not using generic Church Booleans from the beginning?

If the [Boolean logic](/posts/lambda-calculus-via-c-sharp-5-boolean-logic) is implemented with this generic version of Church Booleans, then:

```csharp
public static partial class ChurchBoolean
{
    // And = a => b => a(b)(False)
    public static Boolean<TTrue, TFalse> And<TTrue, TFalse>
        (this Boolean<Boolean<TTrue, TFalse>, Boolean<TTrue, TFalse>> a, Boolean<TTrue, TFalse> b) => 
            (Boolean<TTrue, TFalse>)a(b)(False<TTrue, TFalse>);

    // Or = a => b => a(True)(b)
    public static Boolean<TTrue, TFalse> Or<TTrue, TFalse>
        (this Boolean<Boolean<TTrue, TFalse>, Boolean<TTrue, TFalse>> a, Boolean<TTrue, TFalse> b) => 
            (Boolean<TTrue, TFalse>)a(True<TTrue, TFalse>)(b);

    // Not = boolean => boolean(False)(True)
    public static Boolean<TTrue, TFalse> Not<TTrue, TFalse>
        (this Boolean<Boolean<TTrue, TFalse>, Boolean<TTrue, TFalse>> boolean) => 
            (Boolean<TTrue, TFalse>)boolean(False<TTrue, TFalse>)(True<TTrue, TFalse>);

    // Xor = a => b => a(b(False)(True))(b(True)(False))
    public static Boolean<TTrue, TFalse> Xor<TTrue, TFalse>
        (this Boolean<Boolean<TTrue, TFalse>, Boolean<TTrue, TFalse>> a, Boolean<Boolean<TTrue, TFalse>, Boolean<TTrue, TFalse>> b) => 
            (Boolean<TTrue, TFalse>)a((Boolean<TTrue, TFalse>)b(False<TTrue, TFalse>)(True<TTrue, TFalse>))((Boolean<TTrue, TFalse>)b(True<TTrue, TFalse>)(False<TTrue, TFalse>));
}
```

The type parameter becomes too noisy. It is difficult to read or use these functions.

## Currying and type inference

The [part of currying](/posts/lambda-calculus-via-c-sharp-1-fundamentals-closure-currying-and-partial-application) mentioned currying may cause some noise for [type inference in C#](/posts/understanding-csharp-3-0-features-3-type-inference). Here is an example:

```csharp
Swap = λt.CreateTuple (Item2 t) (Item1 t)
```

C# logic is simple, but the type information has to be given so it is noisy:

```csharp
// Swap = tuple => Create(tuple.Item2())(tuple.Item1())
public static Tuple<T2, T1> Swap<T1, T2>
    (this Tuple<T1, T2> tuple) => Create<T2, T1>(tuple.Item2())(tuple.Item1());
```

When invoking the curried Create function, the type arguments cannot be omitted. This is signature of Create:

```csharp
Func<T2, Tuple<T1, T2>> Create<T1, T2>(T1 item1)
```

After currying, T2’s appearances are all relocated to Create’s returned type. So during the 2 applications of Create(item1)(item2), C# compiler does not even know how to compile first application Create(item1). It cannot infer what return type is wanted. The application code will always end up as:

```csharp
ChurchTuple.Create<int, string>(1)("a");
```

So, only for convenience of C# coding and less noise for readability, this [uncurried](/posts/lambda-calculus-via-c-sharp-1-fundamentals-closure-currying-and-partial-application) helper method can be created:

```csharp
public static Tuple<T1, T2> _Create<T1, T2>
    (T1 item1, T2 item2) => Create<T1, T2>(item1)(item2);
```

Now T2 is relocated back to parameter, so type arguments are not mandatory:

```csharp
ChurchTuple._Create(1, "a");
```

Much less noise. `_Create` is also tagged with underscore since its uncurrying is for adapting C# type inference feature.
