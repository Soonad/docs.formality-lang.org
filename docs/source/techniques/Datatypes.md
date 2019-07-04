# Datatypes

Most traditional functional languages of the Haskell/ML family feature algebraic datatypes or ADTs. Those are essentially the bread-and-butter of functional programming. From Lists to Trees to Monads, every data structure in those languages is an ADT. Formality-Core does not include native ADTs; instead, it provides the building blocks for the user to assemble their own datatypes. This is important when it comes to interaction net compilation because the same data structure could be represented in different ways, with different runtime characteristics, each one having benefits and drawbacks. Being explicit about them allows the programmer to have control over the runtime behavior of their programs.

## Scott-Encoding

The Scott-Encoding represents ADTs as a series of lambdas. It doesn't involve boxes and behaves almost identically to traditional ADTs. The advantage of Scott-Encoded datatypes is that they have O(1) elimination, i.e., pattern-matching is a constant-time operation. The disadvantage is that, since Formality-Core is terminating, you can't iterate/recurse over them directly; instead, you need to use Church-Encoded helpers. The best way to explain a λ-encoding is by example and comparison, so, here are bools, pairs, and trees, with their Haskell-equivalents:

```javascript
// ::::::::::
// :: Bool ::
// ::::::::::

// data Bool = True | False
// bool_match x true false = case x of { True => true; False => false }

def True       : {true false} true
def False      : {true false} false
def bool_match : {x true false} (x true false)

// ::::::::::
// :: Pair ::
// ::::::::::

// data Pair a b = Pair a b
// pair_match x pair = case x of { Pair a b => pair a b }

def Pair       : {a b pair} (pair a b)
def pair_match : {x pair} (x {a b} (pair a b))

// ::::::::::
// :: Tree ::
// ::::::::::

// data Tree a = Node (Tree a) (Tree a) | Leaf a
// tree_match x node leaf = case x of { Node a b => node a b; Leaf val => leaf val }

def Node       : {a b node leaf} (node a b)
def Leaf       : {val node leaf} (leaf val)
def tree_match : {x node leaf} (x {a b} (node a b) {val} (leaf val))

// ::::::::::::::
// :: Examples ::
// ::::::::::::::

def main:

  let bool_ex =
    let bool       = True
    let true_case  = "Bool is true" 
    let false_case = "Bool is false"
    (bool_match bool true_case false_case)

  let pair_ex =
    let pair      = (Pair 4 5)
    let pair_case = {a b} ["Pair sum is:", |a + b|] 
    (pair_match pair pair_case)

  let tree_ex =
    let tree      = (Node (Leaf 42) (Leaf 64))
    let node_case = {a b} "Tree is a Node"
    let leaf_case = {val} "Tree is a Leaf"
    (tree_match (Node (Leaf 42) (Leaf 64)) node_case leaf_case)

  [bool_ex,
  [pair_ex,
    tree_ex]]
```

Thanks to the Scott-Encoding, any Haskell or Agda program that doesn't involve duplications or recursion can be translated directly to Formality-Core without changes. For iteration and recursion, we need the Church-Encoding.

## Church-Encoding

The Church-Encoding is like a materialization of a Haskell fold. It involves boxes, and is used for bounded iteration and recursion. Without Church-Encodings, Formality-Core would be a boring language where all its programs could do is move/permutate stuff around and stop in linear time. They're only required for variable-size (i.e., recursive) datatypes, since, on the fixed-size case, Church and Scott are identical. Once again, they're better explained through examples:

```javascript
// ::::::::::
// :: List ::
// ::::::::::

// data List a = Cons a (List a) | Nil
// list_fold x cons nil = case x of { Cons x xs => cons x (list_fold xs succ zero); Nil => nil }

def Nil: {cons nil}
  dup cons = cons
  dup nil  = nil
  # nil

def Cons: {x xs cons nil}
  dup cons = cons
  dup nil  = nil
  dup x    = x
  dup xs   = (xs #cons #nil)
  # (cons x xs)

def fold_list: {x cons nil}
  (x cons nil)

// ::::::::::
// :: Tree ::
// ::::::::::

// data Tree a = Node (Tree a) (Tree a) | Leaf a
// tree_fold x node leaf = case x of { Node a b => node (tree_fold a node leaf) (tree_fold b node leaf); Leaf val => leaf val }

def Node: {a b node leaf}
  dup node = node
  dup leaf = leaf
  dup a    = (a #node #leaf)
  dup b    = (b #node #leaf)
  # (node a b)

def Leaf: {val node leaf}
  dup node = node
  dup leaf = leaf
  dup val  = val
  # (leaf val)

def fold_tree: {x node leaf}
  (x node leaf)

// ::::::::::::::
// :: Examples ::
// ::::::::::::::

def main:

  let list_ex =
    let list      = (Cons #1 (Cons #2 (Cons #3 (Cons #4 Nil))))
    let fold_cons = # {x xs} |x + xs|
    let fold_nil  = # 0
    (fold_list list fold_cons fold_nil)

  let tree_ex =
    let tree      = (Node (Node (Leaf #1) (Leaf #2)) (Node (Leaf #3) (Leaf #4)))
    let fold_node = # {a b} |a + b|
    let fold_leaf = # {val} val
    (fold_tree tree fold_node fold_leaf)

  [list_ex, tree_ex]
```

Notice how, unlike on the Scott-Encoded versions, here we are able to fold over the structures, summing all their contained values. Moreover, due to the use of boxes, elements contained by a Church-Encoded structure must always be one layer above than them, which, by consequence, means that results of folds are one layer above too.


