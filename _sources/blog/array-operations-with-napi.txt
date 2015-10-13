Array Operations with Napi
==========================

.. post:: Nov 11, 2013
   :tags: IPython, NumPy, Python, napi
   :category: napi

If there is one thing that I don't like about doing numerical analysis using
Python_ and NumPy_ is that expressions of chained comparisons and logical
operations containing arrays do not work:


.. code-block:: ipython

   In [1]: from numpy import *

   In [2]: a = arange(20)

   In [3]: 6 <= a < 16
   ---------------------------------------------------------------------------
   ValueError                                Traceback (most recent call last)
   <ipython-input-3-adbdf9a9401c> in <module>()
   ----> 1 6 <= a < 16

   ValueError: The truth value of an array with more than one element is ambiguous. Use a.any() or a.all()


which works just fine for numbers:

.. code-block:: ipython

   In [4]: 6 <= a[10] < 16
   Out[4]: True

And, then there is this:

.. code-block:: ipython

   In [5]: 6 <= a and a < 16
   ---------------------------------------------------------------------------
   ValueError                                Traceback (most recent call last)
   <ipython-input-5-1880bf3f7d9a> in <module>()
   ----> 1 6 <= a and a < 16

   ValueError: The truth value of an array with more than one element is ambiguous. Use a.any() or a.all()

But, not anymore! Not at least for me in interactive IPython sessions :) I have
released a package called napi_ that comes with an IPython_ magic:

.. code-block:: ipython

   In [6]: import napi

   In [7]: %napi
   napi transformer is ON

This magic allows the following to just work:

.. code-block:: ipython

   In [8]: 6 <= a < 16 and a != 10
   Out[8]:
   array([False, False, False, False, False, False,  True,  True,  True,
           True, False,  True,  True,  True,  True,  True, False, False,
          False, False], dtype=bool)

The equivalent without using napi_ is:

.. code-block:: ipython

   In [9]: logical_and(logical_and(6 <= a, a < 16), a != 10)
   Out[9]:
   array([False, False, False, False, False, False,  True,  True,  True,
           True, False,  True,  True,  True,  True,  True, False, False,
          False, False], dtype=bool)

So, interactive sessions with arrays are fun again!