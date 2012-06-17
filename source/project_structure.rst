Project structure -- where things go?
=====================================

Aery32 Software Framework provides a complete project structure to start AVR32 development right away. You just start coding and adding your files. The default project directory structure, if downloaded from GitHub, looks like this::

    projectname/
        aery32/
            ...
        examples/
            ...
        board.c
        board.h
        main.c
        Makefile

It is intended that you work under the root directory most of the time as that is the place where you keep adding your .c source files and .h header files.

**main.c**

.. code-block:: c
    :linenos:

    #include <stdbool.h>
    #include <aery32/gpio.h>
    #include "board.h"

    #define LED AVR32_PIN_PC04

    int main(void)
    {
        /* Put your application initialization sequence here */
        init_board();
        aery_gpio_init_pin(LED, GPIO_OUTPUT);

        /* All done, turn the LED on */
        aery_gpio_set_pin_high(LED);

        for(;;) {
            /* Put your application code here */

        }

        return 0;
    }

The ``main.c`` source file contains the default main function where to start. At the top of the file couple of header files are also included in advance. For example, you most probably are going to use general peripheral input and output pins so ``"aery32/gpio.h"`` has been included.

**board.h**

This is a place for the board specific function prototypes and supportive ``#define`` macros, which provide a way to do configuration. The board initialization functions has been already implemented and can be located inside of ``board.c``.

**board.c**

The default board initialization function can be found here. First it sets all GPIO pins to be inputs. Then it configures the board's power manager: starts the oscillator and clocks the master clock to 66 MHz. When you are changing the way how the board is initialized this is the place where to do it.

**aery32/**

Contains the source files of Aery32 library. The archive of the library appers in this directory after the first compile process. The ``aery32/`` subdirectory contains the header files, where you can find plenty of documentation. You can also find the linker scripts here, which are essential files to define the MCU memory structure in liking process. However, you should not need to hassle with these files.

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

Let's say I would like to separate my source code into a ``my/`` directory under the project root. After creating the directory I have to edit Makefile to get the source files under this directory compiled. So, open Makefile into your editory and find the line::

    SOURCES=$(wildcard *.c)

Edit this line so that it looks like this::

    SOURCES=$(wildcard *.c) $(wildcard my/*.c)

You can also add single .c files at the end of this list.

Example programs
----------------

Aery32 Framework comes with plenty of example programs, which **work out of box**. To test, for example, the LED toggling demo do the following:

**In Windows**

Open Command Prompt and command::

    cp examples\toggle_led.c main.c
    make programs

The quickest way to access Command Prompt is to press Windows-key and R (Win+R) at the same time, and type cmd.

**In Linux**

Open terminal and::

    cp examples/toggle_led.c main.c
    make programs

The following lines of commands overwrite the present ``main.c`` with the example and the uploads (or programs) it into the development board. The program starts running immediately.

.. note::

  Every example program consists from a single file and can be found from ``examples/`` directory.


Where is my C++?
----------------

To use C++ you have to change the `avr32-gcc` compiler to `avr32-g++`. This can be done by editing the Makefile. Find the following line under `Standard user variables` section::

    CC=avr32-gcc

and replace it with::

    CC=avr32-g++

Also change the line below::

    CSTD=gnu99

to::

    CSTD=gnu++98

Or if you feel more experimental, you can chooce one of these: c++0x or gnu++0x.

Now you can use C++ in your project. Remember to use the ``.hh`` header files instead of ``.h`` files. For example, instead of using

.. code-block:: c

    #include >aery32/gpio.h>

use

.. code-block:: c

    #include <aery32/gpio.hh>

At the moment Aery32 Software Framework uses only the C++ namespaces. The benefits of using namespace is that you can omit the *aery_* prefix in the function calls. This has been demonstrated below::

.. code-block:: c
    :linenos:

    #include <stdbool.h>
    #include <aery32/gpio.hh>
    #include "board.h"

    #define LED AVR32_PIN_PC04

    using namespace aery;   // enable aery namespace

    int main(void)
    {
        init_board();
        gpio_init_pin(LED, GPIO_OUTPUT|GPIO_HIGH); // yay! no "aery_" prefix

        for(;;) {
            /* Put your application code here */

        }

        return 0;
    }
