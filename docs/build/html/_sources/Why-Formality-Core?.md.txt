# Why Formality-Core?

Formality-Core (FMC) is a minimal, efficient functional programming language designed to be a "low-level" compile-target for more feature-rich functional languages. Differently from most functional languages, FMC has amazing computational characteristics such as not requiring a garbage-collector, beta-reduction being a constant-time operation, being able to perform dynamic fusion, being strongly normalizing and massively parallelizable. In other words, FMC is really, really fast, and aims to be the ultimate assembly of resource-aware functional languages.

In order for all that to be possible, FMC makes some fundamental changes to the memory model behind the λ-calculus, allowing it to be compatible with an optimal functional runtime as [Lamping's Abstract Algorithm](https://medium.com/@maiavictor/solving-the-mystery-behind-abstract-algorithms-magical-optimizations-144225164b07), which is based on a beautiful model of computation, [Interaction-Nets](https://pdfs.semanticscholar.org/1731/a6e49c6c2afda3e72256ba0afb34957377d3.pdf). The most important change is the fact its lambdas are affine: a lambda-bound variables can't be used more than once. At first, this sounds like an extreme limitation, but this is mitigated by FMC's boxed duplication system. It allows you to perform deep copy of arbitrary values, as long as you respect certain structural restrictions. 

Formality-Core is very similar to Rust. The main difference is that Rust mitigates the limitations of affine lambdas with a complex ownership system which includes borrows, lifetimes, mutable references. In FMC, you're instead encouraged to simply `.clone()` structures whenever you need to. Due to the underlying interaction-net runtime, those copies are performed lazily, granularly and in parallel. They're, thus, much less expensive than a Rust-like `.clone()` (and often free).

Here is an brief illustration of the process of compiling a Formality-Core term to our interaction-net runtime, reducing it in parallel, and reading the result:

![FM Interaction Net](https://github.com/moonad/formality/raw/master/docs/images/inet-simulation.gif)
