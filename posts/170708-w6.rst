.. title: Status update week 6 GSoC
.. slug: gsoc-week6
.. date: 2017-07-08 21:19:00 UTC+02:00
.. tags: Python, SymPy
.. category: 
.. link: 
.. description: Sixth week of developing code-generation in SymPy for GSoC.
.. type: text

Finishing up tutorial material
------------------------------
We have been busy adding the final touches to the upcoming tutorial. (you can read
about the tutorial in ealier blog posts).

I thought I would share a technique for doing exercises in Jupyter notebooks.


The %exercise magic
-------------------
There are quite a few ways to provide exercises in jupyter notebooks
(eventually I think there will be a de facto standard, either as a
separate package or included with the notebook itself). But I
essentially wanted a special version of ``%load`` which would load a
file into a cell but replace some parts of the expressions with gaps
for the tutorial attendee to fill in.

Thanks to IPython being a popular project there was already `a
solution <https://stackoverflow.com/a/38103336/790973>`_
posted on StackOverflow which did nearly exactly this.

After some tweaking I came up with this

.. code:: python

    import IPython.core.magic as ipym
    
    @ipym.magics_class
    class ExerciseMagic(ipym.Magics):
    
        @ipym.line_magic
        def exercise(self, line, cell=None):
            token = '  # EXERCISE: '
            out_lst = []
            for ln in open(line).readlines():
                if token in ln:
                    pre, post = ln.split(token)
                    out_lst.append(pre.replace(post.rstrip('\n'), '???') + '\n')
                else:
                    out_lst.append(ln)
            out_str = '# %exercise {0}\n{1}'.format(line, ''.join(out_lst))
            self.shell.set_next_input(out_str, replace=True)
    
    def load_ipython_extension(ipython):
        ipython.register_magics(ExerciseMagic)
    
    def unload_ipython_extension(ipython):
        pass


The downside of using this magic is that reloading the cell requires
the user to uncomment the ``# %exercise ...`` line, delete the rest of
the cell and rerun it. This will only feel natural if you've already
worked with the ``%load`` magic before (which in my humble opinion
is suboptimal since you often want bi-directional editing whenever you
pull out that magic).
