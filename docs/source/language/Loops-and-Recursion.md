# Loops and Recursion

Formality-Core uses Church-Encodings to implement bounded loops and recursion. Church-Encodings are beyond the scope of this tutorial, but, for now, you can use this helper `for` function to perform loops, as illustrated below:

```javascript
def for: {numb init loop stop}
  dup init = init
  dup exec = (numb loop)
  dup stop = stop
  # (stop (exec init))

def main:
  let init = # 1
  let loop = # {x} |x * 2|
  let stop = # {x} ["2**10 is:", x]
  (for ~10 init loop stop)
```

Notice the `~10` above. This built-in syntax desugars to Church-Encoded Nat `10`, which is used by `for` to "fuel" the iterations of the loop. Just for the sake of illustration, `~3` becomes:

```javascript
{succ}
  dup succ = succ
  # {zero} (succ (succ (succ zero)))
```

## Bounded Recursion

The same idea above can be used to implement bounded recursion. This program, for example, converts a Scott-Encoded Tree to a Church-Encoded Tree:

```javascript
def tree_fold: {size}
  let func = {go tree}
    let case_node = {a b go}
      get [go, a] = (go a)
      get [go, b] = (go b)
      [go, (FNode a b)]
    let case_leaf = {val go}
      [go, (FLeaf val)]
    (tree case_node case_leaf go)
  dup fold = (size #func)
  # {tree} (snd (fold {tree}[{x}x,tree] tree))

def main:
  let tree = (Node (Node (Leaf #1) (Leaf #2)) (Node (Leaf #3) (Leaf #4)))
  let size = ~7
  dup fold = (tree_fold size)
  # (fold tree)
```

Here, `Node` and `Leaf` are Scott-Encoded constructors, and `FNode` and `FLeaf` are Church-Encoded constructors, as defined on the last section. Notice how `tree_fold` needed to know the size of the input tree (`7`). This is because the Church-Encoded Nat `7` is used to unroll `func` up to a depth of `7`, essentially emulating a recursive function with a bounded number of iterations. With this technique, arbitrary recursive functions can be defined, as long as you put a "gas limit" on how much work they can perform. Again, an in-depth explanation of how those work is outside the scope of this reference for now, but we'll be complementing it soon.
