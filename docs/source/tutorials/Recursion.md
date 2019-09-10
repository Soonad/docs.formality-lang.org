# Recursion

## Boxed definitions

Since bounded recursive functions are so common, Formality has built-in syntax for them, relying on "boxed definitions". To make a boxed definition, simply annotate it with a `!` instead of a `:`. That has two effects. First, the whole definition is lifted to `level 1`. Second, it allows you to use boxed definitions inside `<>`s: the parser will automatically unbox them for you. For example, instead of this:

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
!foo : !Word
  40

!bar : !Word
  2

!main : !Word
  <foo> + <bar>
```

Both programs are the same, except the later is shorter.

## Recursion

Formality allows you to turn a boxed definition into a recursive function by appending `*N` to its name (where `N` is a new variable, which will track how many times the function was called), and adding a "halt-case" with a `* term` on the last line (where `term` is an expression that will be returned if the function hits its call limit). So, for example, a factorial function could be written as:

```javascript
!fact*n : ! {i : Word} -> Word
  if i .= 0:
    1
  else:
    i * fact(i - 1)
  * 0
  
!main : !Word
  <fact*100>(12) // 100 is the call limit; write `<fact*>` to use 2^256-1
```

This is not much different from the usual `fact` definition, except we explicitly set the "halt-case" to be `0` on the last line. That means that, if the function "runs out of gas", it will stop and return `0` instead. As a shortcut, if your "halt-case" is simply one of the function's argument, you can write the `*` on it instead, as in, `fact ! {*i : Word} -> Word`. 

The maximum recursion depth of the `Ind` defined by the base library is `2^256-1`. In other words, despite being terminating, in practice, Formality is capable of doing anything a Turing-complete language could do. After all, no language could perform more than `2^256-1` calls without getting a stack-overflow, or exhausting your computer's CPU/memory.

## (TODO)

Cover things like:

- Simple recursive functions and boxed definitions

    ```javascript
    !double*N : !{case *n : Nat} -> Nat
    | succ => succ(succ(double(n.pred)))
    | zero => zero

    !double.example : !Nat
      <double*>(succ(zero))
    ```

- Polymorphic recursive functions with level-0 parameters

    ```javascript
    !map*N : {~A : Type, ~B : Type, f : !A -> B} -> ! {case list : List(A)} -> List(B)
    | cons => cons(~B, f(list.head), map(list.tail))
    | nil  => nil(~B)
    * nil(~B)

    !map.example : !List(Word)
      <map*(~Word, ~Word, #{x} x + 1)>(Word$[1,2,3,4])
    ```

- Indexed recursive functions using `N`

```javascript
... vector stuff, fin stuff, etc...
```


- Structural recursive proofs with Bounds

```javascript
// ∀ n . n < n+1
!less_than_succ*N : !{n : Nat, bound : Bound(n, N)} -> Less(n, succ(n))
  (case/Bound bound
  | bound_succ => {e}
    let r0 = cong_unstep(~i, ~N, ~e)
    let r1 = less_than_succ(n, k :: rewrite x in Bound(n, x) with r0)
    let r2 = less_succ(~n, ~succ(n), r1)
    r2
  | bound_zero => {e} less_zero(~zero)
  : {e : i == step(N)} -> Less(n, succ(n)))(refl(~step(N)))
  * absurd(absurd_bound(n, bound), ~Less(n, succ(n)))
```


etc.
