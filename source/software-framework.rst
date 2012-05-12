Aery32 Software Framework
=========================

Project structure -- where things go?
-------------------------------------

Aery32 Software Framework provides a complete project structure to start AVR32 development right away. You just start coding and adding your files. The default project directory structure looks like this::

    examples/
    include/
        aery32/
    ldscripts/
    lib/
    src/
        aery32/
        board.c
        board.h
        main.c
    .gitignore
    Doxyfile
    LICENSE.txt
    Makefile
    README.md

**examples/**

All the example programs are placed under this directory. Every program is completely independent. To test one you can just replace the ``src/main.c`` with one of the example programs and it works out of box when uploaded into the board. Otherwise if you do not need these files you can remove this folder.

**include/**

Contains only Aery32 Software Framework header files -- there are plenty of documentation in these files too. This is also the place for the vendor header files you might need in your project.

**ldscripts/**

This is a place for linker scripts -- essential files to define the mcu memory structure in liking process. You should not need to hassle with these files.

**lib/**

The archive of the compiled aery32 source files appers in this directory after the first compile process.  The ``lib/`` folder is also related with the vendor stuff as external libraries you may be using can be dropped there -- the Makefile scans the folder for the libraries.

**src/**

It is intended that you work under the ``src/`` directory most of the time as that is the place where you keep adding your .c source files and .h header files. You can also place your header files under ``include/`` if you like it that way.

``board.c``, ``board.h`` and ``main.c`` under the ``src/`` folder provides a starting point for your project. These files are not copyrighted so what ever you write into these files is your property and you are allowed to copyright it for yourself.

`src/main.c`

.. code-block:: c
    :linenos:

    #include <stdbool.h>
    #include "aery32/gpio.h"
    #include "board.h"

    #define LED AVR32_PIN_PC04

    int main(void)
    {
        init_board();
        aery_gpio_init_pin(LED, GPIO_OUTPUT|GPIO_HIGH);

        for(;;) {
            /* Put your application code here */

        }

        return 0;
    }

The ``main.c`` source file contains the default main function where to start. At the top of the file couple of header files are also included in advance. For example, you most probably are going to use general peripheral input and output pins so ``"aery32/gpio.h"`` has been included.

`src/board.h`

.. code-block:: c
    :linenos:

    #ifndef __BOARD_H
    #define __BOARD_H

    #ifdef __cplusplus
    extern "C" {
    #endif

    #define OSC0_FREQ 12000000UL

    #define HSBMASK_DEFAULT 0xFFFFFFFF;
    #define PBAMASK_DEFAULT 0xFFFFFFFF;
    #define PBBMASK_DEFAULT 0xFFFFFFFF;

    void init_board(void);

    #ifdef __cplusplus
    }
    #endif

    #endif

This is a place for the board specific function prototypes and supportive #defines, which provide a way to do configuration. The board initialization functions has been already implemented and can be located inside of ``src/board.c``.

`src/board.c`

.. code-block:: c
    :linenos:

    #include <stdbool.h>
    #include "aery32/pm.h"
    #include "aery32/gpio.h"
    #include "board.h"

    void init_board(void)
    {
        /* Initialize all pins input */
        aery_gpio_init_pins(
            porta,      /* pointer to port address */
            0xffffffff, /* pin mask */
            GPIO_INPUT  /* option flags */
        );
        aery_gpio_init_pins(portb, 0xffffffff, GPIO_INPUT);
        aery_gpio_init_pins(portc, 0x0000003f, GPIO_INPUT);

        /* Start oscillator */
        aery_pm_start_osc(
            0,                  /* oscillator number */
            PM_OSC_MODE_GAIN3,  /* oscillator mode, see datasheet p.74 */
            PM_OSC_STARTUP_36ms /* oscillator startup time */
        );

        aery_pm_wait_osc_to_stabilize(0 /* oscillator number */);

        /* Initialize f_vco0 of PLL0 to be 138 MHz. */
        aery_pm_init_pllvco(
            pll0,               /* pointer to pll address */
            PM_PLL_SOURCE_OSC0, /* source clock */
            11,                 /* multiplier */
            1,                  /* divider */
            false               /* high frequency */
        );

        /* Enable PLL0 with divide by two block to set f_pll0 to f_vco0 / 2
         * or 66 MHz. */
        aery_pm_enable_pll(pll0, true  /* divide by two */);

        aery_pm_wait_pll_to_lock(pll0);

        /* Set main clock source to PLL0 == 66 MHz */
        aery_pm_select_mck(PM_MCK_SOURCE_PLL0 /* master clock source */);

        /* Peripheral clock masking. By default all modules are enabled.
         * You might be interested in to disable modules you are not using. */
        pm->hsbmask = HSBMASK_DEFAULT;
        pm->pbamask = PBAMASK_DEFAULT;
        pm->pbbmask = PBBMASK_DEFAULT;

        while (!(pm->isr & AVR32_PM_ISR_MSKRDY_MASK));
            /* Clocks are now masked according to (CPU/HSB/PBA/PBB)_MASK
             * registers. */

    }

