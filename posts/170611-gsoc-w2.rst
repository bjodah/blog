.. title: Status update week 2 GSoC
.. slug: gsoc-week2
.. date: 2017-06-11 23:37:00 UTC+02:00
.. tags: Python, SymPy
.. category: 
.. link: 
.. description: Second week of developing code-generation in SymPy for GSoC.
.. type: text

I have spent the second week of Google Summer of Code on essentially two things:

1. Continued work on type awareness in the code printers (``CCodePrinter`` and
   ``FCodePrinter``). The work is ongoing in gh-12693_.

2. Writing tutorial code_ on code-generation in the form of jupyter notebooks for the
   upcoming SciPy 2017 conference_.

.. _gh-12693: https://github.com/sympy/sympy/pull/12693
.. _conference: https://scipy2017.scipy.org/ehome/220975/493418/
.. _code: https://github.com/sympy/scipy-2017-codegen-tutorial

Precision aware code printers
-----------------------------
After my weekly mentor meeting, we decided to take another approach to
how we are going to represent ``Variable`` instances in the
``.codegen.ast`` module. Previously I had proposed to use quite a
number of arguments (stored in ``.args`` since it inherits
``Basic``). Aaron suggested we might want to represent that underlying
information differently. After some discussion we came to the
conclusion that we could introduce a ``Attribute`` class (inhereting
from ``Symbol``) to describe things such as value const-ness, and
pointer const-ness (those two are available as ``value_const`` and
``pointer_const``). Attributes will be stored in a ``FiniteSet``
(essentially SymPy version of ``set``) and the instances we provide as
"pre-made" in the ``.codegen.ast`` module will be supported by the
printers by default. Here is some example code showing what the
current proposed API looks like (for C99)::

  >>> u = symbols('u', real=True)
  >>> ptr = Pointer.deduced(u, {pointer_const, restrict})
  >>> ccode(Declaration(ptr))
  'double * const restrict u;'

and for Fortran::

  >>> vx = Variable(Symbol('x'), {value_const}, float32)
  >>> fcode(Declaration(vx, 42))
  'real(4), parameter :: x = 42'

The C code printer can now also print code using different math functions depending
on the targeted precision (functions guaranteed to be present in the C99 stanard)::

  >>> ccode(x**3.7, standard='c99', precision=float32)
  'powf(x, 3.7F)'
  >>> ccode(exp(x/2), standard='c99', precision=float80)
  'expl((1.0L/2.0L)*x)'

Tutorial material for code generation
-------------------------------------
Aaron, Jason and I have been discussing what examples to use for the
tutorial on code generation with SymPy. Right now we are aiming to use
quite a few examples from chemistry actually, and more specifically
`chemical kinetics
<https://en.wikipedia.org/wiki/Chemical_kinetics>`_. This is the
precise application which got me started using SymPy for
code-generation, so it lies close to my heart (I do extensive modeling
of chemical kinetics in my PhD studies).

Working on the tutorial material has already been really helpful for
getting insight in development needs for the exisiting classes and
functions used for code-generation. I was hoping to use ``autowrap``
from the ``.utilities`` module. Unfortunately I found that it was not
flexible enough to be useful for integration of systems of ODEs (where
we need to evaluate a vector-valued function taking a vector as
input). I did attempt to subclass the ``CodeWrapper`` class to allow
me to do this. But personally I found those classes to be quite hard to
extend (much unlike the printers which I've often found to be
intuitive).

My current plan for the chemical kinetics case is to first solve it
using ``sympy.lambdify``.  That allows for quick prototyping, and
unless one has very high demands with respect to performance, it is
usually good enough.

The next step is to generate a native callback (it was here I was
hoping to use ``autowrap`` with the Cython backend). The current
approach is to write the expressions as Cython code using a template.
Cython conveniently follows Python syntax, and hence the string
printer can be used for the code generation. Doing this speeds up the
integration considerably. At this point the bottleneck is going back and
forth through the Python layer.

So in order to speed up the integration further, we need to bypass
Python during integration (and let the solver call the user
provided callbacks without going through the interpreter). I did
this by providing a C-code template which relies on ``CVode`` from the
`Sundials <https://computation.llnl.gov/projects/sundials>`_ suite of
non-linear solvers. It is a well established solver and is already
available for Linux, Mac & Windows under conda from the
``conda-forge`` channel. I then provide a thin Cython wrapper, calling
into the C-function, which:

1. sets up the CVode solver
2. runs the integration
3. records some statistics

Using native code does come at a cost. One of the strengths of Python
is that it is cross-platform. It (usually) does not matter if your
Python application runs on Linux, OSX or Windows (or any other
supported operating system). However, since we are doing
code-generation, we are relying on compilers provided by the
respective platform. Since we want to support both Python 2 & Python 3
on said three platforms, there are quite a few combinations to cover.
That meant quite a few surprises (I now know for example that MS
Visual C++ 2008 does not support C99), but thanks to the kind help_ of
`Isuru Fernando <https://github.com/isuruf>`_ I think I will manage to
have all platform/version combinations working during next week.

.. _help: https://github.com/sympy/scipy-2017-codegen-tutorial/issues/2#issuecomment-307538308

Also planned for next week:

- Use ``numba`` together with ``lambdify``
- Use ``Lambdify`` from `SymEngine <https://github.com/symengine>`_
  (preferably with the LLVM backend).
- Make the notebooks more tutorial like (right now they are more of a
  show-case).

and of course: continued work on the code printers. That's all for now
feel free to get in touch with any feedback or questions.
