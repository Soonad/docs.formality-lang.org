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
addition | `|x + y|` | `(x + y) >>> 0`
subtraction | `|x - y|` | `(x - y) >>> 0`
multiplication | `|x * y|` | `(x * y) >>> 0`
division | `|x / y|` | `(x / y) >>> 0`
modulus | `|x % y|` | `(x % y) >>> 0`
exponentiation | `|x ** y|` | `(x ** y) >>> 0`
bitwise-and | `|x & y|` | `x & y`
bitwise-or | `|x | y|` | `x | y`
bitwise-xor | `|x ^ y|` | `x ^ y`
bitwise-not | `|x ~ y|` | `~y`
bitwise-right-shift | `|x >> y|` | `x >>> y`
bitwise-left-shift | `|x << y|` | `x << y`
greater-than | `|x > y|` | `x > y ? 1 : 0`
less-than | `|x < y|` | `x < y ? 1 : 0`
equals | `|x == y|` | `x === y ? 1 : 0`

The type of a native number is `Word`. Note that there is no operator precedence. Instead, all operations must be wrapped by a `||`. For example, to compute `2 + (3 * 7)`, you must write:

```javascript
main : Word
  |2 + |3 * 7||
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
  |a + b|
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
`if n p` | If `n == 0`, evaluates to `snd p`, else, evaluates to `fst p`
`then: a else: b` | Equivalent to `[a, b]`

Usage is straightforward:

```javascript
main : Output
  let age = 30

  if |age < 18|
  then:
    print("boring teenager")
  else:
    print("respect your elders!")
```

## Cpy

Words can be copied with `cpy`:

```javascript
main : Word
  cpy x = 42
  [x, |x + x|]
```

Why this is necessary will become clear later on.

## Functions

Formality includes primitives for creating and applying functions:

syntax | effect
--- | ---
`{x : A, y : B, z : C, ...} -> D` | The type of a function with argumens `x : A`, `y : B`, `z : C`, returning a `D`
`{x, y, z, ...} body` | A function that receives the arguments `x y z` and returns `body` 
`f(x, y, z, ...)` | Applies the function `f` to the arguments `x`, `y`, `z` (curried)

Formality functions are anonymous expressions, like Haskell's lambdas. It has no multi-argument lambdas; `{x, y, z, ...} body` is just a syntax-sugar for `{x} {y} {z} ... body`. This is the same as JavaScript's `x => y => z => ... body`, or Haskell's `\ x y z ... -> body`. Function calls use the more conventional `f(x, y, z)` syntax, which, since functions are curried, is just a syntax-sugar for `f(x)(y)(z)...`. The type of a function is `{x : A, y : B, z : C ...} -> D`, which is equivalent to Agda's `(x : A) -> (y : B) -> (z : C) -> ... D`. Formality does not have a short "arrow-notation" such as `A -> B`; names are mandatory, so, you must write `{x : A} -> B` instead. When annotating a top-level definition with a functional type, you don't need to write the lambdas, they are added implicity. For example:

```javascript
birth_to_age : {birth : Word} -> Word
  |2019 - birth|

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

## Datatypes

Formality includes very powerful syntaxes for the creation and inspection of datatypes. They can be defined with the `T` syntax, which is similar o a Haskell's `data` syntax. It creates one definition for the type of the datatype being defined, and one definition for each of its constructors. To pattern-match against the value of a datatype, you must use the `case<T>` syntax.

### Simple datatypes (enums)

A simple datatype, or enum, can be defined as:

```javascript
T Suit
| clubs
| diamonds
| hearts
| spades

main : Output

  let suit = clubs

  case<Suit> suit
  | clubs    => print("Welcome to it!")
  | diamonds => print("With I had a few.")
  | hearts   => print("You always had mine.")
  | spades   => print("Is it an ace?")
  : Output
```

The program above creates a datatype, `Suit`, with 4 cases (constructors). It then pattern-matches a suit and outputs a different sentence depending on it. Notice that a `case` expression requires a type annotation below it.

### Datatypes with fields

Datatype constructors can have fields:

```javascript
T Vector3D
| v3 {x : Word, y : Word, z : Word}

main : Word

  let a = v3(10, 20, 30)

  case<Vector3D> a
  | v3 => |x + |y + z||
  : Word

```

Notice that, inside the `v3` case, the `x`, `y` and `z` fields are automatically available. To avoid name-shadowing and access a field of an outer pattern-match, you can use either a `let`, or append `^` to the variable name:

```javascript
T Vector3D
| v3 {x : Word, y : Word, z : Word}

main : Word

  let a = v3(10, 20, 30)
  let b = v3(30, 20, 10)

  case<Vector3D> a
  | v3 =>
    case<Vector3D> b
    |v3 =>
      |x + x^|
    : Word
  : Word
```

### Recursive datatypes

A recursive datatype can be defined as expected:

```javascript
T Nat
| succ {pred : Nat}
| zero

main : Output
  let n = succ(succ(succ(zero)))

  case<Nat> n
  | succ => print("n is positive")
  | zero => print("n is zero")
  : Output
```

Since `Nat` in so common, there is a syntax-sugar for it: `0n3` expands to `succ(succ(succ(zero)))`.