The default board initialization function can be found here. First it sets all GPIO pins to be inputs. Then it configures the board's power manager: starts the oscillator and clocks the master clock to 66 MHz. When you are changing the way how the board is initialized this is the place where to do it.



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

How to add new source files
'''''''''''''''''''''''''''

Let's say I would like to separate my source code into a ``src/newfile.c``. After creating the file I have to edit Makefile to get the ``newfile.c`` compiled. So, open Makefile into your editory and find the line::

    sources=main.c board.c

Edit this line so that it looks like this::

    sources=main.c board.c newfile.c

If you need to add a header file for the ``newfile.c``, put it under ``src/`` or ``include/`` directory. You don't have to edit the Makefile at this point anymore.



Library components
------------------

.. toctree::
  :maxdepth: 1

  aery32_gpio
  aery32_interrupts
  aery32_pm
  aery32_rtc
  aery32_spi



Examples
--------

Aery32 Framework comes with plenty of example programs, which **work out of box**. To test, for example, how USB can be used as a virtual COM port connect the board to USB and do the following:

**In Windows**

Open Command Prompt and command::

    cd myaery32-project
    cp examples\usbcdc.c src\main.c
    make programs

The quickest way to access Command Prompt is to press Windows-key and R (Win+R) at the same time, and type cmd.

**In Linux**

Open terminal and::

    cd myaery32-project
    cp examples/usbcdc.c src/main.c
    make programs

The following lines of commands overwrite the present ``main.c`` with the ``usbcdc.c`` example program and uploads\programs it into the development board. The program starts running immediately and writes *"Hello USB"* to COM port. Depending which COM port the board is connected, you can see the results by connecting to the port via terminal program. In Windows you can use `Putty <http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html>`_ for this.

.. note::

  The detailed instructions how to use the specific example program can be found from the example specific page, see the list below.

.. note::

  Every example program consists from a single file and can be found from ``examples/`` directory.



Where is my C++?
----------------

To use C++ you have to change the `avr32-gcc` compiler to `avr32-g++`. This can be done by editing the Makefile. Find the following line under `Standard user variables` section::

    CC=avr32-gcc

and replace it with::

    CC=avr32-g++

Also change this line below::

    CFLAGS=-std=gnu99 -Wall -O2 -mpart=$(mpart) \

to::

    CFLAGS=-std=gnu++98 -Wall -O2 -mpart=$(mpart) \

Or if you feel more experimental, you can chooce one of these: c++0x or gnu++0x.

Now you can use C++ in your project. Remember also to use the ``.hh`` header files of Aery32 instead of ``.h`` files. For example, instead of using

.. code-block:: c

    #include "aery32/gpio.h"

use

.. code-block:: c

    #include "aery32/gpio.hh"

At the moment Aery32 Software Framework enables only the namespaces in C++, so this is pretty much all C++ that comes in Aery32. Of course using C++ also allows checks toward enums, which are used with some functions.

.. code-block:: c
    :linenos:

    #include <stdbool.h>
    #include "aery32/gpio.hh"
    #include "board.h"

    #define LED AVR32_PIN_PC04

    using namespace aery;

    int main(void)
    {
        init_board();
        gpio_init_pin(LED, GPIO_OUTPUT|GPIO_HIGH); // yay! no aery_ prefix \o/

        for(;;) {
            /* Put your application code here */

        }

        return 0;
    }
