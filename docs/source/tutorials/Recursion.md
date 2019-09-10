# Recursion

(TODO)

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
    !map*n : {~A : Type, ~B : Type, f : !A -> B} -> ! {case list : List(A)} -> List(B)
    | cons => cons(~B, f(list.head), map(list.tail))
    | nil  => nil(~B)
    * nil(~B)

    !map.example : !List(Word)
      <map*(~Word, ~Word, #{x} x + 1)>(Word$[1,2,3,4])
    ```

- Advanced, Agda-like structural recursion with Bounds

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
