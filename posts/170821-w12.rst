.. title: Status update week 12 GSoC
.. slug: gsoc-week12
.. date: 2017-08-21 19:30:00 UTC+02:00
.. tags: Python, SymPy
.. category: 
.. link: 
.. description: Twelfth week of developing code-generation in SymPy for GSoC.
.. type: text

Working on new AST nodes
------------------------
`#13100 <https://github.com/sympy/sympy/pull/13100>`_ is shaping up to
be the largest PR of my GSoC project. The design of the new AST nodes
especially (``Token``) is really helpful. But there is still a design
issue: some nodes would naturally take different arguments depending
on what language is being targeted. So I came to the conclusion that I
needed some way of representing attributes. The solution I came up
with would be to have a slightly more capable ``Node`` class
(subclassing ``Token``) which would in turn be subclassed from for
nodes that need attributes.

I also enhanced the printing of both of these classes and introduced a
``String`` class, which in contrast to ``Symbol`` does not accept
assumptions in its constructor, and does not have implied printing
rules of sub- & superscript etc.

Adding ``.codegen.algorithms``
------------------------------
A new submodule ``.codegen.algorithms`` was added, containing a AST
generating function for Newton's method. This makes a nice design
target for both the printers and AST nodes: being able to express the
same AST in differnt languages is definitely an indication that we
have a versatile printing system.

Compiling code on the fly
-------------------------
Introducing the ``.codegen.algorithms`` module also made the need to
test generated code during CI runs clear. Jason Moore has previously
mentioned that he thinks one of my python packages (`pycompilation
<https://github.com/bjodah/pycompilation>`_) would fit nicely into
SymPy. I've been a bit relucatant to port it over since I have felt
that it has not seen enough testing (and only under Linux). But now
there was a need and we could start by making it an internal package
only used by our own tests. That way it will get to mature without
having to worry about deprecation cycles. And once more platforms are
added to SymPy's CI configuration it would also see testing on other
platforms (using AppVeyor for SymPy has been discussed for a long
while now).
