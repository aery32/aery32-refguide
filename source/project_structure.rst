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

To test, for example, the LED toggling demo do the following:

Open Command Prompt and command::

    cp examples\toggle_led.cpp main.cpp
    make programs

The following lines of commands overwrite the present ``main.cpp`` file
with the example and uploads (or programs) it into the development board.
The program starts running immediately.

.. note ::

    The quickest way to access Command Prompt in Windows is to press
    Windows-key and R (Win+R) at the same time, and type cmd.

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

Board specific functions, ``board.h`` and ``board.cpp``
-------------------------------------------------------

The default board initializer function, ``board::init()``, can be found from
the ``board.cpp`` source file. First it sets all the GPIO pins to inputs.
Then it configures the board's power manager. Basically by starting the
external oscillator ``OCS0`` and setting the chip's master (or main) clock
frequency to its maximum, which is 66 MHz. If you like to change the master
clock frequency or want to change the way how the board is initialized,
``board::init()`` is the place where to do it.

In ``board.h`` you define the board specific function prototypes and other
supportive ``#define`` macros. These macros enables a way to do configuration.
For example, the LED pin has been defined to be connected to the pin 4 at
PORTC.

Build system, ``Makefile``
--------------------------

Makefile contains all the make recipes to compile the project and uploading
the compiled binary to the board. See more detailed instructions
from the :doc:`build system <build_system>` section.

.. note ::

    Genrally Makefiles don't have a file prefix like ``.cpp`` etc. and it's
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
