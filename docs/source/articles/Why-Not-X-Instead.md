# Why not X instead?

There are amazing projects with similar goals and vision. In this article, I'll point the reasons we've decided to build Formality rather than using one of them. 

## Why not Plutus, Michelson or Vyper instead?

Okay, so, Plutus in particular is just a "normal" functional language, very similar to Haskell. It has algebraic datatypes and polymorphic functions, which is actually really cool if compared to Solidity, but that's about it. It has none of the tools required to qualify as a "proof language".

**No dependent types.**

You can't have values at the type level at all, so, while you can have polymorphic types like "a List of numbers", you can't have really precise types like "a List with 8 even numbers".

**No induction or inductive datatypes.**

As any mathematician may tell you, you can't go much far in mathematics without induction. Even proving something as simple as `a + b = b + a` requires it. Imagine properties about smart-contracts.

**No equality types.**

Even if you could live without induction, you can't even *express* something like `a + b = b + a` on Plutus. There is no concept of equality on its type system.

**No sigmas ("such that").**

Plutus has a polymorphic "forall", but no "there exists" or "such that". So, you can't say something like "give me a function that returns a number **such that** it is a multiple of 7...". 

**And so on.**

In short, Plutus is just a normal, non-proof language, it simply doesn't have a type language capable of expressing specifications and theorems in any way. The only sense in which I could see someone selling Plutus as more proof-friendly than Solidity would be that it is can be imported in a "real" proof language like Agda. But the same is true for the EVM, and even if it wasn't, it is awfully inconvenient. Formality can perfectly prove theorems about its own programs.

TODO: elaborate about Michelson and Solidity (generally, the same points apply).

From [this Reddit thread](https://www.reddit.com/r/ethereum/comments/d45vpq/im_hyper_bullish_on_ethereum/f09cj2f/?context=2).

## Why not Agda, Coq or Idris instead?

TODO: write this section including points such as:

**Runtime efficiency and portabiliy.**

Write about the difficulty of compiling non-linear languages to inets efficienly. Full λ-calculus closures require high-order, garbage-collected runtimes that are generally too expensive for blockchains and similar. Also write about Formality's 450 LOC runtime and how that matters for portability.

**Implementation complexity.**

Write about our experience trying to hack and extend Agda/Idris with features we needed. 

**Compiler portability.**

Write about the difficulty of running Agda/Idris inside a browser or mobile efficiently, for applications such as Provit, or a web-based smart-contract verifying site, etc.

**Logical consistency.**

We can't fix potential bugs in other languages and we absolutely require consistency for our applications, so a small stable core is important. Agda doesn't have one, Idris has inconsistency bugs. I think Coq would be fine though.

(Note about how much we love Agda and Idris and are actually constantly trying to use those instead, and re-evaluating whether it'd be possible.)
