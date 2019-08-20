# Hello, World!

## Running a program

This is the "Hello, World!" in Formality:

```javascript
import Base@0 open

main : Output
  print("Hello, world!")
```

Save this as `hello.fm` and run it with `fm hello/main`. This will use the interpreter to evaluate `main` and print its result. If everything works, you should see "Hello, world!" in your terminal.

## Imports

The file starts with the `import Base@0 open` directive. That loads the official base libraries. This includes definitions such as `print`, `Output`, as well as helpers used by the string literal syntax. The `@0` after `Base` indicates its global version. You can use Formality without it, but you won't get certain features such as string literals.

You can also load local files. For example, save an `Answers.fm` file in the same directory as `hello.fm`, with the following contents:

```javascript
import Base@0 open

everything : String
  "42"
```

Then change the `hello.fm` file to:

```javascript
import Base@0 open
import Answers

main : Output
  print(Answers/everything)
```


This will output `42`, as expected. You can also avoid the `Answers` namespace with an `open` directive:

```javascript
import Base@0 open
import Answers open

main : Output
  print(everything)
```

Or with a qualifed import, using `as`:


```javascript
import Base@0 open
import Answers as A

main : Output
  print(A/everything)
```

## Global imports

Formality's "package system" is very simple. A file can be saved globally with `fm -s file`. This will give it a unique name with a version, such as `file@7`. Once given an unique name, a file contents will never change, so `file@7` will always refer that exact file. As soon as it is saved globally, you can import it from any other computer. For example, remove `Answers.fm` and change `hello.fm` to:

```javascript
import Base@0 open
import Answers@0 open

main : Output
  print(everything)
```

This will load `Answers@0.fm` inside the `fm_modules` directory and load it as usual.

Right now, global imports are uploaded to our servers, but, in a future, they'll upload files to a decentralized storage such as IPFS/Swarm, and give it an unique name using Ethereum's naming system. Alternatively, you could write a module directly to the blockchain. This will allow its contents to be available permanently, with no downtime.

## Evaluating

Formality is a pure functional language: it has no global state or built-in IO. Instead, when you type `fm <file>/<term>`, Formality will just evaluate the `main` definition inside the `hello.fm` file to its "normal form", stringify it and output the result. As such, `Output : Type` and `print : {str : String} -> Output` are really just base-library utilities that tell the CLI to pretty-print a string. `main` doesn't need to be an `Output`, though. It can be anything. For example, if we evaluate this program:

```javascript
import Base@0 open

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
import Base@0 open

main
  "Hello, world!"
```

By default, `fm` runs on the debug mode, using an interpreter. One of the most interesting aspects of Formality is that its programs can be evaluated optimally, as explained on [this](https://medium.com/@maiavictor/solving-the-mystery-behind-abstract-algorithms-magical-optimizations-144225164b07) post, using an efficent graph-reduction system based on [interaction combinators](https://arxiv.org/abs/0906.0380). To run your program using the optimal reduction algorithm, use the `-o` flag.

## Type-checking

Formality is also a proof language, which means it includes a powerful type-checker. You can execute i with `fm -t <file>/<term>`. If your program is well-typed, you'll see its type. Otherwise, you'll see an error message explaining what is wrong. Due to [Curry-Howard's correspondence](https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence), a Formality type can be viewed as theorem, and a terms of that type as its proof. This allows us to do encode extremelly rich, static guarantees about our programs, as will be explained later.
