Project structure -- where things go?
=====================================

Aery32 Software Framework provides a complete project structure to start
AVR32 development right away. You just start coding and adding your files.
The default project directory structure looks like this::

    projectname/
        aery32/
            ...
        examples/
            ...
        board.cpp
        board.h
        main.cpp
        Makefile
        settings.h

It is intended that you work under the root directory most of the time as
that is the place where you keep adding your .c, .cpp and .h source files.
Notice that Aery32 Framework is a C/C++ framework and thus you can write
your code in both C and C++. Just put the C code in .c files and C++ code
in .cpp files.

The following subsections define each part of the default project structure
in alphabetic order as they are listed above.

Aery32 library, ``aery32/``
---------------------------

The directory called ``aery32/`` contains the source files of the Aery32
library. The archive of the library (.a file) appears in this directory after
the first compile process. The ``aery32/`` subdirectory within the ``aery32/``
contains the header files of the library. Additionally, linker scripts,
which are essential files to define the MCU memory structure are kept under
this directory. Take a look at the ``ldscripts/`` subdirectory if you are
curious.

.. note ::

    Although you can, you should not need to hassle with any file under this
    directory.

Example programs, ``examples/``
-------------------------------

Aery32 Framework comes with plenty of example programs, which are placed
under the ``examples/`` directory. Every file is an independent program
that does not need other files to work. So it should work out of box if you
just replace the default ``main.cpp`` with the example.

To test, for example, the LED toggling demo open the Command Prompt
and command::

    cp examples\toggle_led.cpp main.cpp
    make programs

The following lines of commands overwrite the present ``main.cpp`` file
from the project root with the example, compiles the project and uploads
the new binary to the development board. The program starts running
immediately.

.. note ::

    The quickest way to access Command Prompt in Windows is to press
    Windows-key and R (Win+R) at the same time, and type cmd.

.. note ::

    You are free to remove the ``examples/`` directory from your project
    if you don't need it.

Main source file, ``main.cpp``
------------------------------

The ``main.cpp`` source file contains the default main function where to
start building your project. First the ``board.h`` header file has been
included. This file includes your application specific function prototypes,
which are defined in ``board.cpp``. For your convenience a small
``board::init()`` function is provided by default. This function is called
within the main function at line 11. It's the first function call here.
The second call is to set the LED pin high.

.. code-block:: c++
    :linenos:

    #include "board.h"
    using namespace aery;

    int main(void)
    {
        /*
         * Put your application initialization sequence here. The default
         * board initializer defines the LED pin as output and sets the CPU
         * clock speed to 66 MHz.
         */
        board::init();
        gpio_set_pin_high(LED);

        for(;;) {
            /* Put your application code here */

        }

        return 0;
    }

Board specific functions, ``board.h`` and ``.cpp``
--------------------------------------------------

Use these two files for your board specific functions and macro definitions.
A macro definition is for example the following pin declaration

.. code-block:: c++

    #define LED    AVR32_PIN_PC04

Now you don't have to always recall which pin the LED was connected when
you want to switch it on. So instead of using this

.. code-block:: c++

    aery::gpio_set_pin_high(AVR32_PIN_PC04);

you can use this

.. code-block:: c++

    aery::gpio_set_pin_high(LED);

You can find this default LED macro definition from ``board.h``. There are
also other default board related definitions, which you may need to change
according to your project. Those are for example

.. code-block:: c++

    #define ADC_VREF    3.0
    #define ADC_BITS    10

These two macro definitions are related to the analog to digital converter
(ADC). To change the reference voltage of the ADC, modify the ``ADC_VREF``.
Similarly if you decide to use, for example, only eight bits accuracy alter
``ADC_BITS`` accordingly. You may like to reduce the accuracy in favor
throughput rate of the analog to digital converter. With smaller accuracy
ADCs generally work faster.

From these ADC related settings, we get to one of the functions declared in
the default version of ``board.h``. That's ``board::cnv2volt()``. This
function has not been declared in a library because it's highly dependant
of what's the reference voltage and accuracy of ADC. Notice that this function
uses ``ADC_VREF`` and ``ADC_BITS`` internally to calculate the correct voltage
for the conversion.

It's intended that you define all your board related functions in ``board.h``
and then implement those in ``board.cpp``. For example, if you had a device
which to communicate via SPI, you could write a function like this

.. code-block:: c++

    uint8_t board::write_to_device(uint8_t byte)
    {
        return aery::spi_transmit(spi0, 2, byte);
    }

See how the above function abstracts which SPI and slave select you are using?

Default board initializer
'''''''''''''''''''''''''

The default board initializer function, ``board::init()``, can be found from
the ``board.cpp`` source file. The prototype of this function is declared
in ``board.h``.

Here's what it basicly does by default

- Sets all GPIO pins inputs
- Defines LED pin as output
- Starts the external oscillator ``OCS0``
- Sets the chip's master (or main) clock frequency to its maximum,
  which is 66 MHz

If you like to change the master clock frequency or want to change the way
how the board is initialized, ``board::init()`` is the place where to do it.

.. note::

    All board related functions should use a namespace ``board`` to not
    introduce any name collision with other functions added into the project.

Build system, ``Makefile``
--------------------------

Makefile contains all the make recipes for compiling the project and uploading
the compiled binary to the board. See more detailed instructions
from the :doc:`build system <build_system>` section.

.. note ::

    Generally Makefiles don't have a file postfix like ``.cpp`` and it's
    a common practice to start its name with capital M.

Project wide settings, ``settings.h``
-------------------------------------

In this file you can define project wide global settings. Aery32 Framework
is also aware some of the settings defined in this file. For example, to get
the delay functions work properly you have to define the correct CPU frequency,
``F_CPU``, in this file. Below you can see how some essential settings have
been defined.

.. code-block:: c++

    #define F_OSC0 12000000UL
    #define F_OSC1 16000000UL
    #define F_CPU  66000000UL

.. note ::

    This file is provided to GCC via ``-include``
