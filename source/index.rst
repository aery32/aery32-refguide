.. aery32's documentation master file, created by
   sphinx-quickstart on Tue Dec 06 17:47:29 2011.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to Aery32's documentation!
==================================

.. toctree::
   :maxdepth: 2

   getting_started
   project_structure
   examples
   build_system
   delay_functions
   module_functions
   string_functions
   system_clocks
   contribute

Class drivers
-------------

Aery32 class drivers, abbreviated as *clsdrv*, are high level C++ classes.
Thus the name class driver. Class drivers commonly use low level module
functions to provide even more convenient and feature rich peripheral APIs.
**You most likely want to use thes when ever possible.**

.. toctree::
   :maxdepth: 1

   clsdrv/periph_iodma
   clsdrv/serial_port

Aery32 framework in your favorite editor
----------------------------------------

Aery32 framework is not restricted to any specific integrated development
environment (IDE) or editor. We respect your taste. The best one is the one
you like best and to figure out which one you like best you have to try a few.
We like Sublime Text 2 and thus the framework comes with a preset project-file
for this editor. However, we have kindly made instructions to use Aery32 in
Eclipse as well :)

.. toctree::
  :maxdepth: 2

  use_with_eclipse
  use_with_st2

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

