Example programs, ``examples/``
===============================

Aery32 Framework comes with plenty of example programs, which are placed
under the ``examples/`` directory. Every file is an independent program
that does not need other files to work. So it should work out of box if you
just replace the default ``main.cpp`` with the example.

.. note ::

    You may remove the ``examples/`` directory from your project if you don't
    need it.

For example, let's see how the LED toggling demo looks like

.. code-block:: c++
    :linenos:

    #include "board.h"
    using namespace aery;

    int main(void)
    {
        board::init();
        for(;;) {
            gpio_toggle_pin(LED);
            delay_ms(500);
        }

        return 0;
    }

To test this example open the Command Prompt and command::

    cp examples\toggle_led.cpp main.cpp
    make programs

The following lines of commands overwrite the present ``main.cpp`` file
from the project root with the example, compiles the project and uploads
the new binary to the development board. The program starts running
immediately and you should see how the LED pin blinks every half a second.

.. note ::

    The quickest way to access Command Prompt in Windows is to press
    Windows-key and R (Win+R) at the same time, and type cmd.

