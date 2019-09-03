# Motivation

Formality exists to fill a hole in the current market: there aren't many
languages featuring *theorem proving* that are also built to user-friendly
(rather than PhD-targeting tools), and to be very efficient at runtime. In order
to achieve those goals, years of research and a lot of hard-thinking were put on
the smallest decisions, and we're still making constant progress. Our current
design already brings 3 major innovations:

### Optimal substitutions

Formality's substitution algorithm is asymptotically faster than Haskell,
Scheme, JavaScript and other languages featuring closures. This makes it
extremely fast at evaluating programs that rely on many high-order functions.
For example, do you know GHC's marvelous foldr/build and stream fusion
optimizations? Formality is capable of performing those dynamically, [at
runtime](https://medium.com/@maiavictor/solving-the-mystery-behind-abstract-algorithms-magical-optimizations-144225164b07).
Optimal reductions also allow us to explore new ways to develop algorithms, such
as this "impossibly efficient" [exp-mod
implementation](https://gist.github.com/MaiaVictor/e556062185c5863d814980123e03630f)
based on billions of tuple rotations. Who knows if this may lead to new
breakthroughs in complexity theory?

### A "perfect" execution model

While the actual efficiency of a programming language depends on the
implementation, some languages are inherently slow, by design. JavaScript, for
example, is slower than C: all things equal, its mandatory garbage collector
will be an unavoidable extra cost. Formality's execution model is, by design, as
fast as one could be. For example, due to linearity (actually, affinity), it
doesn't require a garbage collector. Due to the confluence of its
interaction-combinator runtime, it can be evaluated in a massively parallel
fashion. Due to Elementary Affine Logic (and unlike
[BOHM](https://github.com/cls/bohm), it doesn't require bookkeeping machinery
for its optimal substitutions. It can be evaluated lazily, just like Haskell.
Unlike other functional languages, beta-reduction is a O(1), cost-measurable
operation, making it extremelly suitable for blockchains. And so on. While
Formality is, right now, not as fast as mature languages (after all, those have
years of optimizations behind), there is simply no "slow design decision"
stopping it from getting faster than any of them.

### A powerful type-theory

We conjecture that Formality's unique approach to termination allows its type
system to have a bunch of powerful features that would otherwise be impossible
without making the proof language inconsistent. As an example, it features `Type
: Type`, which is powerful and very convenient. It also features mutual
type-level recursion, allowing us to exploit self-types to elegantly represent
datatypes as their own induction schemes, without needing a complex native
datatype system. We're working hard towards proofs of those claims and hope they
can be published soon.

![](https://github.com/moonad/formality/raw/master/docs/images/inet-simulation.gif)
*Interaction Net (inet) simulation*
