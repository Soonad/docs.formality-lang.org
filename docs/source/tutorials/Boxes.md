# Boxes

As explained previously, Formality includes primitives for performing deep copies of boxed terms:

syntax | effect
--- | ---
`#t` | Puts term `t` inside a box
`!T` | The type of a boxed term
`dup x = t; u` | Unboxes `t` and copies it as `x` inside `u`

The most important aspect to understand about boxes is how they're limited by the stratification condition. 

## The Stratification Condition

Deep copies, without restriction, would be very dangerous. For example, this program:

```javascript
main
  let f = {x}
    dup x = x
    x(#x)
  f(#f)
```

Would loop forever, which should never happen in a terminating language. To solve that, Formality relies on the **stratification condition**. It, in short, enforces the following invariant:

> The level a term can never change during the program evaluation.

Where the level of a term is the number of boxes (`#`s) wrapping it on the syntax tree. For example, here:

```javascript
["a", [#"b", #[#"c", ##"d"]]]
```

The string `"a"` isn't wrapped by any box. As such, it is on `level 0`. The string `"b"` is wrapped by one box, so it is on `level 1`. The entire `[#"c", ##"d"]` pair is wrapped by one box, so, `"c"` is wrapped by 2 boxes in total and, thus, is on `level 2`, while `"d"` is wrapped by 3 boxes in total and, thus, is on `level 3`. 

This condition forbids certain programs from being accepted. For example, this one:

```javascript
box : {x : Word} -> !Word 
  # x
```

Isn't allowed because, if it was, we would be able to increase the level of a word. Similarly, this one:

```javascript
main : [:Word, Word]
  dup x = #42
  [x, x]
```

Isn't allowed, because `42` would jump from `level 1` to `level 0` during runtime. But this one is fine:

```javascript
main : ![:Word, Word]
  dup x = #42
  # [x, x]
```

Because `42` remains on `level 1` after being copied. This one is fine too:

```javascript
main : [:!Word, !Word]
  dup x = #42
  [#x, #x]
```

For the same reason.

Because of this, boxes aren't really useful for copying data: information can only flow from lower to higher levels. That is, a term triggering a `dup` must be one level below the term copied, it won't be able to read the results of that copy. But boxes are very useful for implementing control structure: bounded loops, recursion, etc. In fact, Formality's recursive functions are actually desugared to an application of inductive Nats, which use boxes internally.

## Implementing Loops and Recursion

While Formality has a built-in syntax for recursion, it can be insightful to understand how it is implemented under the hoods. To understand it, mind the following program:

```javascript
ten_times : {~T : Type, f : !{x : T} -> T, x : !T} -> !T
  dup f = f
  dup x = x
  # f(f(f(f(f(f(f(f(f(f(x))))))))))

main : !Word
  ten_times(~Word, #{x} x + 2, #0)
```

Here, we define a function, `ten_times`, which takes a function, `f`, creates 10 copies of it, and applies to an argument, `x`. As the result, we're able to repeat the `+ 2` operation 10 times, adding `20` to `0`. This same technique can be used to implement bounded recursion. For example, here:

```javascript
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

We "emulate" a recursive function by using `ten_times` to "build" the recursion tree of "fact" up to 10 layers deep. As such, it only works for inputs up to 10; after that, it hits the "halt" case and returns 0. The good thing about this way of doing recursion is that we're not limited to recurse on structurally smaller arguments. The bad thing is that it is a little bit verbose, requiring an explicit bound, and a halting case for when the function "runs out of gas". Moreover, since we used `ten_times` to make the function, it comes inside a box, on `level 1`. In other words, it is impossible to use it on `level 0`! Instead, we must use the `level 0` to unbox it (with a `dup`), and then use it on `level 1`.

Fortunately, since bounded recursive functions are so common, Formality has built-in syntax for them, relying on "boxed definitions". To make a boxed definition, simply annotate it with a `!` instead of a `:`. That has several effects. First, the whole definition is lifted to `level 1`. Second, it allows you to use boxed definitions inside `<>`s: the parser will automatically unbox them for you. Third, the definition can call itself recursively; if it does, then Formality will assembling the recursion for you using a `Ind` implementation from the Base library. This is an example of usage:

```javascript
!fact*n : ! {i : Word} -> Word
  cpy i = i
  if i .= 0:
    1
  else:
    i * fact(i - 1)
  * 0
  
!main : !Word
  <fact*100>(12)
```

Notice that the halting case was defined with a `*` on the end. If it coincided with one of the function's arguments, you could place a `*` before it instead, i.e., `fact ! {*i : Word} -> Word`. Note also that, since `main` uses a recursive function, it must itself be a boxed definition, annotated with `!`. Alternatively, you could make it a normal definition and perform the unboxing yourself:

```javascript
main : !Word
  dup fact = fact*100
  # fact(12)
```

Both are equivalent.

The maximum recursion depth of the `Ind` defined by the base library is `2^256-1`. In other words, despite being terminating, in practice, Formality is capable of doing anything a Turing-complete language could do. After all, no language could perform more than `2^256-1` calls without exhausting your computer (or this universe's) resources.
