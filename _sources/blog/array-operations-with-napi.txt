Array Operations with Napi
==========================

.. post:: Nov 11, 2013
   :tags: IPython, NumPy, Python, napi
   :category: napi

If there is one thing that I don't like about doing numerical analysis using
Python_ and NumPy_ is that expressions of chained comparisons and logical
operations containing arrays do not work:

.. ipython:: python
   :suppress:

   import napi
   %napi off

.. ipython:: python

   from numpy import *
   a = arange(20)
   6 <= a < 16

which works just fine for numbers:

.. ipython:: python

   6 <= a[10] < 16

And, then there is this:

.. ipython:: python

   6 <= a and a < 16


But, not anymore! Not at least for me in interactive IPython sessions :) I have
released a package called napi_ that comes with an IPython_ magic:

.. ipython::

   In [0]: import napi

   In [0]: %napi

This magic allows the following to just work:

.. ipython:: python

   6 <= a < 16 and a != 10

The equivalent without using napi_ is:

.. ipython:: python

   logical_and(logical_and(6 <= a, a < 16), a != 10)

So, interactive sessions with arrays are fun again!