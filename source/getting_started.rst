Getting Started
===============

`Aery32 development board <http://www.aery32.com>`_ and `framework
<https://github.com/aery32/aery32>`_ is a starting point for AVR32-based
projects. The framework provides a project structure and library
enabling **rapid prototyping and development**. Aery32 aims to be both
**professional** and **fun** that makes it ideal choise for R&D Engineers,
Academics and Hobbyists.

The best way to get started is to follow the `Quick Start <http://www.aery32.com/pages/quick-start>`_ from Aery32's homepage.

.. note::

    Aery32 development board and framework are copyright `Muiku Oy
    <http://www.muiku.com>`_. The framework, which this reference guide
    handles, is licensed under the new BSD license.

Installation
------------

You don't have to install Aery32 Software Framework as you would do with
regular software. Just `download the framework
<https://github.com/aery32/aery32/tags>`_, unzip it and start working with it.

.. note::

    Aery32 Framework is developed with Git distributed version control system
    at GitHub.

Requirements
------------

- Atmel AVR Toolchain
- Batchisp (for Windows) or DFU-programmer (for Mac OS X & Linux)

Basics of the embedded software
-------------------------------

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

Above you can see a basic embedded software coded by C++ programming language for Aery32. This piece of source code can be found from the ``main.cpp`` source file. The ``main()`` function at line 7 is the first function to execute when the program starts -- thus it is called *main*. The ``void`` keyword inside the brackets of the function, tells that the function does not take any arguments. The main function hardly ever takes arguments in embedded software, so this is a very common situation.

The ``int`` keyword, before the main function, indicates that the function will return integer type variable. Again, in the real life embedded software, it is very common that there is no use for the return value of main. The return type has been specified here to be integer type, instead of making it ``void``, only to keep the compiler happy. Otherwise the compiler would give a warning, which we do not want to see. For the sake of consistency the return value has been set zero at line 21, but the running application should never reach that far, or if it does, some serious error has occurred. Altough there is no use for the input arguments and the return value of the main function, the other functions, of course, may have arguments and can return values which are relevant.

Where to put the application code?
''''''''''''''''''''''''''''''''''

If the program should never reach the line 21, you might guess where the code of actual application is placed. Correct! It is placed inside of the infinite ``for(;;)`` loop. This loop goes on and on accomplish the code inside of it until the power is switched off. In this particular software there is only a comment line inside of the loop, so pretty much nothing is happening. What happens has been done at lines 10, 11 and 14. These functions will be executed only once, because those do not fall into the infinite for-loop. Furthermore, these function calls comes with the Aery32 Software Framework. The first one initializes the board, which is pretty much about starting the 12 MHz crystal oscillator and then setting up the main clock to 66 MHz. The second function call initializes a general purpose pin named ``LED`` that is the pin ``PC04`` to be exact as defined at line 5. The ``GPIO_OUTPUT`` part of the line states that the ``LED`` pin will be an output. Making the pin high at line 13 turns the LED on, so in conclusion the job of this software is only to keep the LED burning.
