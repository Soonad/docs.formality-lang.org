## Theorem Proving

This article is a draft.

### Equality

TODO

### Simple proofs

Let's prove a theorem about the boolean not:

```javascript
T Bool
| true
| false

not : {b : Bool} -> Bool
  @ b     ~> Bool
  | true  => false
  | false => true
```


Start with the theorem we want to prove:

```javascript
main : {b : Bool} -> not(not(b)) == b
  ?
```

The type checker complains:

```javascript
Type mismatch.
- Found type... Hole
- Instead of... (not(not(b)) == b)
- When checking ?
- On expression {b} => ?
- With the following context:
- b : Bool
```

Let's pattern-match on `b`.

```javascript
main : {b : Bool} -> not(not(b)) == b
  @ b     ~> not(not(b)) == b
  | true  => ?
  | false => ?
```

Not helpful:

```javascript
Type mismatch.
- Found type... Hole
- Instead of... (not(not(b)) == b)
- When checking ?
- On expression (%b)(~{self} => (not(not(b)) == b), ?)
- With the following context:
- b : Bool
```

Using `self` on the motive:

```javascript
main : {b : Bool} -> not(not(b)) == b
  @ b     ~> not(not(self)) == self
  | true  => ?
  | false => ?
```

Progress, `bs` is specialized `true` on the expected type of the `true` branch:

```javascript
Type mismatch.
- Found type... Hole
- Instead of... (not(not(true)) == true)
- When checking ?
- On expression (%b)(~{self} => (not(not(self)) == self), ?)
- With the following context:
- b : Bool
```

If we reduce both sides, we get the same expression: `{true, false} => true`. In this case, we can use `refl`:

```javascript
main : {b : Bool} -> not(not(b)) == b
  @ b     ~> not(not(self)) == self
  | true  => refl<true>
  | false => ?
```

Progress, compiler now complains about the `false` branch:

```javascript
Type mismatch.
- Found type... Hole
- Instead of... (not(not(false)) == false)
- When checking ?
- On expression (%b)(~{self} => (not(not(self)) == self), refl<true>, ?)
- With the following context:
- b : Bool
```

We can do the same:

```javascript
main : {b : Bool} -> not(not(b)) == b
  @ b     ~> not(not(self)) == self
  | true  => refl<true>
  | false => refl<false>
```

No type error. Our proof is complete!

### Inductive proofs

Let's prove a similar theorem, but for negation on arbitrary-length bit-strings instead of plain booleans:

```javascript
T Bits
| b0 {pred : Bits}
| b1 {pred : Bits}
| be

!bnot*n : !{*bits : Bits} -> Bits
  (@ bits ~> {bnot : {bits : Bits} -> Bits} -> Bits
  | b0    => {bnot} b1(bnot(pred))
  | b1    => {bnot} b0(bnot(pred))
  | be    => {bnot} be)(bnot)
```

Start with the theorem we want to prove:

```javascript
!bnot_theo*n : !{bits : Bits} -> -#(bnot(n))(-#(bnot(n))(bits)) == bits
  ?
  * ?
```


Remember `*` is mandatory on recursive definition to provide the base-case. TODO: explain why the `-#`s on the type. 

The type checker complains:

```javascript
Type mismatch.
- Found type... Hole
- Instead of... bnot(step(n), bnot(step(n), bits)) == bits
- When checking ?
- On expression {bits} => ?
- With the following context:
- n    : Ind
- n    : Ind
- main : {bits : Bits} -> bnot(n, bnot(n, bits)) == bits
- bits : Bits
```

Notice that:

1. It asks for `P(step(n))` instead of `P(n)`.

2. We have, on context, `main`, which gives us `P(n)`.

That's because the body of an recursive is actually the step case of an inductive proof, so all we need to do is, assuming `P(n)`, prove `P(step(n))`!

Let's match against `bits`, using `self` on the motive:

```javascript
!main*n : !{bits : Bits} -> -#(bnot(n))(-#(bnot(n))(bits)) == bits
  @ bits ~> -#(bnot(step(n)))(-#(bnot(step(n)))(self)) == self
  | b0   => ?
  | b1   => ?
  | be   => ?
  * ?
```

