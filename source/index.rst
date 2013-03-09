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

Module functions are library components that operate straightly with the
MCU's internal peripheral modules.

.. toctree::
   :maxdepth: 1

   Naming conventions <functions/naming_conventions>
   General Periheral Input/Output <functions/gpio>
   Power Manager <functions/pm>
   Flash Controller <functions/flashc>
   Interrupt Controller <functions/intc>

   Analog-to-Digital Converter <functions/adc>
   Pulse Width Modulation <functions/pwm>
   Real-time Counter <functions/rtc>
   Serial Peripheral Interface <functions/spi>
   Two-wire (I2C) Interface <functions/twi>
   Universal Sync/Asynchronous Receiver/Transmitter <functions/usart>

Class drivers
-------------

Aery32 class drivers, abbreviated as *clsdrv*, are high level C++ classes.
Thus the name class driver. Class drivers commonly wrap bunch of module
functions into a nice package providing even more convenient and feature rich
APIs. You most likely want to use these when ever possible.

.. toctree::
   :maxdepth: 1

   Input/Output DMA <clsdrv/periph_iodma>
   Serial port <clsdrv/serial_port>

Aery32 framework in your favorite editor
----------------------------------------

Aery32 framework is not restricted to any specific integrated development
environment (IDE) or editor. We respect your taste. The best one is the one
you like best and to figure out which one you like best you have to try a few.
We like Sublime Text 2 and thus the framework comes with a preset project-file
for this editor. However, we have kindly made instructions to use Aery32 in
Eclipse as well :)

.. toctree::
  :maxdepth: 1

  use_with_eclipse
  use_with_st2

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

