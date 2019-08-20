## Box System

Formality's approach to termination is what makes it different from other proof languages like Agda, Idris and Coq. Instead of having native datatypes, structural recursion and so on, we go deeper and change the underlying logic of the system from intuitionist to elementary affine. This is responsible for all the claimed benefits of Formality: optimal reductions, no garbage-collection, massive parallelism, elegant inductive types and so on. But it comes with a huge tradeoff: our lambdas are affine, i.e., bound variables can't be used more than once. This limits what we can do in multiple ways. For example, we can't write a function that multiples a `Word` by itself:

```javascript
square : {x : Word} -> Word
  x * x

main : Word
  square(7)
```

If we try, we get an error:

```javascript
Lambda variable `x` used more than once in:
{x} => (x * x)
```

This can be amended in multiple ways.

### Avoid making a copy.

Suppose we were trying to implement this function:

```javascript
inc_if : {b : Bool, x : Word} -> Word
  @Bool b ~> Word
  | true  => x + 1
  | false => x + 0

main : Word
  inc_if(true, 10)
```

Here, `inc_if` increments a number if a boolean is true. Since `x` is used in different branches, we could avoid copying it by adding an extra lambda:

```javascript
inc_if : {b : Bool, x : Word} -> Word
  (@ b    ~> {x : Word} -> Word
  | true  => {x} x + 1
  | false => {x} x + 0)(x)

main : Word
  inc_if(true, 10)
```

That is, instead of using `x` inside each case of the pattern-match, we return a function that receives `x` and use it in the body of the function.

### Make a manual copy.

For `Word`s in particular, there is a native `cpy` operation that copies it as many times as desired:

```javascript
square : {x : Word} -> Word
  cpy x = x
  x * x

main : Word
  square(7)
```

For everything else, you can write an auxiliary `copy` function:

```javascript
copy : {b : Bool} -> [:Bool, Bool]
  @ b     => [:Bool, Bool]
  | true  => [true, true]
  | false => [false, false]

and_itself : {b : Bool} -> Bool
  get [b0, b1] = copy(b)
  and(b0, b1)
```

### Use boxes

Formality has a primitive for deep-copying values, boxes. 

syntax | description
--- | ---
`# a` | Puts the closed term `a` inside a box
`! A` | The type of a boxed term
`dup x = #a ...` | Removes `a` from its box and makes `n` copies of it as `x`

But this primitive, without restriction, would be very dangerous. For example, this program:

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

Boxes aren't very useful for copying data, because information can only flow from lower to higher levels. Since a term triggering a `dup` must be one level below the term copied, it won't be able to read the results of that copy. But they are very useful for implementing things like (bounded) loops and recursion, without causing the language to be non-terminating.

### Loops and Recursion

To show how bounded loops and recursion can be implemented with boxes, observe the following program:

```javascript
ten_times : {~T : Type, f : !{x : T} -> T, x : !T} -> !T
  dup f = f
  dup x = x
  # f(f(f(f(f(f(f(f(f(f(x))))))))))

main : !Word
  ten_times<Word>(#{x} x + 2, #0)
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
    if i === 0 then:
      1
    else:
      (i * rec(i - 1))
  let halt = {i}
    0
  ten_times<{x : Word} -> Word>(#call, #halt)

main : !Word
  dup fact = fact
  # fact(6)
```

We "emulate" a recursive function by using `ten_times` to "build" the recursion tree of "fact" up to 10 layers deep. As such, it only works for inputs up to 10; after that, it hits the "halt" case and returns 0. The good thing about this way of doing recursion is that we're not limited to recurse on structurally smaller arguments. The bad thing is that it is a little bit verbose, requiring an explicit bound, and a halting case for when the function "runs out of gas". Moreover, since we used `ten_times` to make the function, it comes inside a box, on `level 1`. In other words, it is impossible to use it on `level 0`! Instead, we must use the `level 0` to unbox it (with a `dup`), and then use it on `level 1`.

Fortunately, since bounded recursive functions are so common, Formality has built-in syntax for them, relying on "boxed definitions". To make a boxed definition, simply annotate it with a `!` instead of a `:`. That has several effects. First, the whole definition is lifted to `level 1`. Second, if it uses any boxed definition inside it, it is automatically unboxed. Third, the definition can call itself recursively; if it does, then Formality will assembling the recursion for you using a `Rec` implementation from the Base library. This is an example of usage:

```javascript
fact ! {i : Word} -> Word
  cpy i = i
  if i === 0 then:
    1
  else:
    i * fact(i - 1)
  & 0
  
main ! Word
  fact(12)
```

Notice that the halting case was defined with a `&` on the end. If it coincided with one of the function's arguments, you could place a `&` before it instead, i.e., `fact ! {&i : Word} -> Word`. Note also that, since `main` uses a recursive function, it must itself be a boxed definition, annotated with `!`. Alternatively, you could make it a normal definition and perform the unboxing yourself:

```javascript
main : !Word
  dup fact = fact
  # fact(12)
```

Both are equivalent.

The maximum recursion depth of the `Rec` defined by the base library is `2^64`. In other words, despite being terminating, in practice, Formality is capable of doing anything a Turing-complete language could do. After all, no language could perform more than `2^64` calls without exhausting your computer resources.
