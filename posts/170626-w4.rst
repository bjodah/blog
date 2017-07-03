.. title: Status update week 4 GSoC
.. slug: gsoc-week4
.. date: 2017-06-26 21:39:00 UTC+02:00
.. tags: Python, SymPy
.. category: 
.. link: 
.. description: Fourth week of developing code-generation in SymPy for GSoC.
.. type: text

Continued work on tutorial material
-----------------------------------
During the past week I got `binder <https://mybinder.org>`_ to work with
our tutorial material. Using ``environment.yml`` did require that we had
a conda package available. So I set up our instance of `Drone IO
<https://drone.io>`_ to push to the ``sympy`` conda channel. The new
base dockerimage used in the beta version of binder (v2) does not
contain gcc by default. This prompted me to add gcc to our
``environment.yml``, unfortunately this breaks the build process::

   Attempting to roll back.
   LinkError: post-link script failed for package defaults::gcc-4.8.5-7
   running your command again with `-v` will provide additional information
   location of failed script: /opt/conda/bin/.gcc-post-link.sh
   ==> script messages <==
   <None>

I've reached out on their gitter channel, we'll see if anyone knows
what's up. If we cannot work around this we will have two options:

1. Accept that the binder version cannot compile any generated code
2. Try to use binder with a Dockerimage based on the one used for CI
   tests (with the difference that it will include a prebuilt conda
   environment from ``environment.yml``)

I hope that the second approach should work unless we hit a image size
limitation (and perhaps we need a special base image, I haven't looked
into those details yet).

Jason has been producing some really high quality tutorial notebooks
and left lots of constructive feedback on my initial work. Based on
his feedback I've started reworking the code-generation examples not
to use classes as extensively. I've also added some introductory
notebooks: numerical integration of ODEs in general & intro to SymPy's
cryptically named ``lambdify`` (the latter notebook is still to be
merged into master).

Work on SymPy for version 1.1
-----------------------------
After last weeks mentor meeting we decided that we would try to
`revert <https://github.com/sympy/sympy/pull/12805>`_ a change to
``sympy.cse`` (a function which performs common subexpression
elimination). I did spend some time profiling the new implementation.
However, I was not able to find any obvious bottle-necks, and given
that it is not the main scope of my project I did not pursue this any
further at the moment.

I also `just started <https://github.com/sympy/sympy/pull/12808>`_
looking at introducing a Python code printer. There is a
``PythonPrinter`` in SymPy right now, although it assumes that
``SymPy`` is available. The plan right now is to rename to old
printer to ``SymPyCodePrinter`` and have the new printer primarily print
expressions, which is what the ``CodePrinter`` class does best, even
though it can be "coerced" into emitting statements.


Plans for the upcoming week
---------------------------
As the conference approaches the work intensifies both with respect to
finishing up the tutorial material and fixing blocking issues for a
SymPy 1.1 release. Working on the tutorial material helps finding
things to improve code-generation wise (just today I opened a `minor
issue <https://github.com/sympy/sympy/issues/12810>`_ just to realize
that my `"work-in-progress" pull-request
<https://github.com/sympy/sympy/pull/12693>`_ from an earlier week
(which I intend to get back working on next week too) actually fixes
said issue.
