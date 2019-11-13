Welcome to Formality's documentation!
=====================================

Formality is an optimal functional programming language featuring theorem
proving. It is similar to Agda and Idris in functionality, but takes a different
approach to termination and induction: instead of a somewhat "hard-coded"
structural recursion checker, it is based on Elementary Affine Logic, which
gives it an elegant halting argument and complexity class, as well as great
properties such as optimal substitutions, no garbage-collection, and an
expressive type-theory with self-encoded inductive datatypes. To get a feel for
how the language looks, please take a look at files such as `Data.Bool.fm
<https://github.com/moonad/Formality-Base/blob/master/Data.Bool.fm>`_,
`Data.List.fm
<https://github.com/moonad/Formality-Base/blob/master/Data.List.fm>`_,
`Control.Functor.fm
<https://github.com/moonad/Formality-Base/blob/master/Control.Functor.fm>`_ or
`Relation.Equality.fm
<https://github.com/moonad/Formality-Base/blob/master/Relation.Equality.fm>`_
from `Formality-Base <https://github.com/moonad/Formality-Base>`_, our official
standard libraries.

NOTE: this documentation is slightly outdated after the syntax changes of
November 13, 2019 (version 0.1.170). We're going to move docs to a brand new
site, though, so this will be outdated for the next few days. On the meantime,
check the base libraries (http://github.com/moonad/formality-base) to see the
current syntax.

Table of contents
=====================================

.. toctree::
   :caption: Language
   :maxdepth: 2
   :numbered:
   
      language
   language/1.Motivation
   language/2.Installation
   language/3.Hello,-world!
   language/4.Core-Features
   language/5.Datatypes
   
.. toctree::
    :maxdepth: 2
    :numbered:
    :caption: Theory

    theory/Formality-Net
    theory/Self-Types

.. toctree::
    :maxdepth: 2
    :caption: Tutorials
    :numbered:

    tutorials/1.Dependent-Types
    tutorials/2.Recursion
    tutorials/3.Linearity
    tutorials/4.Boxes
    tutorials/5.Theorem-Proving

.. toctree::
    :maxdepth: 2
    :caption: Articles
    :numbered:

    articles/What-Are-Proofs
    articles/Why-Not-X-Instead
