## Datatypes

Formality includes a powerful datatype system. A new datatype can be defined with the `T` syntax, which is similar to Haskell's `data`, and creates global definitions for its type and constructors. To pattern-match against a value of a datatype, you must use `case<T>`.

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

### Record-like datatypes

Datatype constructors can have fields, allowing them to store values:

```javascript
T Vector3D
| v3 {x : Word, y : Word, z : Word}

main : Word

  let a = v3(10, 20, 30)

  case<Vector3D> a
  | v3 => x + y + z
  : Word
```

Notice that, inside the `v3` case, the `x`, `y` and `z` fields are automatically available. To avoid name-shadowing and access a field of an outer pattern-match, you can either use a `let` or append `^` to the variable name:

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
      x + x^
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

Since `Nat` is so common, there is a syntax-sugar for it: `0n3` expands to `succ(succ(succ(zero)))`.

### Polymorphic datatypes

Polymorphic datatypes allow us to create multiple instances of the same datatype with different contained types.

```javascript
T Vector3D <T : Type>
| v3 {x : T, y : T, z : T} 

main : [:Nat, Word]
  let a = v3<Nat>(0n1, 0n2, 0n3)
  let b = v3<Word>(1, 2, 3)

  let ax =
    case<Vector3D> a
    | v3 => x
    : Nat

  let bx =
    case<Vector3D> b
    | v3 => x
    : Word

  [ax, bx]
```

The program above creates two 3D vectors, the first one storing `Nat`s and the second one storing `Word`s. The polymorphic variable `T`, defined with the `<>` syntax, allowed us to define `Vector3D` only once. With this, we can also create the popular functional List type:

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

This is a little verbose, though, since we had to instantiate each use of cons with `<Word>`. You could make it shorter with `let`s:

```javascript
let cons = cons<Word>
let nil  = nil<Word>
let list = cons(1, cons(2, cons(3, nil)))
```

But, since `List` is so common, there is a built-in syntax-sugar for it, the dot-notation:

```javascript
let list = .Word[1, 2, 3]
```

### Indexed datatypes

Indexes are like polymorphic variables, except that they can change as the structure grows. For example, a Vector is like a List, except that its type stores its own length:


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

The `&` syntax was used to apply the indexes to the returned type of each constructor. For example, `| vnil & zero` would be equivalent to `vnil : Vector T zero` in Agda. In this example, `main` has the type `Vector(String, 0n3)`, meaning it is a vector with exactly 3 strings. If we used `vcons` again, the type would change to `Vector(String, 0n4)`. This feature allows us to annotate our data with very rich static information, allowing us to prevent a wide range of bugs. For example, here is a `head` function that can only be called in non-empty vectors:

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

To understand how it works, notice that the return-type of the `case` expression is allowed to access `len`. This allows Formality to specialize the expected type of each case. On `vcons`, the length is `succ(...)`, so we must provide an element of type `T`. On `vnil`, the length is `zero`, so we must provide any arbitrary `Word`. Then, the return type of the case expression itself is computed based on he index of the matched `vector`, which is `succ(n)`. Since that is positive, the return type is always `T`, i.e., Formality knows it will never fall on the `vnil` case. Calling `vhead` with a non-empty vector is impossible because we'd need an `n` such that `succ(n)` is `zero`, but there is no such natural number. 

Of course, since the length annotation is used only for type-checking purposes, computing it at runtime would be wasteful. That's why we use `~`. This allows the length to be dropped from the compiled output, avoiding any extra runtime cost.

### Self-Encodings

Interestingly, none of the features above are part of Formality's type theory. Instead, they are lightweight syntax-sugars that elaborate to plain-old lambdas. To be specific, a datatype is encoded as is own inductive hypothesis, with "self-types". For example, the Bool datatype desugars to:

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

case_of : {b : Bool, ~P : {x : Bool} -> Type, t : P(true), f : P(false)} -> P(b)
  (%b)(~P, t, f)
```

Here, `$self ...`, `new<T> val` and `%b` are the type, introduction, and elimination of self-types, respectively. You can see how any datatype is encoded under the hoods by asking `fm` to evaluate its type, as in, `fm any_file.Bool` or `fm any_file.Nat`. While you probably won't need to deal with self-encodings yourself, knowing how they work is valuable, since it allows you to express types not covered by the built-in syntax.

**TODO**: write a brief explanation on how Self-Types work (although I think it should be self-explanatory from this example!).
