.. title: Status update week 8 GSoC
.. slug: gsoc-week8
.. date: 2017-07-24 18:42:00 UTC+02:00
.. tags: Python, SymPy
.. category: 
.. link: 
.. description: Eighth week of developing code-generation in SymPy for GSoC.
.. type: text

Work on the new PythonCodePrinter
---------------------------------
I finished up the new ``PythonCodePrinter`` in `#12808
<https://github.com/sympy/sympy/pull/12808>`_. And I have started work
locally on having the NumPy- and Mpmath-printers subclass this new
CodePrinter. Both of these printers were previously subclassing
``LambdaPrinter``, so the change is a bit intrusive with lots of
failing tests in the SymPy test suite. But this is a prerequisite for
changing ``sympy.utilities.lambdify`` to accept custom ``CodePrinter``
subclasses as user input.
