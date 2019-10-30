# Boxes

As explained previously, Formality includes primitives for performing deep copies of boxed terms:

syntax | effect
--- | ---
`#t` | Puts term `t` inside a box
`!T` | The type of a boxed term
`dup x = t; u` | Unboxes `t` and copies it as `x` inside `u`

The `dup` primitive is the one responsible for copying and is extremelly important as it performs those copies lazily, in a way that allows optimal sharing of sub-expressions. It is what makes Formality a great closure evaluator. But, in order to use it properly, you must understand how it is limited: the stratification condition.

## The Stratification Condition

The primitives above, without restriction, would be dangerous. For example, this:

```haskell
main
  let f = {x}
    dup x = x
    x(#x)
  f(#f)
```

Would loop forever, which should never happen in a terminating language. To solve that, Formality relies on the **stratification condition**. In short, it enforces the following invariant:

> The level of a term can never change during the program evaluation.

Where the level of a term is the number of boxes "wrapping" it.

### Counting the level of a term

To understand the restriction above, you must be able to count the level of a term. Let's do it on the following example:

```haskell
["a", #"b", "c", #["d", #"e"], ##"f"]
```

- The string `"a"` isn't wrapped by any box. It is on `level 0`.

- The string `"b"` is wrapped by one box. It is on `level 1`.

- The string `"c"` isn't wrapped by any box. It is on `level 0`.

- The string `"d"` is wrapped by one box. It is on `level 1`.

- The string `"e"` is wrapped by two boxes (one indirect). It is on `level 2`. 

- The string `"f"` is wrapped by two boxes. It is on `level 2`. 

The type of the program above is:

```haskell
[:String, :!String, :String, :![:String, !String], !!String]
```

### Stratification examples

This condition is imposed globally, forbidding certain programs. For example:

```haskell
box : {x : Word} -> !Word 
  # x
```

This isn't allowed because, otherwise, we would be able to increase the level of a word. Similarly, this:

```haskell
main : [:Word, Word]
  dup x = #42
  [x, x]
```

Isn't allowed too, because `42` would jump from `level 1` to `level 0` during runtime. But this:

```haskell
main : ![:Word, Word]
  dup x = #42
  # [x, x]
```

Is fine, because `42` remains on `level 1` after being copied. And this:

```haskell
main : [:!Word, !Word]
  dup x = #42
  [#x, #x]
```

Is fine too, for the same reason.

## Applications

### Copying "static" data

In general of this, boxes aren't very useful for copying data. That's because information can only flow from lower to higher levels. So, for example, if some piece of data is generated on level `2`, you can copy it on level `3`, but you can't use it again on level `2`. Generally, your program's logic should stay on the highest level, with the lower levels being used to copy static data and generate bounded-depth recursive functions. In fact, Formality's syntax sugars and standard libraries are designed to be used with two levels only: the level `0`, where recursive functions are created and static data is duplicated, and the level 1, where everything is used. So, for example, the `Data.List/map` function on `Base` uses level `0` to make multiple copies of `f`, which are used on level `1`:

```haskell
#map*n : {~A : Type, ~B : Type, f : !A -> B} -> ! {case list : List(A)} -> List(B)
| cons => cons(~B, f(list.head), map(list.tail))
| nil  => nil(~B)
halt: nil(~B)
```

### Implementing loops/recursion

While Formality has a built-in syntax for recursion, it can be insightful to understand how it is implemented under the hoods. Mind the following program:

```haskell
ten_times : {~T : Type, f : !{x : T} -> T, x : !T} -> !T
  dup f = f
  dup x = x
  # f(f(f(f(f(f(f(f(f(f(x))))))))))

main : !Word
  ten_times(~Word, #{x} x + 2, #0)
```

Here, we define a function, `ten_times`, which takes a function, `f`, creates 10 copies of it, and applies to an argument, `x`. As the result, we're able to repeat the `+ 2` operation 10 times, adding `20` to `0`. This same technique can be used to implement bounded recursion. For example, here:

```haskell
ten_times : {~T : Type, f : !{x : T} -> T, x : !T} -> !T
  dup f = f
  dup x = x
  # f(f(f(f(f(f(f(f(f(f(x))))))))))

fact : !{n : Word} -> Word
  let call = {rec, i}
    cpy i = i
    if i .= 0:
      1
    else:
      (i * rec(i - 1))
  let halt = {i}
    0
  ten_times(~{x : Word} -> Word, #call, #halt)

main : !Word
  dup fact = fact
  # fact(6)
```

We "emulate" a recursive function by using `ten_times` to "build" the recursion tree of "fact" up to 10 layers deep. As such, it only works for inputs up to 10; after that, it hits the "halt" case and returns 0. The good thing about this way of doing recursion is that we're not limited to recurse on structurally smaller arguments. The bad thing is that it is a little bit verbose, requiring an explicit bound, and a halting case for when the function "runs out of gas". Moreover, since we used `ten_times` to make the function, it comes inside a box, on `level 1`. In other words, it is impossible to use it on `level 0`! Instead, we must use the `level 0` to unbox it (with a `dup`), and then use it on `level 1`. As usual, you could simplify it with a boxed definition:

```haskell
#main : !Word
  ($fact)(6)
```

Formality's recursion syntax builds a similar program, except: 

1. Instead of a hard-coded max call limit, it is configurable `*N`.

2. Instead of simple repetition, it uses `Nat` induction, allowing you to use the call count, `N`, in types.

So, for example, when you write:

```haskell
#fact*N : ! {i : Word} -> Word
  if i .= 0:
    1
  else:
    i * fact(i - 1)
halt: 0
```

It gives you a `fact : {N : Ind} -> !{n : Word} -> Word`, instead of a `fact : !{n : Word} -> Word`. You can call it inside a boxed definition with `($fact*MAX_CALLS)(x)`.
