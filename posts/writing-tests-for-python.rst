.. title: Writing tests for Python
.. slug: writing-tests-for-python
.. date: 2017-01-18 00:13:33 UTC+01:00
.. tags: python
.. category: 
.. link: 
.. description: 
.. type: text

A coworker asked me about "tests" from a programming practice perspective, "what's the big deal?".
For me, writing tests is a very small price to pay to comfortably daring to change code without
the fear of breaking it. Every bug I fix, even during the design phase, gives rise to at least one test.

No library or script is too small to have tests, here is a silly example (let's call it ``rootof.py``):

.. code-block:: python

   #!/usr/bin/env python

   import argh

   def root_of(number=1.0):
       return number**0.5

   def test_root_of():
       assert root_of(4) == 2
       assert abs(root_of(2) - 1.4142) < 1e-4

   if __name__ == '__main__':
       argh.dispatch_command(root_of)

The example is so short it's silly, but the point is that each time you edit ``rootof.py`` you
should run

.. code:: bash

   $ python -m pytest rootof.py

which should exit without errors. That's pretty much the gist of it. Happy testing!

P.S. if you don't have ``argh`` just install it by typing:

.. code:: bash

   $ python -m pip install --user argh

P.P.S if you don't have ``pip``: see `this <https://pip.pypa.io/en/stable/installing/>`_.
