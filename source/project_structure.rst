Project structure -- where things go?
=====================================

Aery32 Software Framework provides a complete project structure to start
AVR32 development right away. You just start coding and adding your files.
The default project directory structure looks like this::

    projectname/
        aery32/
            ...
        board.cpp
        board.h
        main.cpp
        Makefile

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

Main source file, ``main.cpp``
------------------------------

The main.cpp source file contains the default main function where to
start building your project. First the board.h header file has been
included. This file includes your application specific function prototypes,
which are defined in board.cpp. For your convenience a small
``board::init()`` function is provided by default. This function is called
within the main function at line 15 and is the first function call.
The second function call before the empty main loop sets the LED pin high.

.. code-block:: c++
    :linenos:

    #include <aery32/all.h>
    #include "board.h"

    using namespace aery;

    #define LED AVR32_PIN_PC04

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

All board specific functions and macro definitions should be placed in
``board.h`` header file. A macro definition is for example the following pin
declaration

.. code-block:: c++

    #define LED    AVR32_PIN_PC04

By doing this you don't need to remember which pin the LED was connected when
you want to switch it on. So instead of using this

.. code-block:: c++

    aery::gpio_set_pin_high(AVR32_PIN_PC04);

you can use this

.. code-block:: c++

    aery::gpio_set_pin_high(LED);

It's intended that you define all your board related functions in board.h
and then implement those in board.cpp. :doc:`Example programs <examples>`
coming with the framework are built in one file with the main function in
purpose, but when used in real application those should be refactored into
board.h and .cpp. For example, consider that you had a device which to
communicate via SPI. To take an advance of the board abstraction you could
write the following board specific function in board.h

.. code-block:: c++

    inline uint8_t board::write_to_device(uint8_t byte)
    {
        return aery::spi_transmit(spi0, 2, byte);
    }

See how the above function abstracts which SPI peripheral number and slave
select your device is connected.

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
how the board is initialized, board::init() is the place where to do it.

.. note::

    All board related functions should use a namespace ``board`` to not
    introduce any name collision with other functions added into the project.

Frequency settings
''''''''''''''''''

.. code-block:: c++

    #define F_OSC0 12000000UL
    #define F_OSC1 16000000UL
    #define F_CPU  66000000UL

These three macro definitions are related to the board operating frequency.
If you choose to change the board CPU frequency, make sure to redefine
it in the board.h, or otherwise delay functions won't work as expected.

Build system, ``Makefile``
--------------------------

Makefile contains all the make recipes for compiling the project and uploading
the compiled binary to the board. See more detailed instructions
from the :doc:`build system <build_system>` section.

.. note ::

    Generally Makefiles don't have a file postfix like ``.cpp`` and it's
    a common practice to start its name with capital M.
