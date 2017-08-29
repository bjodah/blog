.. title: Status update week 10 GSoC
.. slug: gsoc-week10
.. date: 2017-08-07 21:03:00 UTC+02:00
.. tags: Python, SymPy
.. category: 
.. link: 
.. description: Tenth week of developing code-generation in SymPy for GSoC.
.. type: text

Refactoring lambdify
--------------------
In my work to refactor ``lambdify`` I had come up with a solution
where I would dynamically subclass the CodePrinters in ``lambdify`` to
add translations from the old translation dictionaries. I was not
happy with the solution and I don't think Aaron was either, we decided
to keep the old import mechanism of lambdify which populated the
namespace (instead of trying to generate code for the used imports
which I had been trying).

So the work on refactoring ``lambdify`` has continued in `#13046
<https://github.com/sympy/sympy/pull/13046>`_. And with `this
<https://github.com/sympy/sympy/commit/265314fa63f5a662a7a187913d51d55a852b503c>`
commit I hope we are close to getting the new version of ``lambdify``
out the door.

New FloatType representation
----------------------------
With some underlying assumptions about floating point representation
(two's complement etc.) I have now a `new representation
<https://github.com/sympy/sympy/commit/18ea9a583509f28bc88102e73ebff8e0443d7988>`_
of ``FloatType``. I'm much happier with this representation and I
think with it `#12693 <https://github.com/sympy/sympy/pull/12693>`_ is
much closer to getting merged.
