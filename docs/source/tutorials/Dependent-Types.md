## Dependent Types

**Obs: Work in progress**

Cover things like:

- Returning a different type based on the function's input

    ```haskell
    make_unit_or_word : {x : Bool} -> iff(x, ~Type, Unit, Word)
      case/Bool x
      | true  => unit
      | false => 42
      : iff(self, ~Type, Unit, Word)
    ```

- Manipulating equalities

    ```haskell
    b_is_true : {a : Bool, b : Bool, b_is_a : b == a, a_is_true : a == true} -> b == true
      b_is_a :: rewrite a in b == a with a_is_true
    ```

- Specifying precise algorithm (for work outsourcing)

    ```haskell
    // "I want a function that receives a bool and returns a different bool"
    Specification : Type
      {a : Bool} -> [b : Bool, ~Not(a == b)]
      
    // "This is a valid implementation of the Specification"
    not : Specification
      case/Bool a
      | true  => [false , ~true_isnt_false]
      | false => [true  , ~{e} true_isnt_false(sym(~e))]
      : [b : Bool, ~Not(self == b)]
    ```

- Exploiting impossible cases to improve function's interface

    ```haskell
    // If we know that a Maybe isn't none, we can extract its contents
    extract : {~A : Type, x : Maybe(A), not_none : Not(x == none(~A))} -> A
      case/Maybe x
      | just => val
      | none => absurd(not_none(refl(~none(~A))), ~A)
      : A
    ```

etc.
