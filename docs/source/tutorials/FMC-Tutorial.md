# Formality-Core Tutorial

This tutorial aims to teach how to effectively develop Formality-Core code,
assuming experience in functional programming languages, in special Haskell.
Formality-Core is a minimal language based on elementary affine logic, making it
compatible with optimal reductions. It can be seen as the GHC Core to the
upcoming Formality language. Its minimalism, the lack of a type system, and its
unusual boxed duplication system make it a very bare-bones language that isn't
easy to work with directly, demanding that the programmer learns some delicate
techniques to be productive.

## 1. Core features

Before proceeding, you should have Formality
[installed](../language/Installation.md), and be familiar with its core features
and syntax. If you're not yet, please read the entire [Language](../Why.md)
section of this documentation. 

## 2. Simple datatypes

Algebraic Datatypes (ADTs) are the building bricks of functional programming
languages like Haskell, where all programs are just functions operating on ADTs.
Programming in Formality isn't fundamentally different. As such, the best way to
learn it is by learning how to translate Haskell code. 

Let's start with a simple type: booleans. In Haskell, they can be defined with
the following declaration:

```haskell
{-# LANGUAGE NoImplicitPrelude #-}

data Bool
  = True
  | False
```

This puts 2 constructors, `True` and `False` in scope. In Formality, there is no
`data` syntax, instead we use `T`:

```javascript
T Bool
| True
| False
```

Let's attempt to translate a simple function, `not`:

```haskell
not :: Bool -> Bool
not True  = False
not False = True
```

The first thing we must do is get rid of Haskell's equational notation (which
Formality-Core doesn't have) in favor of lambdas and case-ofs, like this:

```haskell
not :: Bool -> Bool
not = \ a -> case a of {
  True  -> False;
  False -> True;
}
```

For the sake of clarity, it is also recommended that each case is given a name
with a `let`, as follows:

```haskell
not :: Bool -> Bool
not = \ a ->
  let case_True  = False in
  let case_False = True in
  (case a of { True -> case_True; False -> case_False })
```

Once a Haskell program is in this shape, translating it to Formality is
straigthforward: we just have to adjust the syntax.

```javascript
not : {x : Bool} -> Bool
  case<Bool> x
  | True  => False
  | False => True
  : Bool
```

Run the program below with `fm <file_name>.main`.

```javascript
T Bool
| True
| False

not : {x : Bool} -> Bool
  case<Bool> x
  | True  => False
  | False => True
  : Bool

main : Bool
  not(False)
```

As an exercise, implement `bool_to_nat`, which returns `1` or `0`.

## 3. Efficient branching

Let's now translate the following `and` function:

```haskell
-- Exhaustive patterns for the sake of demonstration
and :: Bool -> Bool -> Bool
and True  True  = True
and False True  = False
and True  False = False
and False False = False
```

Converting the syntax to Formality:

```javascript
and : {a : Bool, b: Bool} -> Bool
  case<Bool> a
  | True  => b
  | False => False
  : Bool

and2 : {|a : Bool, |b: Bool} -> Bool
| True  | True  = True
        | False = False
| False | True  = False
        | False = False
```

Here is another example:

```javascript
let swap    = 0 // Change to 1 to swap
let big_arr = [ 0, [ 1, [ 2, [ 3,[ 4,[ 5,[ 6,[ 7, 8]]]]]]]]
let big_str = "I'm a big string with a lot of stuff!"
(if swap
  then: {big_arr big_str} [big_str, big_arr]
  else: {big_arr big_str} [big_arr, big_str]
  big_arr big_str)
```

This snippet creates an array and a string and then makes a pair of both with
either the string first or the array first, depending on the value of `swap`.
Despite using `big_str` and `big_arr` in both branches, those values were never
copied thanks to the technique above.

Run the program below with `fmc main`.

