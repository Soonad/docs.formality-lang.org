Welcome to Formality's documentation!
=====================================

Formality is an optimal functional programming language featuring theorem proving. It is similar to Agda and Idris in functionality, but takes a different approach to termination and induction: instead of native datatypes with structural recursion, it uses λ-encodings, self-types and relies on a different underlying logic, "elementary affine", which gives it an elegant halting argument. This gives it some unique properties such as optimal substitutions, practical efficiency, and an elegant underlying theory.

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
   language/Basic-Features
   language/Datatypes
   language/Box-System
   language/Recursion
   
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

