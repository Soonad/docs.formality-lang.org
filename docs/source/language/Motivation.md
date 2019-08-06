# Motivation

Formality exists to fill a hole in the current market: there aren't many languages featuring *theorem proving* that are also 1. built to be normal, user-friendly programming languages (instead of tools built for mathematicians and PhDs), and 2. to be very efficient at runtime. Right now, Formality isn't *that* user-friendly, but we aim to achieve that by simplifying a lot of the obscure idioms through familiar syntax-sugars that hide its more advanced aspects. Regardnig efficiency, it achieves so by relying on Elementary Affine Logic, which gives it many desirable computational characteristics that make Formality an objectively fast language.

### Optimal substitutions

Formality's substitution algorithm is asymptotically faster than Haskell, Scheme, JavaScript and other languages featuring closures. This makes it extremely fast at evaluating programs that rely on many high-order functions. For example, do you know GHC's marvelous foldr/build and stream fusion optimizations? Formality is capable of performing those [at runtime](https://medium.com/@maiavictor/solving-the-mystery-behind-abstract-algorithms-magical-optimizations-144225164b07). This also allows us to explore new ways to develop algorithms, such as this [elegant exp-mod implementation based on billions of tuple rotations](https://gist.github.com/MaiaVictor/e556062185c5863d814980123e03630f), which would be impossible in any other functional language. Who knows if this may lead to new breakthroughs in complexity theory?

### Fast "by design"

While the actual efficiency of a programming language depends on the implementation, some languages are inherently slow, by design. JavaScript, for example, is slower than C: all things equal, its mandatory garbage collector will be an unavoidable extra cost. Formality is, by design, as fast as a language could be. For example, due to linearity (actually, affinity), it doesn't require a garbage collector. Due to the confluence of its interaction-combinator runtime, it can be evaluated in a massively parallel fashion. Due to Elementary Affine Logic (and unlike [BOHM](https://github.com/cls/bohm), it doesn't require bookkeeping machinery for its optimal substitutions. It can be evaluated lazily, just like Haskell. And so on. While Formality is, right now, not as fast as mature languages (after all, those have years of optimizations behind), it will inevitably reach those languages once more mature.

### An elegant underlying theory

We conjecture that Formality's unique approach to termination allows its type system to have a bunch of powerful features that would otherwise be impossible without making the proof language inconsistent. As an example, it features `Type : Type`, which is powerful and very convenient. It also features mutual type-level recursion, allowing us to exploit self-types to elegantly represent datatypes as their own induction schemes, without needing a complex native datatype system. We're working hard towards proofs of those claims and hope they can be published soon.

### Example Code

```javascript
// Vectors are lists with stactically-known lengths
T Vector <T : Type> {len : Nat}
| vcons {~len : Nat, head : T, tail : Vector(T, len)} & succ(len)
| vnil                                                & zero

// A type-safe head that can't be called on non-empty numbers
vhead : {~T : Type, ~n : Nat, vector : Vector(T, succ(n))} -> T
case<Vector> vector
| vcons => head
| vnil  => 0
: case<Nat> len
    | succ => T
    | zero => Word
    : Type

// The built-in equality type
Eq : {~A : Type, ~B : Type, ~a : A, ~b : B} -> Type
  a == b

// Congruence (`a == b` implies `f(a) == f(b)`)
cong : {~A : Type, ~B : Type, ~a : A, ~b : A, ~f : A -> B, ~e : a == b} -> f(a) == f(b)
  rewrite<e>{x in f(a) == f(x)}(refl<f(a)>)
```
