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

Aery32 class drivers are high level C++ classes. Thus the name class driver.
These drivers commonly use low level module functions to provide more
convenient and feature richer APIs to peripherals. You most likely want
to use these in your application rather than lower level functions.

.. toctree::
   :maxdepth: 1

   clsdrv/periph_iodma
   clsdrv/serial_port

Aery32 Framework in your favorite editor
========================================

.. toctree::
  :maxdepth: 2

  use_with_eclipse
  use_with_st2

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

