## Datatypes

Formality includes a powerful datatype system. A new datatype can be defined with the `T` syntax, which is similar to Haskell's `data`. It creates one definition for the type of the datatype being defined, and one definition for each of its constructors. To pattern-match against the value of a datatype, you must use `case<T>`.

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

### Container datatypes

Datatype constructors can have fields:

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

Since `Nat` in so common, there is a syntax-sugar for it: `0n3` expands to `succ(succ(succ(zero)))`.

### Polymorphic datatypes

What if we wanted to make a 3D vector with another type of number? Instead of creating a new datatype, we can simply use a polymorphic definition:

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

Notice how `main` has the type `Vector(String, 0n3)`, showing that it is a vector with exactly 3 strings. If we used `vcons` again, the type would change to `Vector(String, 0n4)`. This feature allows us to annotate our data with pretty much any kind of arbitrary static information, which, in turn, allows us to develop extremely type-safe programs. For example, here is a `head` function that extracts the first element of a non-empty vector:

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

Notice how the return-type of the `case` expression is allowed to access `len`. This allows Formality to specialize the return-type to the vector's length on each case. On `vcons`, the length is `succ(...)`, so we must provide an element of type `T`. On `vnil`, the length is `zero`, so we must provide any arbitrary `Word`. Then, the return type of the case expression itself is computed based on he index of `vector`, which is `succ(n)`. Since that is positive, the return type is always `T`, as Formality knows it will never fall on the `vnil` case. Calling `vhead` with a non-empty vector is impossible because we'd need to call it with an `n` such that `succ(n)` is `zero`, but there is no such natural number. 

Of course, since the length annotation is used only for type-checking purposes, computing it at runtime would be wasteful. That's why we use `~`. This allows the length to be dropped from the compiled output, avoiding any extra runtime cost.

### Self-Encodings

Interestingly, none of the features above are part of Formality's type theory. Instead, they are a lightweight syntax-sugar that elaborates to plain-old lambdas. To be specific, a datatype is encoded as is own inductive hypothesis, using "self-types" to. For example, the Bool datatype desugars to:

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
**TODO**: write a brief explanation on how it works.
