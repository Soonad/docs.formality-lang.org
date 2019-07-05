# Dups and Boxes

Formality-Core includes 2 primitives for explicit, deep copying of arbitrary terms:

syntax | effect
--- | ---
`# a` | Puts `a` inside a box
`dup x = #a ...` | Unwraps `a` from a box, makes `n` copies of it as `x`

In order to explain how those work, let's make a quick detour and explain about complexity.

## Measuring complexity

As mentioned, `let`s and `def`s are very useful for improving your code organization, but they have no computational effect, i.e., they disappear after parsing. This may cause your program to duplicate work. For example:

```javascript
def main:
  let num = ||1 + 2| + |3 + 4||
  [num, num]
```

The program above is equivalent to:

```javascript
def main:
  [||1 + 2| + |3 + 4||, ||1 + 2| + |3 + 4||]
```

And, as such, wastefully evaluates `||1 + 2| + |3 + 4||` twice. One of the nice things about Formality-Core is that it has a very clear cost model, so we can precisely measure this problem. Save the program above and run it with `fmc -ns main`. The `-ns` flags ask `fmc` to run your program with interaction net system (instead of interpreter), and to output the number of "rewrites" needed to fully execute it. Rewrites are lightweight, constant-spacetime operations. In other words, this number is a very good measure of the complexity of the program. A program that takes `400` rewrites is roughly 2x slower than one that takes `200`, and so on. The program above needs `12` rewrites:

```javascript
$ fmc -ns main
[10,10]
{"rewrites":12,"loops":6,"max_len":8}
```

In order to avoid this waste and copy/clone values efficiently, Formality-Core includes a boxed duplication system, which we'll explain now. The first thing to learn about is boxes. 

## Boxes

Boxes are like containers with only one element. You can put any term inside a box using the `#` syntax. For example:

```javascript
def main:
  # "I'm the man in the box"
```

In the program above, the string literal is inside of a box. You can put boxes anywhere you want to. For example:

```javascript
def main:
  # [##1, [2, #"a cat"]]
```

The program above puts 4 boxes in different places. If we use the analogy that *"a box is like an array with one element"*, then it could be written in Python as:

```python
# Python "equivalent"
def main():
  print ([[1]], (2, ["a cat"]))
```

## Duplications

The reason boxes exist is to enable Formality-Core's explicit duplications, which are essentially deep copies of arbitrary values. To perform those, you must use the `dup` syntax:

```javascript
def main:
  dup num = #||1 + 2| + |3 + 4||
  # [num, num]
```

As you can see, this is very similar to the first program presented, with two differences: the `dup` keyword is used instead of `let`, and there is a box (`#`) wrapping the duplicated value and the result. The effect of `dup` is that it takes a boxed value (in this case, `#||1 + 2| + |3 + 4||`), unboxes it (obtaining `||1 + 2| + |3 + 4||`) and replaces every occurrence of `num` by a copy. The result of this program is, ignoring the box, the same as:

```javascript
$ fmc -ns main
# [10,10]
{"rewrites":7,"loops":5,"max_len":6}
```

Except that now it needed roughly half of the rewrites and, thus, it is twice as fast. This is because duplications are done optimally, so, in this case, the additions were performed before duplicating. This is, of course, a simple example, but in more complex, highly functional programs, clever usage of `dup` can amount to huge, asymptotical speedups. 

## The stratification condition

Sadly, in order to use `dup`, there is an important condition that must be met:

> The number of boxes wrapping a term can never change through the program's evaluation.

This is the stratification condition and it essentially causes Formality-Core programs to be split into "layers", based on how boxes are placed. To explain it, let's present an example. Mind the code below:

```javascript
def main:
  [0, [#1, #[#2, ##3]]]
```

Here, the number `0` isn't wrapped by any box. As such, it is on `layer 0`. The number `1` is wrapped by one box, so it is on `layer 1`. The entire `[#2, ##3]` pair is wrapped by one box, so, `2` is wrapped by 2 boxes in total and, thus, is on `layer 2`, while `3` is wrapped by 3 boxes in total and, thus, is on `layer 3`. 

What the stratification condition says is that a term can never migrate from a layer to another. This means that the program below is not legal:

```javascript
def main:
  dup num = # 42
  [num, num]
```

Because, before reduction, `42` is on `layer 1`, but, after reduction, it would be on `layer 0`. This can't happen. If you try to run the program above, it will present a compile-time error. Similarly, the program below is illegal too:

```javascript
def main:
  dup num = # 42
  ## [num, num]
```

Because, if we evaluated it, `42` would migrate from `layer 1` to `layer 2`, which can't happen. Trying to run it would result in a compile-time error too. This program is correct:

```javascript
def main:
  dup num = # 42
  # [num, num]
```

Because each occurrence of `num` is on `layer 1`, which is the same layer where `42` is from. This program is also correct:

```javascript
def main:
  dup num = # 42
  [#num, #num]
```

Because, once again, each occurrence of `num` is on `layer 1`. This means you're able to place boxes as you wish, as long as it doesn't break the stratification condition. 

The stratification condition is what allows Formality-Core to be evaluated by the fastest interaction-net reduction algorithm known, dispensing the expensive "bookkeeping mechanism" used by other optimal evaluators.