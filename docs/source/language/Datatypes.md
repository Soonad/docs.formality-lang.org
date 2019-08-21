## Datatypes

Formality includes a powerful datatype system. A new datatype can be defined with the `T` syntax, which is similar to Haskell's `data`, and creates global definitions for its type and constructors. To pattern-match against a value of a datatype, you must use `@T`.

### Simple datatypes (enums)

A simple datatype, or enum, can be defined as:

```javascript
T Suit
| clubs
| diamonds
| hearts
| spades

main : Output

  let suit = spades

  @Suit suit ~> Output
  | clubs    => print("First rule: you do not talk about Fight Club.")
  | diamonds => print("Queen shines more than diamond.")
  | hearts   => print("You always had mine.")
  | spades   => print("The only card I need is the Ace of Spades! \m/")
```

The program above creates a datatype, `Suit`, with 4 cases (constructors). It then pattern-matches a suit and outputs a different sentence depending on it. The `@` expression requires he name of the matched datatype, the matched value, the returned type, and then each case provided with a `|`. In some cases, the name of the datatype can be omitted.

### Record-like datatypes

Datatype constructors can have fields, allowing them to store values:

```javascript
T Vector3D
| v3 {x : Word, y : Word, z : Word}

main : Word

  let a = v3(10, 20, 30)

  @Vector3D a ~> Word
  | v3 => x + y + z
```

Notice that, inside the `v3` case, you can access the `x`, `y` and `z` fields of he matched value. To prevent name-shadowing, you can use a `let`:

```javascript
T Vector3D
| v3 {x : Word, y : Word, z : Word}

main : Word

  let a = v3(10, 20, 30)
  let b = v3(30, 20, 10)

  @Vector3D a ~> Word
  | v3 =>
    let a.x = x
    @Vector3D b ~> Word
    |v3 =>
      let b.x = x
      a.x + b.x
```

You can also use `^` to refer to a variable on the outer scope:

```javascript
T Vector3D
| v3 {x : Word, y : Word, z : Word}

main : Word

  let a = v3(10, 20, 30)
  let b = v3(30, 20, 10)

  @Vector3D a ~> Word
  | v3 =>
    @Vector3D b ~> Word
    |v3 =>
      x + x^
```

### Recursive datatypes

A recursive datatype may contain fields of the same type. An example is `Nat`:

```javascript
T Nat
| succ {pred : Nat}
| zero

main : Output
  let n = succ(succ(succ(zero)))

  @Nat n ~> Output
  | succ => print("n is positive")
  | zero => print("n is zero")
```

Since it is so common, there is a syntax-sugar for it: `0n3`, which expands to `succ(succ(succ(zero)))`.

### Polymorphic datatypes

Polymorphic datatypes allow us to create multiple instances of the same datatype with different contained types.

```javascript
T Vector3D <T : Type>
| v3 {x : T, y : T, z : T} 

main : [:Nat, Word]
  let a = v3<Nat>(0n1, 0n2, 0n3)
  let b = v3<Word>(1, 2, 3)

  let ax =
    @Vector3D a ~> Na
    | v3 => x

  let bx =
    @Vector3D b ~> Word
    | v3 => x

  [ax, bx]
```

The program above creates two 3D vectors, the first one storing `Nat`s and the second one storing `Word`s. The polymorphic variable `T`, defined with the `<>` syntax, allowed us to define `Vector3D` only once. With this, we can also create the popular functional List type:

```javascript
T List <T : Type>
| cons {head : T, tail : List(T)}
| nil

main : Output
  let list = cons<Word>(1, cons<Word>(2, cons<Word>(3, nil<Word>)))

  @List list ~> Output
  | cons     => print("List has elements")
  | nil      => print("List is empty")
```

This is a little verbose, though, since we had to instantiate each use of cons with `<Word>`. You could make it shorter with `let`s:

```javascript
let cons = cons<Word>
let nil  = nil<Word>
let list = cons(1, cons(2, cons(3, nil)))
```

But, since `List` is so common, there is a built-in syntax-sugar for it, the dollar sign:

```javascript
let list = Word$[1, 2, 3]
```

### Indexed datatypes

Indexes are like polymorphic variables, except that they can change as the structure grows. For example, a Vector is like a List, except that its type stores its own length:

```javascript
T Vector <T : Type> {len : Nat}
| vcons {~len : Nat, head : T, tail : Vector(T, len)} & succ(len)
| vnil                                                & zero

main : Vector(String, 0n3)
  vcons<String>(~0n2, "ichi",
  vcons<String>(~0n1, "ni",
  vcons<String>(~0n0, "san",
  vnil<String>)))
```

The `&` syntax was used to apply the indexes to the returned type of each constructor. For example, `| vnil & zero` would be equivalent to `vnil : Vector T zero` in Agda. In this example, `main` has the type `Vector(String, 0n3)`, meaning it is a vector with exactly 3 strings. If we used `vcons` again, the type would change to `Vector(String, 0n4)`. This feature allows us to annotate our data with very rich static information, allowing us to prevent a wide range of bugs. For example, here is a `vhead` function that can only be called in non-empty vectors:

```javascript
T Vector <T : Type> {len : Nat}
| vcons {~len : Nat, head : T, tail : Vector(T, len)} & succ(len)
| vnil & zero

vhead : {~T : Type, ~n : Nat, vector : Vector(T, succ(n))} -> T
  @ vector ~> @Nat len ~> Type | succ => T | zero => Word
  | vcons  => head
  | vnil   => 1337

main : Output
  let vec =
    vcons<String>(~0n2, "uno",
    vcons<String>(~0n1, "dos",
    vcons<String>(~0n0, "tres",
    vnil<String>)))

  print(vhead<String>(~0n2, vec))
```

To understand how it works, notice that the return-type of the `case` expression is allowed to access `len`. This allows Formality to specialize the expected type of each case. On `vcons`, the length is `succ(...)`, so we must provide an element of type `T`. On `vnil`, the length is `zero`, so we must provide any arbitrary `Word`. Then, the return type of the case expression itself is computed based on the index of the matched `vector`, which is `succ(n)`. Since that is positive, the return type is always `T`, i.e., Formality knows it will never fall on the `vnil` case. Calling `vhead` with an empty vector is impossible because we'd need an `n` such that `succ(n)` is `zero`, but there is no such natural number. 

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