```javascript
def True: {True False}
  True

def False: {True False}
  False

def and: {a b}
  let case_a_True = {b}
    let case_b_True  = True
    let case_b_False = False
    (b case_b_True case_b_False)
  let case_a_False = {b}
    let case_b_True  = False
    let case_b_False = False
    (b case_b_True case_b_False)
  (a case_a_True case_a_False b)

def main: 
  let bool = (and True False)

  // Prints the value of Bool
  let case_True  = "I'm true!"
  let case_False = "I'm false!"
  (bool case_True case_False)
```

As an exercise, implement `or`.

## 4. Datatypes with fields

In Haskell, pairs can be defined as:

```haskell
{-# LANGUAGE NoImplicitPrelude #-}

data Pair a b
  = NewPair a b
```

This puts 1 constructor, `NewPair`, in scope. This is the corresponding
Formality-Core definitions:

```javascript
NewPair {a, b} {NewPair}
  NewPair(a, b)
```

Notice that those are mostly similar to `True` and `False`, except now there are
two fields, `{a b}`, involved. Let's write the first accessor function in the
Formality-ready form:

```haskell
fst :: Pair a b -> a
fst = \ pair -> 
  let case_NewPair = \ a b -> a in
  case pair of { NewPair a b -> case_NewPair a b }
```

Notice that, here, each field of the datatype became a lambda on the
`case_NewPair` expression. This is the Formality translation:

```javascript
def fst: {pair}
  let case_NewPair = {a b} a
  (pair case_NewPair)
```

Run the program below with `fmc main`.

```javascript
def NewPair: {a b} {NewPair}
  (NewPair a b)

def get_first: {pair}
  let case_NewPair = {a b} a
  (pair case_NewPair)

def main: 
  let pair = (NewPair 1 2)

  // Prints the first element of `pair`
  (get_first pair)
```

As an exercise, implement `pair_swap`.

## 5. Non-recursive datatypes

Here are 2 more Haskell datatypes:

```haskell
data Maybe a
  = Nothing
  | Just a

data Either a b
  = Left a
  | Right b
```


This puts 4 constructors, `Nothing`, `Just`, `Left`, `Right` in scope. Those are
the corresponding Formality-Core definitions:

```javascript
def Nothing: {Nothing Just}
  Nothing

def Just: {a} {Nothing Just}
  (Just a)

def Left: {a} {Left Right}
  (Left a)

def Right: {b} {Left Right}
  (Right b)
```

From this, you should be able to grasp the general pattern:

```javascript
def Ctor_0: {ctor_0_field_0 ctor_0_field_1 ...} {Ctor_0 Ctor_1 ...}
  (Ctor_0 ctor_0_field_0 ctor_0_field_1 ...)

def Ctor_1: {ctor_1_field_0 ctor_1_field_1 ...} {Ctor_0 Ctor_1 ...}
  (Ctor_1 ctor_1_field_0 ctor_1_field_1 ...)

...
```

Let's now implement a `from_just :: Maybe a -> Either String a` function that
either extracts the value of a `Maybe`, or returns an error:

```
def from_just: {error_msg maybe_val}
  let case_nothing = (Left error_msg)
  let case_just    = {val} (Right val)
  (maybe_val case_nothing case_just)
```

By this point, you should be able to understand this. The general pattern for
matching is:

```
let case_Ctor_0 = {ctor_0_field_0 ctor_0_field_1 ...} result_on_case_Ctor0
let case_Ctor_1 = {ctor_1_field_0 ctor_1_field_1 ...} result_on_case_Ctor1
...
```

Run the program below with `fmc main`.

```javascript
def Nothing: {Nothing Just}
  Nothing

def Just: {a} {Nothing Just}
  (Just a)

def Left: {a} {Left Right}
  (Left a)

def Right: {b} {Left Right}
  (Right b)

def from_just: {error_msg maybe_val}
  let case_nothing = (Left error_msg)
  let case_just    = {val} (Right val)
  (maybe_val case_nothing case_just)

def main:
  let maybe_a = Nothing
  let maybe_b = (Just 3)
  (from_just "I'm not a number." maybe_a)
```

