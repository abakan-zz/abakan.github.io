Fast Parsing and Memory Efficiency
==================================

.. post:: Nov 10, 2013
   :tags: Python, ProDy, NumPy
   :category: ProDy

When designing ProDy_ PDB parser (:func:`~prody.proteins.pdbfile.parsePDB`) and
:mod:`~prody.atomic` classes, I primarily optimized the code for speed of
parsing, and memory usage was of lower importance. In many extreme cases
involving processing 100s of PDB_ structures (~10K atoms on average) in memory,
ProDy has done fine. This lasted until some users pushed it to the limits
recently. Their use case involved manipulating multiple trajectory frames
containing ~160K atoms. The bottleneck was ProDy's memory usage, and their
application kept crashing.

.. _PDB: http://www.pdb.org/

Speed and Memory Usage Optimization
-----------------------------------

Optimization of :func:`~prody.proteins.pdbfile.parsePDB` for speed involved
minimizing object instantiation and memory allocation/reallocation. To this
end, parser reads all lines in a given PDB_ file, and instantiates NumPy_
arrays with lengths equal to the number of lines. Once parsing is finished,
data lines fill in 85 to 90% of the allocated memory. User gets views of these
arrays for the used portion of the memory in
:class:`~prody.atomic.atomgroup.AtomGroup` instances.

I had also improved the parser to parse only well-defined subsets of atoms,
such as backbone (``parsePDB(filename, subset='backbone')``) or carbon alpha
(``subset='calpha'``) atoms, to make it even faster. However, when parsing
subsets, only less than 50% of the allocated memory would be used. For a
trajectory frame containing 160K atoms most of being water, using subsets would
allocate 30 to 40 times more memory than its needed. The fix for this was easy,
simply resizing data arrays by calling their :meth:`~numpy.ndarray.resize`
methods after parsing is finished. With release of v1.4.8, ProDy now uses just
enough memory to handle atomic data:

.. code-block:: ipython

   In [1]: from prody import parsePDB

   In [2]: %time all_atoms = parsePDB('1JJ2')
   CPU times: user 1.27 s, sys: 13 ms, total: 1.28 s
   Wall time: 1.29 s

   In [3]: %time ca_atoms = parsePDB('1JJ2', subset='ca')
   CPU times: user 529 ms, sys: 2 ms, total: 531 ms
   Wall time: 530 ms

   In [4]: all_atoms.numBytes() // ca_atoms.numBytes()
   Out[4]: 26


Pointer Classes and Serialization
---------------------------------

Of course an :class:`~prody.atomic.atomgroup.AtomGroup` class that encapsulates
all atomic data is not good enough for most analysis tasks. User needs to
access individual atoms or arbitrary subsets atoms. I handled this using
:class:`~prody.atomic.pointer.AtomPointer` classes, which simply holds a
reference to an :class:`~prody.atomic.atomgroup.AtomGroup` instance and a list
of indices.

:class:`~prody.atomic.atom.Atom`, :class:`~prody.atomic.residue.Residue`,
:class:`~prody.atomic.chain.Chain`, and
:class:`~prody.atomic.selection.Selection` classes derived from
:class:`~prody.atomic.pointer.AtomPointer` provide the same interface that
:class:`~prody.atomic.atomgroup.AtomGroup` does, i.e. same methods for
getting and setting data. The key for speed and efficiency is that they are
instantiated only when they are needed, for instance upon indexing of
:class:`~prody.atomic.atomgroup.AtomGroup` with an atom index or a residue
number.


In this design, both memory usage and speed were important considerations. So,
I used slots_ for optimizing both. Objects with slots don't have a dictionary
and this helps saving memory and making instantiation faster. This matters a
lot in applications where large number of instances are created (e.g. an
:class:`~prody.atomic.atom.Atom` instance for 160K of atoms). This
implementation looks like the following:

.. code-block:: python
   :linenos:

   class Atomic(object):
       """Base class for all classes handling atomic data."""
       __slots__ = []

   class AtomGroup(Atomic):
       """Class that encapsulates atomic data and coordinate arrays."""
       __slots__ = ['_data', '_coords', ] # etc.

   class AtomPointer(Atomic):
       """Base for classes that point to AtomGroup data,
       e.g. Atom, Residue, etc."""
       __slots__ = ['_ag', '_acsi']

   class Atom(AtomPointer):
       __slots__ = AtomPointer.__slots__ + ['_index'] # store atom index

   class Residue(AtomPointer):
       __slots__ = AtomPointer.__slots__ + ['_indices'] # store atom indices


The limitation of this is that objects become non-serializable (see
:mod:`pickle`).  Adding the following :meth:`~object.__getstate__` and
:meth:`~object.__setstate__` methods to the
:class:`~prody.atomic.atomic.Atomic` class completely resolved the limitation:

.. code-block:: python
   :linenos:

   class Atomic(object):
       """Base class for all classes handling atomic data."""
       __slots__ = []

       def __getstate__(self):
           return dict([(slot, getattr(self, slot))
                        for slot in self.__class__.__slots__])

       def __setstate__(self, state):
           for slot in self.__class__.__slots__:
               try:
                   value = state[slot]
               except KeyError:
                   pass
               else:
                   setattr(self, slot, value)

.. _slots: http://docs.python.org/3/reference/datamodel.html#slots

Well, actually both of these fairly easy optimizations and refinements were
something I had in mind for a while. I guess all I needed to make these
improvements was receiving some users complaints :)