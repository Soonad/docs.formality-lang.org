Welcome to Formality's documentation!
=====================================

Formality is an optimal functional programming language featuring theorem proving. It is similar to Agda and Idris in functionality, but takes a different approach to termination and induction: instead of a somewhat "hard-coded" structural recursion checker, it is based on Elementary Affine Logic, which gives it an elegant halting argument and complexity class, as well as great properties such as optimal substitutions, no garbage-collection, and an expressive type-theory with self-encoded inductive datatypes. To get a feel for how the language looks, please take a look at files such as `Data.Bool.fm <https://gitlab.com/moonad/Formality-Base/blob/master/Data.Bool.fm>`_, `Data.List.fm <https://gitlab.com/moonad/Formality-Base/blob/master/Data.List.fm>`_, `Control.Functor.fm <https://gitlab.com/moonad/Formality-Base/blob/master/Control.Functor.fm>`_ or `Relation.Equality.fm <https://gitlab.com/moonad/Formality-Base/blob/master/Relation.Equality.fm>`_ from `Formality-Base <https://gitlab.com/moonad/Formality-Base>`_, our official standard libraries.

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
   language/4.Basics
   language/5.Datatypes
   language/6.7.Boxes
   language/8.Theorem-Proving
   
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

