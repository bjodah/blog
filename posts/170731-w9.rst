.. title: Status update week 9 GSoC
.. slug: gsoc-week9
.. date: 2017-07-24 18:42:00 UTC+02:00
.. tags: Python, SymPy
.. category: 
.. link: 
.. description: Ninth week of developing code-generation in SymPy for GSoC.
.. type: text

This week has been mostly spent on two refactoring lambdify and
enhancing the representation of types in our AST nodes under ``.codegen.ast``

Refactoring lambdify
--------------------
The tricky business of chaning ``lambdify`` to use our new CodePrinter
(or more importantly: letting the user pass their own subclasses
continues).

There are currently dictionaries in ``sympy.utilities.lambdify`` which
contains translations of SymPy entities to functions in corresponding
targeted python module (e.g. NumPy functions if the numpy module is
targeted). This week I've `worked
<https://github.com/sympy/sympy/pull/13046>`_
toward removing dependence on these,
and instead rely on the user providing print methods instead.

Work on types in ``.codegen.ast``
---------------------------------
In my `WIP PR for AST nodes
<https://github.com/sympy/sympy/pull/12693>`_ I have made more type
related changes. I've also changed the Fortran printer to use the new
type information. I am not 100% satisfied with the way the floating
point types are being represented at the moment (I'm using floating
point numbers to store the max, min and epsilon values for example).
I will probably rewrite those classes to use the underlying number of
bits for mantissa and exponent as the canonical data for the class
instances. 

I finished up the new ``PythonCodePrinter`` in `#12808
<https://github.com/sympy/sympy/pull/12808>`_. And I have started work
locally on having the NumPy- and Mpmath-printers subclass this new
CodePrinter. Both of these printers were previously subclassing
``LambdaPrinter``, so the change is a bit intrusive with lots of
failing tests in the SymPy test suite. But this is a prerequisite for
changing ``sympy.utilities.lambdify`` to accept custom ``CodePrinter``
subclasses as user input.
