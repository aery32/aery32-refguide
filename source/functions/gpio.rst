General Periheral Input/Output, ``#include <aery32/gpio.h>``
------------------------------------------------------------

To initialize any pin to be output high, there is a oneliner which can be used

.. code-block:: c++

    gpio_init_pin(AVR32_PIN_PC04, GPIO_OUTPUT|GPIO_HIGH);

The first argument is the GPIO pin number and the second one is for options. For 100 pin Atmel AVR32UC3, the GPIO pin number is a decimal number from 0 to 69. Fortunately, you do not have to remember which number represent what port and pin. Instead you can use predefined aliases as it was done above with the pin PC04 (5th pin in port C if the PC00 is the 1st).

The available pin init options are:

.. hlist::
    :columns: 3

    - ``GPIO_OUTPUT``
    - ``GPIO_INPUT``
    - ``GPIO_HIGH``
    - ``GPIO_LOW``
    - ``GPIO_FUNCTION_A``
    - ``GPIO_FUNCTION_B``
    - ``GPIO_FUNCTION_C``
    - ``GPIO_FUNCTION_D``
    - ``GPIO_INT_PIN_CHANGE``
    - ``GPIO_INT_RAISING_EDGE``
    - ``GPIO_INT_FALLING_EDGE``
    - ``GPIO_PULLUP``
    - ``GPIO_OPENDRAIN``
    - ``GPIO_GLITCH_FILTER``
    - ``GPIO_HIZ``

These options can be combined with the pipe operator (boolean OR) to carry out several commands at once. Without this feature the above oneliner should be written with two lines of code:

.. code-block:: c++

        gpio_init_pin(AVR32_PIN_PC04, GPIO_OUTPUT);
        gpio_set_pin_high(AVR32_PIN_PC04);

Well now you also know how to set pin high, so you may guess that the following function sets it low

.. code-block:: c++

    gpio_set_pin_low(AVR32_PIN_PC04);

and that the following toggles it

.. code-block:: c++

    gpio_toggle_pin(AVR32_PIN_PC04);

and finally it should not be surprise that there is a read function too

.. code-block:: c++

    state = gpio_read_pin(AVR32_PIN_PC04);

But before going any further, let's quickly go through those pin init options. ``FUNCTION_A``, ``B``, ``C`` and ``D`` assing the pin to the specific peripheral function, see datasheet pages 45--48. ``INT_PIN_CHANGE``, ``RAISING_EDGE`` and ``FALLING_EDGE`` enables interrupt events on the pin. Interrupts are trigged on pin change, at the rising edge or at falling edge, respectively. ``GPIO_PULLUP`` connects pin to the internal pull up resistor. ``GPIO_OPENDRAIN`` in turn makes the pin operate as an open drain mode. This mode is gererally used with pull up resistors to guarantee a high level on line when no driver is active. Lastly ``GPIO_GLITCH_FILTER`` activates the glitch filter and ``GPIO_HIZ`` makes the pin high impedance.

Usually you want to init several pins at once -- not only one pin. This can be done for the pins that have the same port.

.. code-block:: c++

    gpio_init_pins(porta, 0xffffffff, GPIO_INPUT); /* initializes all pins input */

The first argument is a pointer to the port register and the second one is the pin mask.

.. note::

    Most of the combinations of GPIO init pin options do not make sense and have unknown consecuences.

Local GPIO bus
--------------

AVR32 includes so called local bus interface that connects its CPU to device-specific high-speed systems, such as floating-point units and fast GPIO ports. To enable local bus call

.. code-block:: c++

    gpio_enable_localbus();

When enabled you have to operate with `local` GPIO registers. That is because, the convenience functions described above does not work local bus. To ease operating with local bus Aery32 GPIO module provides shortcuts to local ports by declaring ``lporta``, ``b`` and ``c`` global pointers. Use these to read and write local port registers. For example, to toggle pin through local bus you can write

.. code-block:: c++

    lporta->ovrt = (1 << 4);

.. note::

    CPU clock has to match with PBB clock to make local bus functional

To disable local bus and go back to normal operation call

.. code-block:: c++

    gpio_disable_localbus();