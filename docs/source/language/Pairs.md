# Pairs

Formality-Core comes with native pairs. It includes 4 primitives:

syntax | effect
--- | ---
`[a,b]` | Creates a pair with elements `a` and `b`
`fst p` | Extracts the first element of a pair
`snd p` | Extracts the second element of a pair
`get [a,b] = p ...` | Extracts both elements of a pair as `a` and `b` in `...`

### Creating a pair

They can be created with the `[a, b]` syntax. For example, to create a pair with the numbers `3` and `4`, this is what you write:

```javascript
def main:
  [3, 4]
```

Since we don't have `print` yet, pairs can be temporarily used when we want to output multiple results on the console. For example,

```javascript
def main:
  ["Formality-Core was developed in:", 2019]
```

### Getting its elements

You can access the first element of a pair with `fst`, and the second one with `snd`. For example, the program below:

```javascript
def main:
  fst [3, 4]
```

Evaluates to `3`. Since Formality programs are just expressions (terms), you can mix numeric operators and pairs as you'd expect:

```javascript
def main:
  [|fst [3, 4] + 10|, 14]
```

The term above evaluates to `[13, 14]`. You can also extract both elements of a pair simultaneously with a projection, using the `get` syntax:

```javascript
def main:
  get [a, b] = [3, 4]
  |a + b|
```

The program above extracts the elements of `[a,b]`, defining the variables `a = 3`, `b = 4`, and then adds them, resulting in `7`. 

### Nesting

You can also have infinitely nested pairs. For example:

```javascript
def main:
  get [a, b] = [[1, 2], [3, 4]]
  a
```

The program above outputs `[1, 2]`. You can **not** have nested `get`s, though. For example:

```javascript
def main:
  get [[a,b], [c,d]] = [[1, 2], [3, 4]]
  a
```

The program above is a syntax error. In order to extract all 4 elements, you must perform a series of `get`s:

```javascript
def main:
  get [ab, cd] = [[1, 2], [3, 4]]
  get [a, b] = ab
  get [c, d] = cd
  a
```

The program above outputs `1`.