### Polymorphic datatypes

What if we wanted to make a 3D vector with another type of number? Instead of creating a new datatype, we can simply use a polymorphic definition:

```javascript
T Vector3D <T : Type>
| v3 {x : T, y : T, z : T} 

main : [:Nat, Word]
  let a = v3<Nat>(0n1, 0n2, 0n3)
  let b = v3<Word>(1, 2, 3)

  let ax = case<Vector3D> a | v3 => x : Nat
  let bx = case<Vector3D> b | v3 => x : Word

  [ax, bx]
```

The program above creates two 3D vectors, each one storing a different type of numbers: `a` stores `Nat` and `b` stores `Word`. It then extracts the `x` component of each one and makes a pair. In order to avoid defining `Vector3D` twice, a polymorphic variable was introduced and applied with the `<>` syntax. This allows us to create the well-known functional List type:

```javascript
T List <T : Type>
| cons {head : T, tail : List(T)}
| nil

main : Output
  let list = cons<Word>(1, cons<Word>(2, cons<Word>(3, nil<Word>)))

  case<List> list
  | cons => print("List has elements")
  | nil  => print("List is empty")
  : Output
```

Notice how we had to instantiate each use of cons with `<Word>`. Because of that, polymorphic datatypes tend to get quote verbose, but you can make it shorter with `let`s:

```javascript
let cons = cons<Word>
let nil  = nil<Word>
let list = cons(1, cons(2, cons(3, nil)))
```

Moreove, since `List` is so common, there is a built-in syntax-sugar for it, the dot-notation:

```javascript
let list = .Word[1, 2, 3]
```

### Indexed datatypes

One could say indexed datatypes are the main difference between a proof language and a normal functional language. They are like polymorphic datatypes, except that the type can depend on values, not only other types, which can change as the structure grows. For example, a Vector is like a List, except that its type stores its own length:


```javascript
T Vector <T : Type> {len : Nat}
| vcons {~len : Nat, head : T, tail : Vector(T, len)} & succ(len)
| vnil & zero

main : Vector(String, 0n3)
  vcons<String>(~0n2, "ichi",
  vcons<String>(~0n1, "ni",
  vcons<String>(~0n0, "san",
  vnil<String>)))
```

Notice how `main`  has type `Vector(String, 0n3)`, showing that it is a vector with exactly 3 strings. If we used `vcons` again, the type would change to `Vector(String, 0n4)`. This feature allows us to annotate our data with pretty much any kind of arbitrary static information, which, in turn, allows us to extremelly type-safe programs. For example, here is a `head` function that extracts the first element of a non-emptpy vector:

```javascript
T Vector <T : Type> {len : Nat}
| vcons {~len : Nat, head : T, tail : Vector(T, len)} & succ(len)
| vnil & zero

vhead : {~T : Type, ~n : Nat, vector : Vector(T, succ(n))} -> T
  case<Vector> vector
  | vcons => head
  | vnil  => 1337
  : case<Nat> len
    | succ => T
    | zero => Word
    : Type

main : Output
  let vec =
    vcons<String>(~0n2, "uno",
    vcons<String>(~0n1, "dos",
    vcons<String>(~0n0, "tres",
    vnil<String>)))

  print(vhead<String>(~0n2, vec))
```

Notice how the return-type of the `case` expression is allowed to access `len`, i.e., the length of the matched vector. This allows Formality to specialize the return-type to the length of each case. On `vcons`, the length is `succ(...)`, so we must provide an element of type `T`. On `vnil`, the length is `zero`, so we must provide any arbitrary word. Then, the return type of the case expression since the length of the vector is always positive, Formality knows it will never fall on the `vnil` case. Calling `vhead` with a non-empty vector is impossible, because we'd need to call it with an `n` such that `succ(n)` is `zero`, which is impossible. 

Of course, since the length annotation is used only for type-checking purposes, computing it at runtime would be wasteful. That's why we use `~`. This allows the length to be dropped from the compiled output, avoiding any extra runtime cost.

## Self-Encodings

Interestingly, none of the features above are part of Formality's type theory. Instead, they are a lightweight syntax-sugar that elaborates to plain-old lambdas. To be specific, a datatype is encoded as is own inductive hypothesis, using "self-types" to . For example, the Bool datatype desugars to:

```javascript
Bool : Type
  $self
  { ~P    : {x : Bool} -> Type
  , true  : P(true)
  , false : P(false)
  } -> P(self)

true : Bool
  new<Bool>{~P, true, false} true

false : Bool
  new<Bool>{~P, true, false} false

case_of : {b : Bool, ~P : {x : Bool} -> Type, t : Bool(true), f : Bool(false)} -> P(b)
  (%b)(~P, t, f)
```

Here, `$self ...`, `new<T> val` and `%b` are the type, introduction and elimination of self-types, respectivelly. You can see how any datatype is encoded under the hoods by asking `fm` to evaluate its type, as in, `fm any_file.Bool` or `fm any_file.Nat`. While you probably won't need to deal with self-encodings yourself, knowing how they work is valuable, since it allows you to express types not covered by the built-in syntax. TODO: write a brief explanation on how it works.
