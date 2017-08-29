.. title: Status update week 13 GSoC
.. slug: gsoc-week13
.. date: 2017-08-28 21:15:00 UTC+02:00
.. tags: Python, SymPy
.. category: 
.. link: 
.. description: 13th week of developing code-generation in SymPy for GSoC.
.. type: text

This was the last week of work on GSoC. I have been hard at work
improving documentation and examples for the code.

Adding much needed documentaiton
--------------------------------
I've spent the weekend adding examples and writing up documentation
for my big PR `#13100 <https://github.com/sympy/sympy/pull/13100>`_
which is not yet merged. I am quite excited how this PR turned out and
I am happy with the design of the underlying AST nodes.

Rewriting
---------
A new submodule ``.codegen.rewriting`` was added (in `#13194
<https://github.com/sympy/sympy/pull/13194>`_), this allows a user to
rewrite expressions using special math functions. The provided rules
are those to rewrite to C99's special math functions (``expm1``,
``log1p`` etc.). I think it will be a useful addition (I have myself
had the need for exactly this in my own research). The design is quite
simple thanks to the excellt ``replace`` function in SymPy. There are
still some corner cases (I have an "xfailed" test checked in for
example).
