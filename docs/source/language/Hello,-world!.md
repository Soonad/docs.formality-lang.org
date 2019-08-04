# Hello, World!

## Running a program

This is the "Hello, World!" in Formality:

```javascript
main : Output
  print("Hello, world!")
```

Save this as `main.fm` and run it with `fm hello.main`. This will use the interpreter to evaluate `main` and print its result. The first time you do it may take a while, since `fm` will load the base libraries and place them on `fm_modules`. If everything works, you should see "Hello, world!" in your terminal.

Despite the name, `print`, Formality is a pure functional language: it has no notion of state, input, or output. Instead, when you type `fm <file>.<term>`, Formality will just evaluate the `main` definition inside the `hello.fm` file, stringify it and output the result. Here, `Output` is just a type defined on the base libraries that tell the CLI to pretty-print a string, and `print` is just a `String -> Output` function defined on the base libraries. `main` doesn't need to be an `Output`, though. It can be anything. For example:

```javascript
main : String
  "Hello, world!"
```

If you evaluate this program, it will output:

```
{cons, nil} => cons(1819043144,
{cons, nil} => cons(1998597231,
{cons, nil} => cons(1684828783,
{cons, nil} => nil)))
```

Which is the λ-encoded version of the "Hello, world!" string as an UTF-8 buffer. `{a, b, ...} => ...` is the syntax for a lambda. Formality will fully evaluate terms, even inside lambdas, in order to show their normal forms.

Note that type annotations are optional. For example, this works fine:

```
main
  print("Hello, world!")
```

Of course, annotating types is always recommended, and, in order to prove a theorem, every function used in your proof must be well-typed.

Note that, by default, `fm` runs on the debug mode, using an interpreter. To run a program using interaction nets and optimal reductions, use the `-o` flag.

## Type-checking and Theorem Proving

You can execute the type-checker with `fm -t <file>.<term>`. If you use it on the first "Hello, World!", it will print `Output`, which means `main` is a well-typed program of type `Output`. Due to the richness of Formality's type-system, well-typed programs can also be seen as mathematical proofs, so the type-checker can be seen as a formal proof checker. 
