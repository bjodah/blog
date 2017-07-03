.. title: Status update week 5 GSoC
.. slug: gsoc-week5
.. date: 2017-07-03 20:37:00 UTC+02:00
.. tags: Python, SymPy
.. category: 
.. link: 
.. description: Fith week of developing code-generation in SymPy for GSoC.
.. type: text

Continued work on tutorial material
-----------------------------------
Cross-platform work is always a bit tricky. Python does in general
a great job of allowing programs to access the filesystem, network etc
in an OS agnostic manner, but sometimes there is no way around having
to write special code for each platform. In Python that often means
that we will do runtime inspection:: 

  >>> sys.platform
  'linux'  # or e.g. darwin 
  >>> os.name
  'posix'  # or e.g. nt

To add another dimension of complexity we sometimes deal with 32-bit
and sometimes 64-bit systems (most headaches are caused by pointer
sizes, size of ``int`` and ``long``), in python we can get the size of
``int`` of the platform quite easily::

  >>> sys.maxsize - 2**63 + 1
  0

when you, in addition to this, are doing code-generation, you are most
likely relying on a compiler. That is our third dimension of
complexity, on linux that (usually) translates to gcc & llvm/clang
(but Intel's icc/ifort & portland group compilers and not uncommon).
This third "dimension" has been the number one struggle the past
week (and in particular the `combination
<https://github.com/sympy/scipy-2017-codegen-tutorial/issues/13>`_
mingw/`windows
<https://github.com/sympy/scipy-2017-codegen-tutorial/pull/14>`_). At
this point I feel that it is not worthwhile to continue pursuing ``mingw``
support on Windows.

It is quite likely that a few of our attendees will have some
technical difficulties with system installed compilers. Therefore, I'm
particularly happy that we now have got our `binder
<https://mybinder.org>`_ environment to work flawlessly. It took some
work to setup a `Dockerfile
<https://github.com/sympy/scipy-2017-codegen-tutorial/blob/master/Dockerfile>`_,
but now it feels reassuring to have this as a fall-back.


Work on SymPy for version 1.1
-----------------------------
The `work <https://github.com/sympy/sympy/pull/12808>`_ on the
``PythonCodePrinter`` has progressed since last week and should
hopefully soon be ready for merging. The purpose of this class is to
be stepping stone moving away from having e.g. ``lambdify`` relying on
the StrPrinter, and instead use the ``CodePrinter`` class in
``.printing.codeprinter``. Changing the super class showed to be
trickier than first anticipiated (I thought it would be an afternoons
work at most). I originally planned to rename the original
``PythonPrinter`` to something more appropriate (``SymPyCodePrinter``)
together with its convenience functions. However, since this would
incur a deprecation for only name change, it was decided that we will
have to live with the old naming (to avoid raising tons of deprecation
warnings in downstream projects).

We had identified two extensions we wanted to introduce to the
codegeneration/autowrap factilities before the v1.1 release:

- Kenneth Lyons (@ixjlyons) `spear-headed
  <https://github.com/sympy/sympy/pull/12833>`_ the effort to allow
  custom CodeGen subclasses to be passed to ``codegen``
- ...and Jason the `changes
  <https://github.com/sympy/sympy/pull/12843>`_ to the
  ``CythonWrapper`` to allow compiler arguments to be passed to
  autowrap. 


Plans for the upcoming week
---------------------------
With a release candidate of SymPy v1.1 out the door (an impressive
effort led by @asmeurer) and most cross-platform issues out of the
way, it is about time for the final iterations of the  tutorial
material (and hopefully iron out any related issues in the rc).

At the end of the week it's time for me to fly to the US to attend
SciPy 2017. I have high expectations and believe that it will be a
great catalyst for the rest of the summers work on code-generation.
