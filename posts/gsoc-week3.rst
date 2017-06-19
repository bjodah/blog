.. title: Status update week 3 GSoC
.. slug: gsoc-week3
.. date: 2017-06-19 23:15:00 UTC+02:00
.. tags: Python, SymPy
.. category: 
.. link: 
.. description: Third week of developing code-generation in SymPy for GSoC.
.. type: text

Fast callbacks from SymPy using SymEngine
-----------------------------------------
My main focus the past week has been to get ``Lambdify`` in SymEngine
to work with multiple output parameters. Last year Isuru Fernando lead
the development_ to support jit-compiled callbacks using LLVM in SymEngine.
I started work on leveraging this in the Python wrappers of SymEngine
but my work stalled due to time constraints.

But since it is very much related to code generation in SymPy I did
put it into my time-line (later in the summer) in my GSoC
application. But with the upcoming SciPy conference, and the fact that
it would make a nice addition to our tutorial, I have put in work_ to
get this done earlier than first planned.

Another thing on my to-do-list from last week was to get ``numba`` working
with ``lambdify``. But for this to work we need to wait for a new upstream
release of ``numba`` (which they are hoping to release before the SciPy
conference_).

.. _development: https://github.com/symengine/symengine/pull/1094
.. _work: https://github.com/symengine/symengine.py/pull/112
.. _conference: https://scipy2017.scipy.org/ehome/220975/493418/

Status of codegen-tutorial material
-----------------------------------
I have not added any new tutorial material this week, but have been
working on making all notebooks work under all targeted operating
systems. However, every change to the notebooks have to be checked
on all operating systems using both Python 2 and Python 3. This
becomes tedious very quickly so I decided to enable continuous
integration on our repository_. I followed conda-forges approach: Travis CI
for OS X, CircleCI for Linux and AppVeyor for Windows (and a private
CI server for another Linux setup). And last night
I *finally* got green light on all 4 of our CI services.

.. _repository: https://github.com/sympy/scipy-2017-codegen-tutorial


Plans for the upcoming week
---------------------------
We have had a performance regression_ in ``sympy.cse`` which has bit me
multiple times this week. I managed to craft_ a small test case
indicating that the algorithmic complexity of the new function is
considerably worse than before (effectively making it useless for many
applications). In my weekly mentor-meeting (with Aaron) we discussed
possibly reverting that change_. I will first try to see if I can
identify easy-to-fix bottlenecks by profiling the code. But the
risk is that it is too much work to be done before the upcoming
new release of SymPy, and then we will simply revert for now (choosing
speed over extensiveness of the sub-expression elimination).

.. _regression: https://github.com/sympy/sympy/issues/12411
.. _craft: https://github.com/sympy/sympy_benchmarks/pull/38
.. _change: https://github.com/sympy/sympy/pull/11232

I still need to test the notebooks using not only ``msvc`` under Windows
(which is currently used in the AppVeyor tests), but also ``mingw``. I did
manage to get it working locally but there is still some effort left
in order to make this work on AppVeyor. It's extra tricky since there
is a bug_ in ``distutils`` in Python 3 which causes the detection of mingw
to fail. So we need to either:

- Patch ``cygwincompiler.py`` in ``distutils`` (which I believe we can do
  if we create a conda package for our tutorial material).
- ...or use something else than ``pyximport`` (I'm hesitant to do this
  before the conference).
- ...or provide a gcc executable (not a ``.bat`` file) that simply
  spawns ``gcc.bat`` (but that executable would need to be compiled
  during build of our conda package).

.. _bug: https://bugs.python.org/issue21821

Based on my work on making the CI services work, we will need to
provide test scripts for the participants to run. We need to provide
the organizers with these scripts by June 27th so this needs to be
decided upon during next week. I am leaning towards providing an
``environment.yml`` file together with a simple instruction of
activating said environment, e.g.::

  $ conda env create -f environment.yml
  $ source activate codegen17
  $ python -c "import scipy2017codegen as cg; cg.test()"

This could even be tested on our CI services.

I also intend to add a (perhaps final) tutorial notebook for chemical
kinetics where we also consider diffusion. We will solve the PDE using
the method of lines. The addition of a spatial dimension in this way
is simple in principle, things do tend to become tricky when handling
boundary conditions though. I will try to use the simplest possible
treatment in order to avoid taking focus from what we are teaching
(code-generation).

It is also my hope that this combined diffusion-reaction model is a
good candidate for ``ufuncify`` from
``sympy.utilities.autowrap``.
