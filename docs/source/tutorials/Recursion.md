# Recursion

## Boxed definitions

Since bounded recursive functions are so common, Formality has built-in syntax for them, relying on "boxed definitions". To make a boxed definition, prepend `#` to its name. That has two effects. First, the whole definition is lifted to `level 1`. Second, it allows you to use boxed definitions inside `<>` as the parser will automatically unbox them for you. For example, instead of this:

```javascript
foo : !Word
  #40

bar : !Word
  #2

main : !Word
  dup foo = foo
  dup bar = bar
  # foo + bar
```

We could write:

```javascript
#foo : !Word
  40

#bar : !Word
  2

#main : !Word
  <foo> + <bar>
```

Both programs are the same, except the later is shorter.

## Recursion

Formality allows you to turn a boxed definition into a recursive function by appending `*N` to its name (where `N` is a new variable, which will track how many times the function was called), and adding a "halt-case" with a `halt: term` on the last line (where `term` is an expression that will be returned if the function hits its call limit). So, for example, a factorial function could be written as:

```javascript
#fact*N : ! {i : Word} -> Word
  if i .= 0:
    1
  else:
    i * fact(i - 1)
halt: 0
```

This is not much different from the usual `fact` definition, except we explicitly set the "halt-case" to be `0` on the last line. That means that, if the function "runs out of gas", it will stop and return `0` instead. As a shortcut, if your "halt-case" is simply one of the function's argument, you can write the `*` on it instead, as in, `fact*N ! {*i : Word} -> Word`. To call it, you must set an explicit max call limit with `*N`:

```javascript
main : !Word
  dup f = fact*100
  # f(12)
```

Or, with boxed definitions:

```javascript 
#main : !Word
  <fact*100>(12)
```

The `f*N` syntax configures the call limit of a recursive function. Here, we used `100`. Note this is actually just a shortcut for a function application: we could have written `fact(*100)` instead. We could also have omitted the number as in `<fact*>(x)`, which would default to `2^256-1`. This limit is so absurdly large that, for all practical purposes, our functions are no less powerful than the ones found in other languages. After all, `2^256-1` is so large that no real computer could reach this amount of calls anyway. In fact, the entire observable universe has less particles than that!

**Obs: Work in progress**

Cover things like:

- Simple recursive functions and boxed definitions

    ```javascript
    #double*N : !{case halt n : Nat} -> Nat
    | succ => succ(succ(double(n.pred)))
    | zero => zero

    #double.example : !Nat
      <double*>(succ(zero))
    ```

- Polymorphic recursive functions with level-0 parameters

    ```javascript
    #map*N : {~A : Type, ~B : Type, f : !A -> B} -> ! {case list : List(A)} -> List(B)
    | cons => cons(~B, f(list.head), map(list.tail))
    | nil  => nil(~B)
    halt: nil(~B)

    #map.example : !List(Word)
      <map*(~Word, ~Word, #{x} x + 1)>(Word$[1,2,3,4])
    ```

- Indexed recursive functions using `N`

```javascript
... vector stuff, fin stuff, etc...
```


- Structural recursive proofs with Bounds

```javascript
// ∀ n . n < n+1
#less_than_succ*N : !{n : Nat, bound : Bound(n, N)} -> Less(n, succ(n))
  (case/Bound bound
  | bound_succ => {e}
    let r0 = cong_unstep(~i, ~N, ~e)
    let r1 = less_than_succ(n, k :: rewrite x in Bound(n, x) with r0)
    let r2 = less_succ(~n, ~succ(n), r1)
    r2
  | bound_zero => {e} less_zero(~zero)
  : {e : i == step(N)} -> Less(n, succ(n)))(refl(~step(N)))
halt: absurd(absurd_bound(n, bound), ~Less(n, succ(n)))
```


etc.
