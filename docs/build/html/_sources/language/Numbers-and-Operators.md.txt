# Numbers and Operators

Formality-Core includes native unsigned 32-bit numbers and numeric operations. Numbers are written as usual. Test it by running:

```javascript
def main:
  1234
```

Numeric operations are similar to the ones found in most programming languages, except that there is no operator precedence. Instead, **all** numeric operations must be wrapped by a `||`. For example, to compute `2 + 2`, you'd write:

```javascript
def main:
  |2 + 2|
```

And the dot product of `[1,2,3]` and `[4,5,6]` would be written as:

```javascript
def main:
  |||1 * 4| + |2 * 5|| + |3 * 6||
```

While slightly inconvenient, this allows Formality-Core's parser to avoid backtracking, so is a welcome feature given that aesthetics isn't its main goal. Here is a list of all numeric operations available:

name | syntax | JavaScript equivalent
--- | --- | ---
|addition | `\|x + y\|` | `(x + y) >>> 0`
|subtraction | `\|x - y\|` | `(x - y) >>> 0`
|multiplication | `\|x * y\|` | `(x * y) >>> 0`
|division | `\|x / y\|` | `(x / y) >>> 0`
|modulus | `\|x % y\|` | `(x % y) >>> 0`


exponentiation | `\|x ** y\|` | `(x ** y) >>> 0`
bitwise-and | `\|x & y\|` | `x & y`
bitwise-or | `\|x \| y\|` | `x \| y`
bitwise-xor | `\|x ^ y\|` | `x ^ y`
bitwise-not | `\|x ~ y\|` | `~y`
bitwise-right-shift | `\|x >> y\|` | `x >>> y`
bitwise-left-shift | `\|x << y\|` | `x << y`
greater-than | `\|x > y\|` | `x > y ? 1 : 0`
less-than | `\|x < y\|` | `x < y ? 1 : 0`
equals | `\|x == y\|` | `x === y ? 1 : 0`
