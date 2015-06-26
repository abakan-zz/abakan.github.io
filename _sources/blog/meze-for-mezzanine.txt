Meze for Mezzanine
==================

.. post:: Jun 10, 2013
   :tags: Meze, Mezzanine, Django, Python, Sphinx
   :category: Meze

I have been using Sphinx_ for documenting my open-source Python packages and
also maintaining a personal site. It is a great tool that handles code
highlighting, cross-referencing, and hierarchical structures, and conversion of
content in reStructuredText_ to HTML, PDF, and other formats. For documenting
ProDy_ package, it has been extremely useful, but for my personal site, I
thought, time has come for a change.

Recently, I started learning Django_ and came across Mezzanine_
:wiki:`content management system` (CMS). It is an excellent CMS out of the box,
but before I could make the switch to it for maintaining my personal site, I
wanted to be able to write in reStructuredText and use some Sphinx extensions.
To achieve this, I developed a small app called Meze_, which brings Sphinx_
flavors to Mezzanine_.

Meze adds ``source`` field to blog post and page models of Mezzanine_ for
storing reStructuredText. It converts this into HTML using Sphinx with minimal
file I/O operations (in a fraction of a second) and stores output in rich text
field. HTML code in this field then can be further edited by the user. Here, I
show only some features of Sphinx that I could not live without.


Code snippets
-------------

One of the most attractive features of Sphinx_ is that it uses Pygments_ for
syntax highlighting. I like the look of outcome and that it does not require
any client side code. I wanted to be able to use it, as I anticipate that most
of my posts will include snippets. Here is an example which is interpreted as
Python code:

.. code-block:: none

   >>> from prody import *
   >>> p = parsePDB('1mkp')

This will display as follows:

>>> from prody import *
>>> p = parsePDB('1mkp')

It is also possible to highlight code in many other languages, such as Ruby:

.. code-block:: none

   .. code-block:: ruby
      :linenos:

      -199.abs # some things are easier in ruby
      "Nice Day!".downcase.split("").uniq.sort.join

And, this is what you will see:

.. code-block:: ruby
   :linenos:

   -199.abs # some things are easier in ruby
   "Nice Day!".downcase.split("").uniq.sort.join

See more on displaying code examples `here
<http://sphinx-doc.org/markup/code.html>`_.

External links
--------------

extlinks_ and intersphinx_ are two Sphinx extensions that have been
indispensable when writing ProDy_ documentation and tutorials.  You can use
these to link external websites/databases and other projects' documentation
with *minimal text*. For example:

  Random numbers from :wiki:`normal distribution` can be drawn using
  :func:`numpy.random.normal`.

The following is what you need to write to get the links:

.. code-block:: rst

  Random numbers from :wiki:`normal distribution` can be drawn using
  :func:`numpy.random.normal`.

And, you can give an example as follows:

>>> from numpy.random import normal
>>> sample = normal(size=100000)
>>> sample.mean()
0.00074603441644642131
>>> sample.std()
0.99862148165643938

To be continued
---------------

Well, now I have no reason not to post more frequently. I plan to show more
examples on what one can do with Meze soon.

.. _Pygments: http://pygments.org/
.. _reStructuredText: http://docutils.sourceforge.net/rst.html
.. _extlinks: http://sphinx-doc.org/ext/extlinks.html
.. _intersphinx: http://sphinx-doc.org/ext/intersphinx.html