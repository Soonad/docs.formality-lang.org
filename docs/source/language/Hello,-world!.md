# Hello, World!
From now on, we will explain each of Formality-Core's features. After we're done, we'll explain how they relate with optimal reductions, interaction nets, and functional compilers. To get started, let's print a `"Hello, world"`. Differently from traditional languages, Formality-Core is a pure expression language. It has no statements, which means no `"print"`. So, how do we get an output to the console? Simple: we ask `fmc` to evaluate a single expression, and consider its result as the output. As such, this is how a hello world looks like:

```javascript
def main:
  "Hello, world!"
```

Save this as `main.fmc` and using the terminal, go to the directory where you saved the file, then run it with `fmc main`. If you see `"Hello, world!"` in your terminal, then it worked. You can also enter just `fmc` for a list of options. We support multiple evaluators, including an interpreter, lazy/strict interaction-nets and even native JS closures, all with different characteristics. 