As an exercise, implement `to_maybe :: Either a b -> Maybe b`.

## 6. Unboxed copying

Remember that Formality-Core functions can only use its bound variable once.
Because of that, it is hard to make duplicates of a value. For example, this
isn't possible:

```javascript
// square : Nat -> Nat
def square: {n}
  |n * n|
```

Here, we learned how to avoid this problem when the copy is needed in a
different branch, but what if we really need the value twice, like on the case
above? Fortunately, as explained on the
[wiki](https://github.com/moonad/Formality/wiki/Dups-and-Boxes), Formality-Core
includes an explicit duplication system that allows us to write this:

```javascript
// square : !Nat -> !Nat
def square: {n}
  dup n = n
  # |n * n|
```

But this kind of definition has an important limitation: it can only affect a
number in a layer below it! In general, a term on layer `N` can't read any
information from a term on layer `N+1`. Because of that, you **often want all
data of your program to live in a single layer and avoid boxes as much as
possible**. But then, how do we copy values? On the `square` case, what you want
to do is to use the `cpy` primitive, which allows you to copy a number as many
times as you want:

```javascript
// square : Nat -> Nat
def square: {n}
  cpy n = n
  |n * n|
```

For user-defined algebraic datatype, you must write an explicit `copy` function,
which performs a pattern-match and explicitly returns copies of the same value.
For example:

```javascript
def copy_bool: {b}
  let case_true  = [True, True]
  let case_false = [False, False]
  (b case_true case_false)

def main:
  let bool = True
  get [bool_cpy_0, bool_cpy_1] = (copy_bool bool)
  ...
```

This is an annoying complication that could and should be automated on the
Formality language. When dealing with Formality-Core, writing explicit copy
functions is one of the programmer's job. As an exercise, write a
`copy_bool_pair` function that copies a pair of bools (i.e., `(copy_bool_pair
[True, False]) == [[True,False], [True,False]]`).

## 7. Recursive datatypes

In Haskell, natural numbers and linked lists can be defined as:

```haskell
{-# LANGUAGE NoImplicitPrelude #-}

data Nat
  = Succ Nat
  | Zero

data List a
  = Cons a (List a)
  | Nil
```

This puts 4 constructors, `Succ`, `Zero`, `Cons`, `Nil` in scope. Those are the
corresponding Formality-Core definitions:

```javascript
def Succ: {n} {Succ Zero}
  (Succ n)

def Zero: {Succ Zero}
  Zero

def Cons: {x xs} {Cons Nil}
  (Cons x xs)

def Nil: {Cons Nil}
  Nil
```

There is no surprise here: the definitions are identical to non-recursive
datatypes. What changes, though, is that we can't use those datatypes as
expected. This, for example, won't work:

```javascript
def length: {list}
  let case_cons = {x xs} (Succ (length xs))
  let case_nil  = Nil
  (list case_cons case_nil)
```

The problem is that, being a terminating language, recursion isn't allowed.
Recursive datatypes without recursive functions are of limited use. What now?

That's when boxes become useful. Remember that I told you to avoid using boxes
to copy data? That's because the real use of boxes is to capture loops, folds,
and recursion through the so-called "Church-Encodings". This is best explained
through examples. To recursive over a list of length up to 4, this is how you do
it:

```javascript
def Cons: {x xs} {Cons Nil}
  (Cons x xs)

def Nil: {Cons Nil}
  Nil

def rec4: {call stop}
  dup call = call
  dup stop = stop
  # (call (call (call (call stop))))

def length:
  let call = {rec list}
    let case_cons = {x xs} |1 + (rec xs)|
    let case_nil  = 0
    (list case_cons case_nil)
  let stop = 0
  dup length = (rec4 #call #stop)
  # {list}
    (length list)

def main:
  dup length = length
  # let list = (Cons 7 (Cons 7 Nil))
    ["Length of the list is:", (length list)]
```

Let's take a moment to understand what is going on here because this is very
important and unusual. First, we create `rec4`, which is the "Church-Encoded"
natural number 4, which allows us to execute a recursive function up to 4 calls.
Then, we implement `length` using it. There, we define `call`, which is exactly
what the usual Haskell `length` function would look like, except that it
receives an extra argument, `rec`, to refer to itself. Then, we define `stop`,
which is the "default" value in the case the recursion hits its call limit
(here, `4`). Then we create the `length` function by applying `rec4` to `#call`
and `#stop`. That function will only work up to 4 calls. Since we used `dup`s on
`layer 0`, this `length` function will only be available on `layer 1`. That's
why there is a `#` before `{list}`: our entire function is written on layer 1.
On this case, we just receive a list and apply `length` to it.

Crazy, right? This is, by far, the most confusing aspect of Formality, so, don't
get afraid if you don't get it at first. In general, though, you can expect
Formality-Core programs to follow roughly the form of the `length` function.
First, we "configure" the recursive calls and their "call limits" on layer 0,
and then we use them on layer 1. From my experience, under normal circumstances,
Formality-Core programs should use no more than two layers. Take a look at
[`Kaelin`](https://github.com/moonad/Formality/blob/master/stdlib/kaelin.fmc), a
small game written in Formality-Core: recursive functions like `write`,
`update`, `vec2_range` are "configured" on layer 0, while the game logic and its
data live on layer 1. (Note: actually, right now, it is on layer 2, but
disregard as this will be changed soon.)

Note: since `recN` is so common, in order to avoid writing it for each `n`,
there is quick syntax-sugar to generate it. First, define `rec` as:

```
def rec: {n call stop}
  dup callN = (n call)
  dup stop  = stop
  # (callN stop)
```

Then, you can write `(rec ~N)` for any `N`.

## 8. Fusing loops

The `~` syntax generates a ultra-compact Church Nats. For example, `~256`
becomes:

```javascript
{s}
  (dup s0 = s
  (dup s1 = #{x} (s0 (s0 x))
  (dup s2 = #{x} (s1 (s1 x))
  (dup s3 = #{x} (s2 (s2 x))
  (dup s4 = #{x} (s3 (s3 x))
  (dup s5 = #{x} (s4 (s4 x))
  (dup s6 = #{x} (s5 (s5 x))
  (dup s7 = #{x} (s6 (s6 x))
  (dup s8 = #{x} (s7 (s7 x))
    #{x} (s8 x))))))))))
```

Instead of the full form (with `256` applications of `s`). This compact form is
useful for getting asymptotical speed-ups over traditional functional languages.
For example, the program below:

```javascript
def True: {True False}
  True

def False: {True False}
  False

def not: {a True False}
  let case_True  = False
  let case_False = True
  (a case_True case_False)

def main: 
  let num      = ~1000000000000
  dup num_nots = (num #not)
  # (num_nots True "num is even" "num is odd")
```

Applies `not` one trillion times to the boolean `True` and returns instantly if
running on interaction nets (`fmc -l main`). As an experiment, you can try
changing the number to anything else and it will always output whether `num` is
even or odd, even a computer shouldn't be able to perform 1 trillion function
calls that quickly. That's one of the cool aspects of Formality and boils down
to its ability to perform fusion (like Haskell's foldr/build technique) at
runtime, merging compositions of `not` and essentially performing a loop of `N`
iterations in `O(log(N))` graph-rewrites. I write more about this effect [on
this
article](https://medium.com/@maiavictor/solving-the-mystery-behind-abstract-algorithms-magical-optimizations-144225164b07),
and implement a quick, elegant `exp_mod` [on this
Gist](https://gist.github.com/MaiaVictor/e556062185c5863d814980123e03630f).

---

Uff, that's a lot of information already! What now? Help me [improving this
guide](https://github.com/moonad/docs.formality-lang.org/issues)
