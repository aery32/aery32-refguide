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
   build_system
   examples

   system_clocks

   delay_functions
   string_functions

   contribute

Module functions
----------------

Modules are library components that operate straightly with the MCU's
internal peripheral modules.

.. toctree::
   :maxdepth: 2

   functions/naming_convention
   function/adc

Class drivers
-------------

Aery32 class drivers, abbreviated as *clsdrv*, are high level C++ classes.
Thus the name class driver. Class drivers commonly wrap bunch of module
functions into a nice package providing even more convenient and feature rich
APIs. You most likely want to use these when ever possible.

.. toctree::
   :maxdepth: 2

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

