# If and Cpy

Formality-Core has 2 extra primitives to deal with numbers:

syntax | effect
--- | ---
`if n p` | If `n == 0`, evaluates to `snd p`, else, evaluates to `fst p`
`dup x = n ...` | Makes multiple copies of the number `n` as `x`

It also has an alternative syntax for the creation of pairs:

syntax | effect
--- | ---
`then: a else: b` | Equivalent to `[a, b]`

## If

The reason for `if` is to allow you to branch based on the value of a number, otherwise, that wouldn't be possible. Note that `if` is a binary operation that receives a number `n`, a pair `p`, and returns either `fst p` or `snd p`.  This allows you to branch based on the value of a number. Together with the `then: a else: b` syntax, it allows one to write `if-then-else` statement similar to traditional languages:

```javascript
def main:
  let age = 30

  if |age < 18|
  then:
    "boring teenager"
  else:
    "respect your elders!"
```

## Cpy

There is also a syntax to clone a number, creating multiple copies of it:

```javascript
def main:
  cpy x = 42
  [x, |x + x|]
```

The program above outputs `[42, 84]`. This primitive is necessary because, in Formality-Core, any fixed-size data structure can be natively copied without "boxes" (which will be explained next), with the only exception being native numbers. This primitive fixes this, allowing numbers to be copied without boxes.

