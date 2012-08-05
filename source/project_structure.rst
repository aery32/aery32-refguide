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

It is intended that you work under the root directory most of the time as that is the place where you keep adding your ``.c``, ``.cpp`` and ``.h`` source files. Notice that Aery32 Framework is a C/C++ framework and thus you can write .c and .cpp files.

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

.. code-block:: c++
    :linenos:

    #ifndef __BOARD_H
    #define __BOARD_H

    extern "C" {
        #include <inttypes.h>
    }

    // ----------------------------------------------------------------------
    // Board specific settings
    // ----------------------------------------------------------------------
    #define F_CPU 66000000UL

    #define LED AVR32_PIN_PC04

    #define ADC_VREF 3.0
    #define ADC_BITS 10

    #define HSBMASK_DEFAULT 0xFFFFFFFF
    #define PBAMASK_DEFAULT 0xFFFFFFFF
    #define PBBMASK_DEFAULT 0xFFFFFFFF


    // ----------------------------------------------------------------------
    // Board functions
    // ----------------------------------------------------------------------
    void init_board(void);

    static inline double cnv2volt(uint32_t cnv)
    {
        return cnv * (ADC_VREF / (1UL << ADC_BITS));
    }

    #endif


**board.cpp**

The default board initialization function, ``init_board()``, can be found here. First it sets all the GPIO pins to inputs. Then it configures the board's power manager. Basicly the external oscillator ``OCS0`` is started and the master clock frequency is set to 66 MHz. If you like to change the master clock frequency or want to change the way how the board is initialized, this is the place where to do it.

.. code-block:: c++
    :linenos:

    #include "board.h"
    #include <aery32/pm.h>
    #include <aery32/gpio.h>
    #include <aery32/flashc.h>

    using namespace aery;

    void init_board(void)
    {
        gpio_init_pins(porta, 0xffffffff, GPIO_INPUT);
        gpio_init_pins(portb, 0xffffffff, GPIO_INPUT);
        gpio_init_pins(portc, 0x0000003f, GPIO_INPUT);

        pm_start_osc(0, OSC_MODE_GAIN3, OSC_STARTUP_36ms);
        pm_wait_osc_to_stabilize(0);

        pm_init_pllvco(pll0, PLL_SOURCE_OSC0, 11, 1, false); // VCO0 = 132 MHz
        pm_enable_pll(pll0, true); // PLL0 = 66 MHz
        pm_wait_pll_to_lock(pll0);

        pm_init_pllvco(pll1, PLL_SOURCE_OSC0, 16, 1, true); // VCO1 = 192 MHz
        pm_enable_pll(pll1, true); // PLL1 = 96 MHz
        pm_wait_pll_to_lock(pll1);

        flashc_init(FLASH_1WS, true); // One wait state for flash
        pm_select_mck(MCK_SOURCE_PLL0); // Main clock speed is now 66 MHz

        /*
         * Peripheral clock masking. By default all modules are enabled.
         * You might be interested in to disable modules you are not using. */
        pm->hsbmask = HSBMASK_DEFAULT;
        pm->pbamask = PBAMASK_DEFAULT;
        pm->pbbmask = PBBMASK_DEFAULT;

        while (!(pm->isr & AVR32_PM_ISR_MSKRDY_MASK));
            /*
             * Clocks are now masked according to (CPU/HSB/PBA/PBB)_MASK
             * registers.
             */

    }

**aery32/**

This directory contains the source files of Aery32 library. The archive of the library (.a file) appears in this directory after the first compile process. The ``aery32/`` subdirectory contains the header files of the library. Linker scripts, which are essential files to define the MCU memory structure are placed under the ``ldscripts/`` directory. However, you should not need to hassle with those files.

**examples/**

All the example programs are placed under this directory. Every program is completely independent. Read more below.

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
