## Formality Net

Formality terms are compiled to a memory-efficient interaction net system. Interaction nets are just graphs where nodes have labelled ports, one being the main one, plus a list of "rewrite rules" that are activated whenever two nodes are connected by their main ports. Our system includes 6 types of node:

In our implementation, `CON`, `OP1`, `OP2` and `ITE` require 128 bits, while `NUM` and `ERA` are unboxed, i.e., they're stored inside other nodes and use no extra space. Notice our system doesn't include a costly book-keeping machinery to keep track of Bruijn indices; instead, it can be seen as a practical extension to [symmetric interaction-combinators](https://pdfs.semanticscholar.org/1731/a6e49c6c2afda3e72256ba0afb34957377d3.pdf), in order to add numeric primitives.

### Node types
![node_types](https://github.com/moonad/Formality/blob/master/docs/images/fm-net-node-types.png)

### Rewrite rules
![rewrite_rules](https://github.com/moonad/Formality/blob/master/docs/images/fm-net-rewrite-rules.png)

### Compilation
![compilation](https://github.com/moonad/Formality/blob/master/docs/images/fm-net-compilation.png)
