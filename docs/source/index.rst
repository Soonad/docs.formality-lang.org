Welcome to Formality's documentation!
=====================================

Formality is an optimal functional programming language featuring theorem proving. It is similar to Agda and Idris in functionality, but takes a different approach to termination and induction: instead of a somewhat "hard-coded" structural recursion checker, it is based on Elementary Affine Logic, which gives it an elegant halting argument and complexity class, as well as great properties such as optimal substitutions, no garbage-collection, and an expressive type-theory with self-encoded inductive datatypes.

For an initial impression of how the language looks like, be encouraged o look some files such as [Data.Bool.fm](https://gitlab.com/moonad/Formality-Base/blob/master/Data.Bool.fm), [Data.List.fm](https://gitlab.com/moonad/Formality-Base/blob/master/Data.List.fm), [Control.Functor.fm](https://gitlab.com/moonad/Formality-Base/blob/master/Control.Functor.fm) or [Relation.Equality.fm](https://gitlab.com/moonad/Formality-Base/blob/master/Relation.Equality.fm) from [Formality-Base](https://gitlab.com/moonad/Formality-Base), our official standard libraries.

Table of contents
=====================================

.. toctree::
   :caption: Language
   :maxdepth: 2
   :numbered:
   
      language
   language/Motivation
   language/Installation
   language/Hello,-world!
   language/Basics
   language/Datatypes
   language/Boxes
   language/Theorem-Proving
   
.. toctree::
    :maxdepth: 2
    :numbered:
    :caption: Techniques

.. toctree::
    :maxdepth: 2
    :numbered:
    :caption: Runtime

    runtime/Formality-Net

.. toctree::
    :maxdepth: 2
    :caption: Tutorial