Now the complaint becomes:

```javascript
Type mismatch.
- Found type... Hole
- Instead of... bnot(step(n), bnot(step(n), b0(pred))) == b0(pred)
- When checking ?
- On expression {bits} => ?
- With the following context:
- n    : Ind
- n    : Ind
- main : {bits : Bits} -> bnot(n, bnot(n, bits)) == bits
- bits : Bits
- pred : Bits
```

This is better, because now it expects `b0(pred)` instead of just `bits` . This allows the left-side of the equation to be reduced to:

```javascript
b0(bnot(n, bnot(n, pred))) == b0(pred)
```

This is perfect, because we can use the inductive hypothesis to get this same equation, without the `b0`s. As in, we need to go...

```javascript
from :    bnot(n, bnot(n, bs))  == b0(bs)
to   : b0(bnot(n, bnot(n, bs))) ==    bs
```

All we need is to add `b0` on both sides. We can do it with `cong`, from the base libraries:

```javascript
!main*n : !{bits : Bits} -> -#(bnot(n))(-#(bnot(n))(bits)) == bits
  @ bits ~> -#(bnot(step(n)))(-#(bnot(step(n)))(self)) == self
  | b0   => cong(~Bits, ~Bits, ~(-#(bnot(n)))((-#(bnot(n)))(pred)), ~pred, ~b0, ~main(pred))
  | b1   => ?
  | be   => ?
  * ?
```

TODO: should we have a built-in syntax to simplify `cong`?

Now the checker complains about the `b1` case:

```javascript
Type mismatch.
- Found type... Hole
- Instead of... bnot(step(n), bnot(step(n), b1(pred))) == b1(pred)
- When checking ?
- On expression {pred} => ?
- With the following context:
- n    : Ind
- n    : Ind
- main : {bits : Bits} -> bnot(n, bnot(n, bits)) == bits
- bits : Bits
- pred : Bits
```

We can easily complete this proof now:

```javascript
!main*n : !{bits : Bits} -> -#(bnot(n))(-#(bnot(n))(bits)) == bits
  @ bits ~> -#(bnot(step(n)))(-#(bnot(step(n)))(self)) == self
  | b0   => cong(~Bits, ~Bits, ~(-#(bnot(n)))((-#(bnot(n)))(pred)), ~pred, ~b0, ~main(pred))
  | b1   => cong(~Bits, ~Bits, ~(-#(bnot(n)))((-#(bnot(n)))(pred)), ~pred, ~b1, ~main(pred))
  | be   => refl<be>
  * refl<bits>
```

Note that the big difference here, with relation to Agda/Coq proofs, is that, in their cases, since recursive functions are defined by structural recursion, inductive proofs are also defined by recursion on the structure. For example, if we wanted to prove this theorem in Agda, we'd just match the bit-string, prove the base case by reflexivity, and prove the recursve case by calling `main` recursively on `pred`.

In Formality, it **looks** like the proof is the same, but there is a subtle, yet important, difference: under the hoods, we're not actually recursing on the `Bits` structure. Instead, we're folding over `n : Ind`, a datatype capturing the inductive hypothesis on natural numbers. As such, in order to prove that `bnot(n, bnot(n, bits)) == bits` hold for any `n`, we first must prove that it is true for `n == 0`, i.e., when `bits` has a maximum recursion depth of `0` (i.e., is "out-of-gas"), which is true by reflexivity since the function returns `bits` itself. We then prove that, assuming this is true for a maximum recursion depth of `n`, then it is also true for `bits(step(n))`. This is the `step` case, which coincides with Agda's proof.

Proving that kind of inductive theorem on Formality is a little more verbose than in Agda, since you have to wrap the whole proof inside `Ind`, and always tell the compiler what to do when the function "runs out of gas". In exchange, since termination is guaranteed by EAL, there is no "structural recursion" chhcker, so you're allowed to be more flexible in your recursive definitions.
