.. title: Status update week 11 GSoC
.. slug: gsoc-week11
.. date: 2017-08-14 20:17:00 UTC+02:00
.. tags: Python, SymPy
.. category: 
.. link: 
.. description: Eleventh week of developing code-generation in SymPy for GSoC.
.. type: text

Checking in the new AST node types
----------------------------------
`#12693 <https://github.com/sympy/sympy/pull/12693>`_ got merged 🎉.
It took a few rewrites essentially but I fell that the design of the
new nodes will allow us to scale with reasonable maintance cost when
adding new language specific nodes. The base class for new AST nodes
(``Token``) to sublcass from allows one to implement nodes in an
expressive manner by setting ``__slots__``. The constructor of
``Token`` then sets the ``.args`` of ``Basic`` based on ``__slots__``
this has the benefit that you need not write setters and getters using
``@property`` decorators (which quickly becomes tiresome when you have
many classes). 

Checking in the refactored PythonPrinter and changes to lambdify
----------------------------------------------------------------
Finally the challenging work of refactoring ``lambdify`` got merged
into SymPy's master branch: `#13046
<https://github.com/sympy/sympy/pull/13046>`_. We eventually decided
to drop the contents of the old translations dictionaries but leave
them be (in an empty state) in case users were modifying those in
their code. Hopefully this approach doesn't break any code out
there. Given how popular ``lambdify`` is among SymPy's users, it is a
bit worrying that the test suite is not that extensive. I do remember
a google engineer mentioning that the follow the "Beyonce principle":
"I you liked it you should have put a test on it". Funny at is may be
I hope I don't need to defend these changes with that arguement.
