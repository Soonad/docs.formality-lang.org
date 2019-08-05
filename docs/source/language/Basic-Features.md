# Basic Features

## Let

Formality includes `let` expressions, which allow you to give names to expression.

```
main : String
  let hello = "Hello, world!"
  print(hello)
```

`let` expressions can be infinitely nested.

```
main : String
  let output = 
    let hello = "Hello, world!"
    print(hello)
  output
```

`let` performs parse-time substitution. This means it can't be used for sharing/copying, as will become clear later on.

## Words

Formality includes native, unsigned, 32-bit numbers, and many numeric operations:

name | syntax | JavaScript equivalent
--- | --- | ---
addition | `x + y` | `(x + y) >>> 0`
subtraction | `x - y` | `(x - y) >>> 0`
multiplication | `x * y` | `(x * y) >>> 0`
division | `x / y` | `(x / y) >>> 0`
modulus | `x % y` | `(x % y) >>> 0`
exponentiation | `x ** y` | `(x ** y) >>> 0`
bitwise-and | `x & y` | `x & y`
bitwise-or | `x | y` | `x | y`
bitwise-xor | `x ^ y` | `x ^ y`
bitwise-not | `x ~ y` | `~y`
bitwise-right-shift | `x >> y` | `x >>> y`
bitwise-left-shift | `x << y` | `x << y`
greater-than | `x > y` | `x > y ? 1 : 0`
less-than | `x < y` | `x < y ? 1 : 0`
equals | `x === y` | `x === y ? 1 : 0`

The type of a native number is `Word`. Note that there is no operator precedence. An expression like `3 * 10 + 1` is always parsed as `3 * (10 + 1)`. You can use parenthesis to change the order.

```javascript
main : Word
  (3 * 10) + 1
```

## Pairs

Formality includes native pairs.

syntax | description
--- | ---
`[x : A, B(x)]` | The type of a pair
`[a, b]` | Creates a pair with elements `a` and `b`
`fst(p)` | Extracts the first element of a pair
`snd(p)` | Extracts the second element of a pair
`get [a, b] = p ...` | Extracts both elements of a pair

Note that the type of a pair is `[x : A, B(x)]`, because the type of the second element can depend on the value of the first. When it doesn't, you can write just `[:A, B]` instead. Using pairs is straightforward.

- Creating, extracting one element with `fst`

```javascript
main : Word
  let pair = [1, 2]
  fst(pair)
```

- Creating, extracting both elements with `get`

```javascript
main : Word
  let pair  = [1, 2]
  get [a,b] = pair
  a + b
```

- Nesting

```javascript
main : [:Word, Word]
  let pairs = [[1, 2], [3, 4]]
  fst(pairs)
```


## If

`if` allows branching with a `Word` condition.

syntax | description
--- | ---
`if n p` | If `n === 0`, evaluates to `snd p`, else, evaluates to `fst p`
`then: a else: b` | Equivalent to `[a, b]`

Usage is straightforward:

```javascript
main : Output
  let age = 30

  if age < 18 then:
    print("boring teenager")
  else:
    print("respect your elders!")
```

## Cpy

Words can be copied with `cpy`:

```javascript
main : [:Word, Word]
  cpy x = 42
  [x, x + x]
```

Why this is necessary will become clear later on.

## Functions

Formality includes primitives for creating and applying functions:

syntax | effect
--- | ---
`{x : A, y : B, z : C, ...} -> D` | The type of a function with argumens `x : A`, `y : B`, `z : C`, returning a `D`
`{x, y, z, ...} body` | A function that receives the arguments `x y z` and returns `body` 
`f(x, y, z, ...)` | Applies the function `f` to the arguments `x`, `y`, `z` (curried)

Formality functions are anonymous expressions, like Haskell's lambdas. It has no multi-argument lambdas; `{x, y, z, ...} body` is just a syntax-sugar for `{x} {y} {z} ... body`. This is the same as JavaScript's `x => y => z => ... body`, or Haskell's `\ x y z ... -> body`. Function calls use the more conventional `f(x, y, z)` syntax, which, since functions are curried, is just a syntax-sugar for `f(x)(y)(z)...`. The type of a function is `{x : A, y : B, z : C ...} -> D`, which is equivalent to Agda's `(x : A) -> (y : B) -> (z : C) -> ... D`. As usual, function types can be shortened as `A -> B` when `x` is unused. When annotating a top-level definition with a functional type, you don't need to write the lambdas, they are added implicity. For example:

```javascript
birth_to_age : {birth : Word} -> Word
  2019 - birth

main : Word
  birth_to_age(1990)
```

Lambdas and applications can be erased with a `~`, which causes them to vanish from the compiled output. This is useful, for example, to write polymorphic functions without extra runtime costs. For example, this is the polymorphic identity function:

```javascript
id : {~T : Type, x : T} -> T
  x

main : Word
  id(~Word, 42)
```
