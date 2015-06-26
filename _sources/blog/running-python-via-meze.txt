Running Python via Meze
=======================

.. post:: Jul 12, 2013
   :tags: Django, Meze, Mezzanine, Python, Sphinx, IPython, Matplotlib
   :category: Meze


`IPython directive`_ is a very handy Sphinx_ extension for writing tutorials
and documentation. In fact, I used it extensively when developing ProDy_
tutorials. With this directive, you just let Sphinx run a piece of code and
incorporate output into the output HTML. It even allows for making plots and
saving image files automatically. This post shows how to use it via Meze_.

.. _IPython directive: http://matplotlib.org/sampledoc/ipython_directive.html


IPython directive
-----------------

To illustrate this, let's generate a sample of random numbers using
:func:`~numpy.random.normal` function and make a histogram:

.. ipython::

   In [1]: from pylab import *

   In [1]: from numpy.random import normal

   In [1]: sample = normal(size=10000)

   In [1]: figure();

   In [1]: hist(sample);

   @savefig sample_normal_distribution.png width=4in
   In [1]: title('Normal Distribution (sample size = 10000)');


Here is the source for the example:

.. code-block:: rst

   .. ipython::

      In [1]: from pylab import *

      In [1]: from numpy.random import normal

      In [1]: sample = normal(size=10000)

      In [1]: figure();

      In [1]: hist(sample);

      @savefig sample_normal_distribution.png width=4in
      In [1]: title('Normal Distribution (sample size = 10000)');

Note that you need to leave a blank line between each input line in the
directive. Semicolons following :func:`~matplotlib.pyplot.figure`,
:func:`~matplotlib.pyplot.hist`, and :func:`~matplotlib.pyplot.title` suppress
the outputs of these functions.


Sphinx configuration
--------------------

To start using IPython directive, you need to install Matplotlib_, and append
the following lines to your ``SPHINX_CONF`` variable in Mezzanine_
:file:`settings.py` file:

.. code-block:: python

   SPHINX_CONF = """
   # IPython extensions come with Matplotlib
   extensions = [
       ...
       'matplotlib.sphinxext.ipython_console_highlighting',
       'matplotlib.sphinxext.ipython_directive']

   # folder for saving figures
   ipython_savefig_dir = '/path/to/public_html/static/media/uploads'
   """

``ipython_savefig_dir`` should point to a folder visible from the web. I have
set it to the folder that `Mezzanine` stores uploaded files, so that I will be
able to remove files via the admin interface when needed.


Security risks
--------------

Needless to say, ability to run Python on your server can have severe
consequences. For a personal blog with a single user, like this one, it should
be safe. Implementing permissions to use Meze_, however, will be a good
addition in a future release for multi-user blogs. Until then, post
responsibly.