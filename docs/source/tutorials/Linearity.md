## Linearity

Formality's approach to termination is what makes it different from other proof languages like Agda, Idris and Coq. Instead of having native datatypes, structural recursion and so on, we go deeper and change the underlying logic of the system from intuitionist to elementary affine. This is responsible for all the claimed benefits of Formality: optimal reductions, no garbage-collection, massive parallelism, elegant inductive types and so on. But it comes with a huge tradeoff: our lambdas are affine, i.e., bound variables can't be used more than once. This limits what we can do in multiple ways. For example, we can't write a function that `and`s a `Bool` with itself:

```javascript
import Base@0

self_and : {x : Bool} -> Bool
  and(x, x)

main : Bool
  self_and(true)
```

If we try to check the type using `fm -t <file_name>/main`, we get an error:

```shell
Lambda variable `x` used more than once in:
{x} => and(x, x)
```

There are multiple ways to avoid this situation.

### Reuse the variable on multiple branches

If the "duplicated" variable is used in different branches, as in:

```javascript
import Base@0

// Negates `b` if `a` is true
not_if : {a : Bool, b : Bool} -> Bool
  case/Bool a
  | true  => not(b)
  | false => b
  : Bool

main : Bool
  not_if(true, true)
```

Then, we can avoid copying `b` with a clever trick: return a lambda on each branch. Like this:

```javascript
import Base@0

// Negates `b` if `a` is true
not_if : {a : Bool, b : Bool} -> Bool
  (case/Bool a
  | true  => {b} not(b)
  | false => {b} b
  : Bool -> Bool)(b)

main : Bool
  not_if(true, true)
```

Notice that instead of using `b` directly, the `case/Bool` expression returns, in each case, a different lambda, which is then applied to a single `b`. This prevents using it more than once, and is allowed. This technique is extremelly important for Formality development. 

### Use case'd arguments

While the trick above is powerful, it increases code complexity. Fortunatelly, if you use `case`'d arguments instead of `case/T` expressions, Formality will automatically do it for you. For example, this works:

```javascript
import Base@0

// Negates `b` if `a` is true
not_if : {case a : Bool, b : Bool} -> Bool
| true  => not(b)
| false => b

main : Bool
  not_if(true, true)
```

And is much less verbose than the solution above. In practice, this features allows you to use a variable once per branch of the function, instead of once per function.

### Make an explicit copy

For `Word`s in particular, there is a native `cpy` operation that copies it as many times as desired:

```javascript
square : {x : Word} -> Word
  cpy x = x
  x * x

main : Word
  square(7)
```

When a `Word` is an argument of a top-level function, then you don't even need to add `cpy`. Formality does it for you. I.e., this works:

```javascript
square : {x : Word} -> Word
  x * x

main : Word
  square(7)
```

For other types, you can write an auxiliary `copy` function:

```javascript
import Base@0

// An explicit copying function
copy_bool : {b : Bool} -> [:Bool, Bool]
  case/Bool b
  | true  => [true, true]
  | false => [false, false]
  : [:Bool, Bool]

and_itself : {b : Bool} -> Bool
  get [b0, b1] = copy_bool(b) // performs an explicit copy
  and(b0, b1)

main : Bool 
  and_itself(true)
```

This is also a very important technique. So, in short, when you need to use a variable more than once, this is what you should do:

1. Is it a `Word`? If so, just `cpy` it.

2. Is the usage in different branches? Then manually return lambdas (or use `case`'d arguments).

3. Otherwise, copy the structure with an explicit `copy` function.

### Use boxes

Formality has another primitive for deep-copying values, boxes. When dealing with data, though, you almost never want to use boxes to perform copies, due to the stratification condition, which essentially segregates the language in levels, blocks communication from higher to lower levels. Regardless, they can still be useful sometimes. See, for example, how `map` is defined for lists:

```javascript
#map*n : {~A : Type, ~B : Type, f : !A -> B} -> ! {case list : List(A)} -> List(B)
| cons => cons(~B, f(list.head), map(list.tail))
| nil  => nil(~B)
halt: nil(~B)
```

Here, `f` is duplicated on level `0`, allowing it to be used multiple times on level `1`. The tradeoff is that, relative to programs on level `1`, `f` must be seen as static. So, if an user input arrives on level `0`, for example, it can't affect the shape of `f`.

Boxes really shine when implementing control flow like loops and recursion. This will be explained in more details on the next section.
