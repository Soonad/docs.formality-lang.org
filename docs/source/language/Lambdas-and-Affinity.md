## Lambdas

Formality includes 2 primitives for creating and applying functions:

syntax | effect
--- | ---
`{x y z} body` | Creates lambdas that receive the arguments `x y z` and return `body` 
`(f x y z)` | Applies the lambda `f` to the arguments `x`, `y`, `z` (curried)

## Currying

Lambdas are powerful because they allow us to create modular, reusable blocks of code. In Formality-Core, functions are called with the `(f x y z)` syntax rather than `f(x,y,z)`. To create them, you use the `{x y z} body` syntax, which is equivalent to JavaScript's `x => y => z => body`, or Python's `lambda x: lambda y: lambda z: body`. Formality doesn't have multi-argument lambdas, but you can emulate them with a sequence of lambdas:

```javascript
def sqr_dist: {a b}
  get [ax, ay] = a
  get [bx, by] = b
  |||bx - ax| ** 2| + ||by - ay| ** 2||

def main:
  let a = [2,2]
  let b = [5,6]
  ["Squared dist is: ", (sqr_dist a b)]
```

Here, a function of two arguments, `sqr_dist`, was emulated by using two lambdas of one argument, `{a}` and `{b}`. This means you can create partially applied functions, like in languages like Haskell. Unlike those, though, you can **not** dispense the parenthesis. For example, this is not correct:

```javascript
def func: {x y}
  |x + y|

def main:
  let num = func 1 2
  num
```

Because you need a parenthesis to call `func` on `1` and `2`, i.e., you must write it as `(func 1 2)`. 

## Affinity

A more notable difference is the fact that Formality's lambdas are affine, which means variables bound by lambdas can only be used, at most, once. For example, this is illegal:

```javascript
def square: {x}
  |x * x|

def main:
  (square 3)
```

After all, if this was allowed, the boxed duplication system would be redundant, as you could copy values with functions. This is a very important restriction, and perhaps the most important skill when writing Formality code is to figure out clever ways to circumvent this limitation. Here are 3 different ways to implement the program above:

```javascript
def square: {x}
  |x ** 2|

def square: {x}
  cpy x = x
  |x * x|

def square: {x}
  dup n = x
  # |n * n|
```

The first two are preferable because they avoid using boxes. A general rule is: *if you know statically the size of the input and the output of your function, you can implement it without boxes*. In the cases this is not possible such as when you deal with variable length structures (such as lists), boxed duplication is there to save the day. 

Despite affine lambdas being somewhat hard to work with, they're part of the reason Formality has so great computational characteristics. Moreover, they're not as restrictive as they seem. In fact, almost any non-terminating program that you'd use in practice can be written in Formality-Core with clever use of boxes (technically, any one that terminates in `2^(2^(2^...n))` steps, where `n` is the program size and the number of `2`s is equal to its depth).

[< Dups and Boxes](https://github.com/moonad/Formality/wiki/Dups-and-Boxes) | [Loop and Recursion >](https://github.com/moonad/Formality/wiki/Loops-and-Recursion)