Using Travis-CI for Testing
===========================

.. post:: Jul 31, 2013
   :tags: Python, ProDy, GitHub, Testing
   :category: ProDy

I too have the following cool image on my `ProDy repository`_ on GitHub_:

  .. image:: https://secure.travis-ci.org/abakan/ProDy.png?branch=master
     :target: http://travis-ci.org/#!/abakan/ProDy


I kept noticing this on more and more repositories, lately. The image showing
build status is provided by `Travis CI`_, a hosted, distributed
:wiki:`continuous integration` service.

`Travis CI`_ performs automated tasks, such as building the software and
running unit tests, after every push to selected GitHub_ repositories.
Status of builds can be displayed anywhere on the web, and developers can be
notified via email and
`other channels <http://about.travis-ci.org/docs/user/notifications/>`_.
It is also possible to
`skip <http://about.travis-ci.org/docs/user/how-to-skip-a-build/>`_ a CI when a
commit does not change the source code.


This will help me reduce testing that I perform before making a new ProDy
release. Here, I describe the initial setup I used and some optimization I
performed to cut the overall build time by half.

Initial setup
-------------

The initial setup was easier than I expected it to be. I just signed in to
`Travis CI`_ with my GitHub_ account by clicking the link at the top of their
homepage. Then, I was directed to a page where I turned Travis service hook for
my projects, after waiting a little for the service to setup my account.

After I turned the service on for ProDy, I needed to include a configuration in
:file:`.travis.yml` file in the repository root directory:

.. code-block:: yaml

   language: python
   python:
     - 2.7
     - 3.2
   install:
     - pip install --use-mirrors numpy
     - if [[ $TRAVIS_PYTHON_VERSION == '2.7' ]]; then pip install --use-mirrors matplotlib; fi
   before_script:
     - python setup.py build install
   script:
     - prody test -l full


This file tells Travis that the language of the project is Python, and I want
the tests to be run using Python versions 2.7 and 3.2. ``install`` commands
install required packages, ``before_script`` commands install ProDy.  Finally,
``script`` commands run the tests.

Note that I install `matplotlib`_ only when running tests with Python 2.7,
since its installation with Python 3.2 failed. Something I will take care of
later.

Optimization
------------

ProDy_ API tests run fast. Over 500 unit tests take under 1 to 3 seconds to
finish, depending on the machine. Also, running tests for ProDy applications
increase this up to a minute. Building on `Travis CI`_ using the initial setup
took much longer with more than 4 minutes. Most of the time was spent on
installing NumPy_ and Matplotlib_ packages using pip_.

I did the following optimizations to speed up setup process:

  1. I used `virtualenv` :option:`--system-site-packages` option to skip
     installation of some of the requirements, i.e. NumPy.

  2. I used ``sudo apt-get`` to install required packages. Using pip_ takes a
     while to compile Numpy and Matplotlib. Using apt-get helped saving around
     a minute for each.

Here is the configuration with optimizations:

.. code-block:: yaml

   language: python
   python:
     - 2.7
     - 3.2
   virtualenv:
     system_site_packages: true
   install:
     - sudo apt-get update -qq
     - if [[ $TRAVIS_PYTHON_VERSION == "2.7" ]]; then sudo apt-get install python-matplotlib; fi
     - if [[ $TRAVIS_PYTHON_VERSION == "3.2" ]]; then sudo apt-get install python3-dev python3-numpy; fi
     - python setup.py build install
   script:
     - prody test -l full

The following are the timings before and after optimizations:

  ======  ======  =====
  Python  Before  After
  ======  ======  =====
  2.7     4:12s   2:10s
  3.2     4:57s   1:3s
  ======  ======  =====

Repeated builds gave similar results.

References
----------

It looks like :wiki:`Travis CI` has been around for more than a year. It became
more popular in 2012 after a crowd funding campaign to fund its further
development. So, it was not hard to find blog posts that helped me to learn
more about the service and figure out the optimizations:

  * A `blog post <http://danielnouri.org/notes/2012/11/23/use-apt-get-to-install-python-dependencies-for-travis-ci/>`_ from Daniel Nouri
  * `Another post <http://justcramer.com/2012/05/03/using-travis-ci/>`_ from David Cramer
  * Also, Travis CI docs for `Python projects <http://about.travis-ci.org/docs/user/languages/python/>`_


.. _virtualenv: http://www.virtualenv.org/
.. _Travis CI: https://travis-ci.org/
.. _ProDy repository: https://github.com/abakan/ProDy