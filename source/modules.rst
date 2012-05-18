Modules
=======

**Modules are library components that operate with the AVR32 internal peripherals.** Every module has its own namespace according to the module name. For example, Power Manager has module namespace of ``pm_``, Realtime Counter falls under the ``rtc_``, etc. To use the module component, just include its header file. These header files are also named after the module name. So, for example, to include and use functions that operate with the Power Manager include the file ``aery32/pm.h``.

.. hint::

    If you use C++, instead of .h file, include .hh.

General Periheral Input/Output (gpio)
-------------------------------------

To init any pin output high, there is a oneliner

.. code-block:: c

    aery_gpio_init_pin(AVR32_PIN_PC04, GPIO_OUTPUT|GPIO_HIGH);

The first argument is GPIO pin number and the second is for options. For 100 pin Atmel AVR32UC3 the GPIO pin number is a decimal number from 0 to 69. Fortunately, you do not have to remember which number represent what port and pin. Instead you can use predefined aliases as it was done above with the pin PC04 (5th pin in port C if the PC00 is the 1st).

The available options are

- GPIO_OUTPUT
- GPIO_INPUT
- GPIO_HIGH
- GPIO_LOW
- GPIO_FUNCTION_A
- GPIO_FUNCTION_B
- GPIO_FUNCTION_C
- GPIO_FUNCTION_D
- GPIO_INT_PIN_CHANGE
- GPIO_INT_RAISING_EDGE
- GPIO_INT_FALLING_EDGE
- GPIO_PULLUP
- GPIO_OPENDRAIN
- GPIO_GLITCH_FILTER
- GPIO_HIZ

These options can be combined with the pipe operator (boolean OR) to carry out several commands at once. Without this feature the above oneliner should be written with in lines of code

.. code-block:: c

        aery_gpio_init_pin(AVR32_PIN_PC04, GPIO_OUTPUT);
        aery_gpio_set_pin_high(AVR32_PIN_PC04);

Well now you also know how to set pin high, so you may guess that the following function sets it low

.. code-block:: c

    aery_gpio_set_pin_low(AVR32_PIN_PC04);

and that the following toggles it

.. code-block:: c

    aery_gpio_toggle_pin(AVR32_PIN_PC04);

and finally there is a read function

.. code-block:: c

    state = aery_gpio_read_pin(AVR32_PIN_PC04);

But before going any further, let's quickly go through those pin init options. ``FUNCTION_A``, ``B``, ``C`` and ``D`` assing the pin to the specific peripheral function, see datasheet pages 45--48. ``INT_PIN_CHANGE``, ``RAISING_EDGE`` and ``FALLING_EDGE`` enables interrupt events on that pin. Interrupts are trigged on pin change, at the rising edge or falling edge, respectively. ``GPIO_PULLUP`` connects pin to internal pull up resistor. ``GPIO_OPENDRAIN`` in turn makes the pin operate as an open drain mode. This mode is gererally used with pull up resistors to guarantee a high level on line when no driver is active. Lastly ``GPIO_GLITCH_FILTER`` activates the glitch filter and ``GPIO_HIZ`` makes pin high impedance.

.. note::

    Most of the combinations of GPIO init pin options do not make sense and have unknown consecuences.

Usually you want to init several pins at once -- not only one pin. This can be done for the pins that have the same port.

.. code-block:: c

    aery_gpio_init_pins(porta, 0xffffffff, GPIO_INPUT); // initializes all pins input

The first argument is a pointer to the port register and the second is pin mask. Aery32 GPIO module declares these ``porta``, ``b`` and ``c`` global pointers to the ports by default. Otherwise, you should have been more verbose and use ``&AVR32_GPIO.port[0]``, ``&AVR32_GPIO.port[1]`` and ``&AVR32_GPIO.port[2]``, respectively.

.. hint::

    As ``porta``, ``b`` and ``c`` are pointers to the GPIO port you can access its registers with arrow operator, for example, instead of using function ``aery_gpio_toggle_pin(AVR32_PIN_PC04)`` you can write ``portc->ovrt = (1 << 4);``

Local GPIO bus
''''''''''''''

AVR32 includes so called local bus interface that connects its CPU to device-specific high-speed systems, such as floating-point units and fast GPIO ports. To enable local bus

.. code-block:: c

    aery_gpio_enable_localbus();

When enabled you have to operate with `local` GPIO registers. The convinience functions described above does not work anymore. Aery32 GPIO module provides shortcuts to local bus by declaring ``lporta``, ``b`` and ``c`` global pointers. Use these to read and write local port registers. For example, to toggle pin through local bus you can write

.. code-block:: c

    lporta->ovrt = (1 << 4);

.. hint::

    Refer to datasheet pages 175--177 for GPIO Register Map.

.. note::

    CPU clock has to match with PBB clock to make local bus functional

To disable local bus and go back to normal

.. code-block:: c

    aery_gpio_disable_localbus();

Power Manager (pm)
------------------

Realtime Counter (rtc)
----------------------

Serial Periheral Bus (spi)
--------------------------
