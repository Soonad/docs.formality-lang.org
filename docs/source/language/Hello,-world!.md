# Hello, World!

## Running a program

This is the "Hello, World!" in Formality:

```javascript
main : Output
  print("Hello, world!")
```

Save this as `main.fm` and run it with `fm hello.main`. This will use the interpreter to evaluate `main` and print its result. The first time you do it may take a while since `fm` will load the base libraries and place them on `fm_modules`. If everything works, you should see "Hello, world!" in your terminal.

Formality is a pure functional language: it has no global state or built-in IO. Instead, when you type `fm <file>.<term>`, Formality will just evaluate the `main` definition inside the `hello.fm` file, stringify it and output the result. As such, `Output : Type` and `print : {str : String} -> Output` are really just base-library utilities that tell the CLI to pretty-print a string. `main` doesn't need to be an `Output`, though. It can be anything. For example, if we evaluate this program:

```javascript
main : String
  "Hello, world!"
```

It will output:

```
{cons, nil} => cons(1819043144,
{cons, nil} => cons(1998597231,
{cons, nil} => cons(1684828783,
{cons, nil} => nil)))
```

Which is the λ-encoded version of the "Hello, world!" string (as an UTF-8 buffer). Formality will fully evaluate terms, even inside lambdas, in order to show their normal forms. 

Type annotations are optional. For example, this works fine:

```
main
  print("Hello, world!")
```

By default, `fm` runs on the debug mode, using an interpreter. To run a program using interaction nets and optimal reductions, use the `-o` flag.

## Type-checking

You can execute the type-checker with `fm -t <file>.<term>`. If you use it on the first "Hello, World!", it will display `Output`, which means `main` is a well-typed program of type `Output`. Due to the richness of Formality's type-system, well-typed programs can also be seen as mathematical proofs, so the type-checker can be seen as a formal proof checker. 
