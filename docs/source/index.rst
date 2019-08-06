Welcome to Formality's documentation!
=====================================

Formality is an optimal functional programming language featuring theorem proving. It is similar to Agda and Idris in functionality, but takes a different approach to termination and induction: instead of native datatypes with structural recursion, it uses λ-encodings, self-types and relies on a different underlying logic, "elementary affine", which gives it an elegant halting argument. This gives it some unique properties such as optimal substitutions, practical efficiency, and an elegant underlying theory. To elaborate,

Optimal substitutions
-------- 

Formality's substitution algorithm is asymptotically faster than Haskell, Scheme, JavaScript and other languages featuring closures. This makes it extremely fast at evaluating programs that rely on many high-order functions. For example, do you know GHC's marvelous foldr/build and stream fusion optimizations? Formality is capable of performing those `at runtime <https://medium.com/@maiavictor/solving-the-mystery-behind-abstract-algorithms-magical-optimizations-144225164b07>`_. This also allows us to explore new ways to develop algorithms, such as this `elegant exp-mod implementation based on billions of tuple rotations <https://gist.github.com/MaiaVictor/e556062185c5863d814980123e03630f>`_, which would be impossible in any other functional language. Who knows if this may lead to new breakthroughs in complexity theory?


Fast "by design"
-------- 

While the actual efficiency of a programming language depends on the implementation, some languages are inherently slow, by design. JavaScript, for example, is slower than C: all things equal, its mandatory garbage collector will be an unavoidable extra cost. Formality is, by design, as fast as a language could be. For example, due to linearity (actually, affinity), it doesn't require a garbage collector. Due to the confluence of its interaction-combinator runtime, it can be evaluated in a massively parallel fashion. Due to Elementary Affine Logic (and unlike `BOHM <https://github.com/cls/bohm>`_), it doesn't require bookkeeping machinery for its optimal substitutions. It can be evaluated lazily, just like Haskell. And so on. While Formality is, right now, not as fast as mature languages (after all, those have years of optimizations behind), it will inevitably reach those languages once more mature.

An elegant underlying theory
-------- 

We conjecture that Formality's unique approach to termination allows its type system to have a bunch of powerful features that would otherwise be impossible without making the proof language inconsistent. As an example, it features `Type : Type`, which is powerful and very convenient. It also features mutual type-level recursion, allowing us to exploit self-types to elegantly represent datatypes as their own induction schemes, without needing a complex native datatype system. We're working hard towards proofs of those claims and hope they can be published soon.

```javascript
// Vectors are lists with stactically-known lengths
T Vector <T : Type> {len : Nat}
| vcons {~len : Nat, head : T, tail : Vector(T, len)} & succ(len)
| vnil & zero

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
```javascript

Table of content
=====================================

.. toctree::
   :caption: Language
   :maxdepth: 2
   :numbered:
   
      language
   language/Installation
   language/Hello,-world!
   language/Basic-Features
   language/Datatypes
   language/Box-System
   language/Recursion
   
.. toctree::
    :maxdepth: 2
    :numbered:
    :caption: Techniques

.. toctree::
    :maxdepth: 2
    :numbered:
    :caption: Runtime

    runtime/Formality-Net

.. toctree::
    :maxdepth: 2
    :caption: Tutorial

