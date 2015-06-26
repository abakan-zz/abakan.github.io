DruGUI - Druggability Suite
===========================

.. post:: Jul 17, 2013
   :tags: VMD, NAMD, DruGUI, Druggability

Recently, we had a hands-on `Computational Biophysics Workshop`_ in Pittsburgh.
First three days were on NAMD_/VMD_, and the last two were on
ProDy_/NMWiz_, and a new software called DruGUI_.

DruGUI is a VMD plugin that I developed for setup and analysis of
:wiki:`druggability` simulations. These simulations involve probing protein
binding sites with small organic solvent molecules (probes) that mimic drugs.
Simulations are performed in the presence of water, and the protein is fully
flexible. We showed that the method is able to identify allosteric and flexible
drug binding sites and give a reasonable estimate of the maximal achievable
affinity by a drug molecule. You can learn more from our paper published in
*Journal of Chemical Theory and Computation* ([AB12]_). Also, the lecture that
I gave at the hand-on workshop is available online (`Druggability lecture`_).

DruGUI plugin facilitates preparation of the protein-probe system ready to be
simulated using NAMD, and analysis of trajectories for identification of
druggable/ligandable binding sites. My plan is to get DruGUI incorporated into
VMD, after getting some feedback. So, give it a try, and let me know how it
works for you!


**DruGUI Links**

  * `Download & Installation <http://ahmetbakan.com/drugui/intro.html>`_
  * `Tutorial Online <http://ahmetbakan.com/drugui/index.html>`_
  * `Tutorial PDF <http://ahmetbakan.com/drugui/_downloads/DruGUI_Tutorial.pdf>`_
  * `Druggability lecture`_

**References**

.. [AB12] Bakan A, Nevins N, Lakdawala AS, Bahar I
   `Druggability Assessment of Allosteric Proteins by Dynamics Simulations in the Presence of Probe Molecules <http://pubs.acs.org/doi/abs/10.1021/ct300117j>`_
   *J Chem Theory Comput* **2012** 8(7):2435-2447

.. _NAMD: http://www.ks.uiuc.edu/Research/namd/
.. _VMD: http://www.ks.uiuc.edu/Research/vmd/

.. _Computational Biophysics Workshop: http://www.ks.uiuc.edu/Training/Workshop/Pittsburgh2013
.. _Druggability lecture: http://www.ks.uiuc.edu/Training/Workshop/Pittsburgh2013/lectures/day5/Bakan-Lecture-Druggability.pdf