# Undefined behavior

In languages like C, indexing an array outside of its bounds is [undefined behavior](https://en.wikipedia.org/wiki/Undefined_behavior), i.e., once you do it, you can't assume anything about the behavior or output of your program. Since Formality-Core doesn't have a type system, it has a set of situations for which the behavior is undefined. In some cases, your program may output an expected result even after encountering an undefined behavior, but that can't be relied on. It is the programmer's responsibility to make sure that no undefined behavior happens. 

Any interaction between primitives that was not explicitly defined should be considered undefined behavior. For example, `|"foo" + 3|`, i.e., attempting to add a string and a number, is undefined behavior because there is no reduction rule covering that case. If you run it with the interpreter, it will warn you with a message error. If you run it with interaction nets, anything can happen. For this reason, it's always good to develop and debug with the interpreter. 

The `cpy` primitive, in particular, is worth noting. If you try to copy a lambda with `cpy`, the interpreter will warn you with a runtime error message, but, if you try to evaluate it with interaction nets, it will actually output the "expected" result in many cases. For example:

```javascript
def main:
  cpy f = {x}x
  f
```

Will output `[{x}x, {x}x]` if evaluated with interaction nets. This doesn't mean you can use `cpy` to copy anything other than native numbers. Here, for example, is a very problematic term:

```javascript
def main:
  cpy square = {x} 
    cpy x = x
    |x * x|
  [(square 2), (square 3)]
```

If you try to evaluate it with interaction nets, it will output `[6,6]`, which is not the "expected" result! The technical reason is that this program has two duplication nodes of identical labels pointing to themselves. They interfere destructively, getting the interaction net system into a corrupt state which may behave unpredictably. 

Similar situations can happen when trying to duplicate a term that isn't inside a box. Boxes are fully erased after compilation, so, interaction nets won't display a nice error message in those cases. Instead, they will go ahead copying the unboxed lambda, and anything can happen afterward. Be very cautious with copies and always use the interpreter to make sure you're not trying to copy an unboxed value!