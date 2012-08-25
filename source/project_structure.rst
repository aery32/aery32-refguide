Project structure -- where things go?
=====================================

Aery32 Software Framework provides a complete project structure to start AVR32 development right away. You just start coding and adding your files. The default project directory structure looks like this::

    projectname/
        aery32/
            ...
        examples/
            ...
        board.cpp
        board.h
        main.cpp
        Makefile

It is intended that you work under the root directory most of the time as that is the place where you keep adding your .c, .cpp and .h source files. Notice that Aery32 Framework is a C/C++ framework and thus you can write your code in both C and C++. Just put the C code in .c files and C++ code in .cpp files.

Main source file, ``main.cpp``
------------------------------

The ``main.cpp`` source file contains the default main function where to start building your project.

.. code-block:: c++
    :linenos:

    #include "board.h"
    #include <aery32/all.h>

    using namespace aery;

    int main(void)
    {
        /*
         * Put your application initialization sequence here. The default
         * board_init() sets up the LED pin and the CPU clock to 66 MHz.
         */
        init_board();

        /* All done. Turn the LED on. */
        gpio_set_pin_high(LED);

        for(;;) {
            /* Put your application code here */

        }

        return 0;
    }

Board specific functions, ``board.h`` and ``board.cpp``
-------------------------------------------------------

The default board initialization function, ``init_board()``, can be found from the ``board.cpp`` source file. First it sets all the GPIO pins to inputs. Then it configures the board's power manager. Basically by starting the external oscillator ``OCS0`` and setting the chip's master (or main) clock frequency to its maximum, which is 66 MHz. If you like to change the master clock frequency or want to change the way how the board is initialized, ``init_board()`` is the place where to do it. In ``board.h`` you define the board specific function prototypes and supportive ``#define`` macros. These macros enables a way to do configuration. For example, the LED pin has been defined to be connected to the pin PC04. However, keep your #definitions sane.

Aery32 library
--------------

The directory called ``aery32/`` contains the source files of the Aery32 library. The archive of the library (.a file) appears in this directory after the first compile process. The ``aery32/`` subdirectory within the ``aery32/`` contains the header files of the library. Linker scripts, which are essential files to define the MCU memory structure are placed under the ``ldscripts/`` directory. Although you can, you should not need to hassle with those files.

Examples
--------

Aery32 Framework comes with plenty of example programs, which are placed under the ``examples/`` directory. Every file should *work out of box* when copied over the project main. To test, for example, the LED toggling demo do the following:

**In Windows**

Open Command Prompt and command::

    cp examples\toggle_led.cpp main.cpp
    make programs

The quickest way to access Command Prompt is to press Windows-key and R (Win+R) at the same time, and type cmd.

**In Linux**

Open terminal and::

    cp examples/toggle_led.cpp main.cpp
    make programs

The following lines of commands overwrite the present ``main.cpp`` file with the example and the uploads (or programs) it into the development board. The program starts running immediately.

