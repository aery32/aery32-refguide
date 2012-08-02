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

It is intended that you work under the root directory most of the time as that is the place where you keep adding your .c source files and .h header files.

**main.cpp**

The ``main.cpp`` source file contains the default main function where to start.

.. code-block:: c++
    :linenos:

    #include "board.h"
    #include <aery32/all.h>

    using namespace aery;

    int main(void)
    {
        /* Put your application initialization sequence here */
        init_board();
        gpio_init_pin(LED, GPIO_OUTPUT);

        /* All done, turn the LED on */
        gpio_set_pin_high(LED);

        for(;;) {
            /* Put your application code here */

        }

        return 0;
    }

**board.h**

This is a place for the board specific function prototypes and supportive ``#define`` macros. These macros provide a way to do configuration, for example.

**board.cpp**

The default board initialization function, ``init_board()``, can be found here. First it sets all the GPIO pins to inputs. Then it configures the board's power manager. Basicly the external oscillator ``OCS0`` is started and the master clock frequency is set to 66 MHz. If you like to change the master clock frequency or want to change the way how the board is initialized, this is the place where to do it.

**aery32/**

This directory contains the source files of Aery32 library. The archive of the library (.a file) appears in this directory after the first compile process. The ``aery32/`` subdirectory contains the header files of the library. Linker scripts, which are essential files to define the MCU memory structure are placed under the ``ldscripts/`` directory. However, you should not need to hassle with those files.

**examples/**

All the example programs are placed under this directory. Every program is completely independent. To test one you can just replace the ``main.c`` with one of the example programs and it works out of box when uploaded into the board. Otherwise if you do not need these files you can remove this folder.


Makefile
--------

Makefile enables the project build process and provides the convenient way to upload the compiled application into the board by using in-system programming bus. To compile the project just command::

    make

or::
    
    make all

To clean the project folder from binaries call::

    make clean

and to recompile all the files::

    make re

When you are ready to upload the program into the board type::

    make program

If you also want to start the program immediately type::

    make program start

or in shorter format::

    make programs

How to introduce new source files in the Makefile
'''''''''''''''''''''''''''''''''''''''''''''''''

Let's say I would like to separate my source code into a ``my/`` subdirectory under the project root. After creating the directory, I have to edit the Makefile. So, open the Makefile into your editory and find the line::

    SOURCES=$(wildcard *.cpp) $(wildcard *.c)

Edit this line so that it looks like this::

    SOURCES=$(wildcard *.cpp) $(wildcard *.c) $(wildcard my/*.cpp)

You can also add single ``.c`` or ``.cpp`` files at the end of this list.

Example programs
----------------

Aery32 Framework comes with plenty of example programs, which **work out of box**. To test, for example, the LED toggling demo do the following:

**In Windows**

Open Command Prompt and command::

    cp examples\toggle_led.cpp main.cpp
    make programs

The quickest way to access Command Prompt is to press Windows-key and R (Win+R) at the same time, and type cmd.

**In Linux**

Open terminal and::

    cp examples/toggle_led.cpp main.cpp
    make programs

The following lines of commands overwrite the present ``main.cpp`` with the example and the uploads (or programs) it into the development board. The program starts running immediately.

.. note::

  Every example program consists from a single file and can be found from ``examples/`` directory.
