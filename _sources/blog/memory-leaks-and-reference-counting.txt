Memory Leaks and Reference Counting
===================================

.. post:: Oct 10, 2013
   :tags: NumPy, ProDy, Python, Evol, C
   :category: ProDy

A while ago, I wrote some C code using Python and NumPy C APIs to parse
:wiki:`multiple sequence alignment` (MSA) files efficiently (available via
ProDy_ function :func:`~prody.sequence.msafile.parseMSA`). It went all well for
months (no bug reports from users) until I wanted to see how fast it really is.
So, I created a 1 GB MSA file by concatenating GPCR family (:pfam:`7tm_1`)
alignment multiple times and while benchmarking with it, I caught two nasty
bugs causing :wiki:`segmentation fault`.

Memory leak
===========

The first one was simple to figure out: a huge memory leak that showed itself
when simply analyzing the Python process with :program:`top` (i.e.
``top -p $(pgrep -d',' python)``). :func:`~prody.sequence.msafile.parseMSA`
function returns the alignment data in a NumPy :class:`~numpy.ndarray` with
character data type. When the array was deallocated, data buffer would remain
in the memory, and it would become a visible problem when dealing with 1 GB
file.

Problematic code
----------------

To parse alignment efficiently, I simply allocate memory large enough to store
the entire file. Once data is parsed, I use ``realloc`` to free the unused
portion and return the buffer in a NumPy :class:`~numpy.ndarray` to the user,
as partly shown here:

.. code-block:: c
   :linenos:

   // msaio.c
   static PyObject *parseSelex(PyObject *self, PyObject *args) {
       char *filename;
       long filesize;
       if (!PyArg_ParseTuple(args, "si", &filename, &filesize))
           return NULL;

       char *data = malloc(filesize * sizeof(char));
       if (!data) {
           return PyErr_NoMemory();
       }

       // ..., parsing, indexing, etc.

       // `index` is the number of chars copied to `data`
       data = realloc(data, index * sizeof(char));

       // determine the shape of 2D array
       npy_intp dims[2] = {index / seqlen, seqlen};
       // creates an array from the data, with desired shape
       PyObject *msa = PyArray_SimpleNewFromData(2, dims, PyArray_CHAR, data);

       // return `msa` ndarray and list/dict of labels
   }

And the Python code using it looks like:

.. code-block:: python
   :linenos:

   import os.path
   from msaio import parseSelex

   filesize = os.path.getsize(filename)
   msa, ... = parseSelex(filename, filesize)


The problem with this is that arrays created with
:c:func:`PyArray_SimpleNewFromData` *does not own* the data. It *is* noted in
its documentation that user is responsible for freeing the memory when the
:class:`~numpy.ndarray` is deallocated, but I guess I was rushing to have a
working piece of code and overlooked it. Several unit tests running on small
files did not let this issue appear either.

.. _documentation: http://docs.scipy.org/doc/numpy/user/c-info.how-to-extend.html#PyArray_SimpleNewFromData

Alternative implementation
--------------------------

Then I spent an entire morning to find a solution, but there was no way to
convince the :class:`~numpy.ndarray` that it owns the data entirely and
potential solutions on the internet led to writing more than a few lines of
code.

Alternatively, using Python to instantiate an :class:`~numpy.ndarray` large
enough to store the file and then working on the data buffer owned by that
array solved the issue with minimal changes in the code:

.. code-block:: c
   :linenos:

   // msaio.c
   static PyObject *parseSelex(PyObject *self, PyObject *args) {
       char *filename;
       PyArrayObject *msa;
       if (!PyArg_ParseTuple(args, "sO", &filename, &msa))
           return NULL;

       // get a pointer to the character buffer
       char *data = (char *) PyArray_DATA(msa);

       // ..., parsing, indexing, etc.

       // determine new size
       npy_intp dims[2] = {index / seqlen, seqlen};
       PyArray_Dims arr_dims;
       arr_dims.ptr = dims;
       arr_dims.len = 2;

       // resize as follows to free unused memory
       PyArray_Resize(msa, &arr_dims, 0, NPY_CORDER);

       // return `msa` ndarray and list/dict of labels
   }

And again, the code using it now looks like:

.. code-block:: python
   :linenos:

   from numpy import empty
   from msaio import parseSelex

   msa = empty(filesize)
   msa, ... = parseSelex(filename, msa)


This was the perfect fix__ with minimum changes in the code!

__ https://github.com/abakan/ProDy/commit/0c9c7d3e657c1383e6d1aa89f68ddde0eb2fe5e5


Reference counting
==================

Memory leak was gone, but segmentation fault problem persisted. This time, it
was related to reference counting. In particular, it was due to
:c:func:`PyList_SetItem` stealing reference counts. Once culprit was
identified, the `fix`__ was simply adding a line of :c:func:`Py_INCREF`.

__ https://github.com/abakan/ProDy/commit/452d056ba388303dc17128ca26febded23c0da70

The problem in this case emerged only when MSA file contained sequences with
same label.


Take home message
=================

Keep calm and carry on *testing* for the expected and unexpected. And, read C
API documentations carefully. There are more caveats in C APIs than there is in
Python APIs. It looks like it will be the best to rely entirely on NumPy C API
functions/macros rather then mixing it with C memory management.

MSA parser performance
======================

So, how fast is the parser? I parsed 1 GB data in 1.3s. At around 700 MB/s data
input rate, it may be as fast as it gets. Here is an example on how to use MSA
parser and classes in ProDy_ and Evol_:

.. code-block:: ipython

   In [1]: from prody import *

   In [2]: fetchPfamMSA('7tm_1', alignment='seed')
   Out[2]: '7tm_1_seed.sth'

   In [3]: msa = parseMSA('7tm_1_seed.sth')

   In [4]: msa['5HT1A_HUMAN']  # sequences are indexed during parsing
   Out[4]: <Sequence: 5HT1A_HUMAN (7tm_1_seed[21]; length 434; 348 residues and 86 gaps)>
