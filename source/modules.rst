Modules
=======

**Modules are library components that operate with the AVR32 internal peripherals.** Every module has its own namespace according to the module name. For example, Power Manager has module namespace of ``pm_``, Realtime Counter falls under the ``rtc_`` namespace, etc. To use the module just include its header file. These header files are also named after the module name. So, for example, to include and use functions that operate with the Power Manager include the file `aery32/pm.h`, ``#include "aery32/pm.h"``.

.. hint::

    If you use C++, instead of .h file, include .hh.

General Periheral Input/Output (gpio)
-------------------------------------

To initialize any pin to output high, there is a oneliner which can be used

.. code-block:: c

    aery_gpio_init_pin(AVR32_PIN_PC04, GPIO_OUTPUT|GPIO_HIGH);

The first argument is the GPIO pin number and the second one is for options. For 100 pin Atmel AVR32UC3, the GPIO pin number is a decimal number from 0 to 69. Fortunately, you do not have to remember which number represent what port and pin. Instead you can use predefined aliases as it was done above with the pin PC04 (5th pin in port C if the PC00 is the 1st).

The available pin init options are:

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

These options can be combined with the pipe operator (boolean OR) to carry out several commands at once. Without this feature the above oneliner should be written with two lines of code:

.. code-block:: c

        aery_gpio_init_pin(AVR32_PIN_PC04, GPIO_OUTPUT);
        aery_gpio_set_pin_high(AVR32_PIN_PC04);

Well now you also know how to set pin high, so you may guess that the following function sets it low

.. code-block:: c

    aery_gpio_set_pin_low(AVR32_PIN_PC04);

and that the following toggles it

.. code-block:: c

    aery_gpio_toggle_pin(AVR32_PIN_PC04);

and finally it should not be surprise that there is a read function too

.. code-block:: c

    state = aery_gpio_read_pin(AVR32_PIN_PC04);

But before going any further, let's quickly go through those pin init options. ``FUNCTION_A``, ``B``, ``C`` and ``D`` assing the pin to the specific peripheral function, see datasheet pages 45--48. ``INT_PIN_CHANGE``, ``RAISING_EDGE`` and ``FALLING_EDGE`` enables interrupt events on the pin. Interrupts are trigged on pin change, at the rising edge or at falling edge, respectively. ``GPIO_PULLUP`` connects pin to the internal pull up resistor. ``GPIO_OPENDRAIN`` in turn makes the pin operate as an open drain mode. This mode is gererally used with pull up resistors to guarantee a high level on line when no driver is active. Lastly ``GPIO_GLITCH_FILTER`` activates the glitch filter and ``GPIO_HIZ`` makes the pin high impedance.

.. note::

    Most of the combinations of GPIO init pin options do not make sense and have unknown consecuences.

Usually you want to init several pins at once -- not only one pin. This can be done for the pins that have the same port.

.. code-block:: c

    aery_gpio_init_pins(porta, 0xffffffff, GPIO_INPUT); // initializes all pins input

The first argument is a pointer to the port register and the second one is the pin mask. Aery32 GPIO module declares these ``porta``, ``b`` and ``c`` global pointers to the ports by default. Otherwise, you should have been more verbose and use ``&AVR32_GPIO.port[0]``, ``&AVR32_GPIO.port[1]`` and ``&AVR32_GPIO.port[2]``, respectively.

.. hint::

    As ``porta``, ``b`` and ``c`` are pointers to the GPIO port, you can access its registers with arrow operator, for example, instead of using function ``aery_gpio_toggle_pin(AVR32_PIN_PC04)`` you could write ``portc->ovrt = (1 << 4);`` Refer to datasheet pages 175--177 for GPIO Register Map.

Local GPIO bus
''''''''''''''

AVR32 includes so called local bus interface that connects its CPU to device-specific high-speed systems, such as floating-point units and fast GPIO ports. To enable local bus call

.. code-block:: c

    aery_gpio_enable_localbus();

When enabled you have to operate with `local` GPIO registers. That is because, the convenience functions described above does not work local bus. To ease operating with local bus Aery32 GPIO module provides shortcuts to local ports by declaring ``lporta``, ``b`` and ``c`` global pointers. Use these to read and write local port registers. For example, to toggle pin through local bus you can write

.. code-block:: c

    lporta->ovrt = (1 << 4);

.. note::

    CPU clock has to match with PBB clock to make local bus functional

To disable local bus and go back to normal operation call

.. code-block:: c

    aery_gpio_disable_localbus();

Power Manager (pm)
------------------

Realtime Counter (rtc)
----------------------

Serial Periheral Bus (spi)
--------------------------

AVR32 UC3A1 includes to separate SPI buses, SPI0 and SPI1. To initialize SPI bus it is good practice to define pin mask for SPI related pins. Refering to datasheet page 45, SPI0 operates from PORTA:

- PA07 => NPCS3
- PA08 => NPCS1
- PA09 => NPCS2
- PA10 => NPCS0
- PA11 => MISO 
- PA12 => MOSI 
- PA13 => SCK

So let's define the pin mask for SPI0 with NPCS0 (Numeric Processor Chip Select, also known as slave select):

.. code-block:: c

    #define SPI0_GPIO_MASK ((1 << 10) | (1 << 11) | (1 << 12) | (1 << 13))

Next we have to initialize these pins to the right peripheral function that is FUNCTION A. To do that use pin initializer from gpio module:

.. code-block:: c

    aery_gpio_init_pins(porta, SPI0_GPIO_MASK, GPIO_FUNCTION_A);

Now the GPIO pins have been assigned appropriately and we are ready to initialize SPI0. Let's init it as a master:

.. code-block:: c

    aery_spi_init_master(spi0);

The only parameter is a pointer to the SPI register. Aery32 declares ``spi0`` and ``spi1`` global pointers by default. After this we have to setup chip select line zero (NPCS0) with the desired spi mode and shift register width

.. code-block:: c

    aery_spi_setup_npcs(spi0, 0, SPI_MODE0, 16);

The shift register width 16 is the maximum, but you can still use arbitrary wide transmission (described later). The minum value for this is 8 bits.

.. hint::

    Chip select baudrate is hard coded to MCK/255. To make it faster you can bitbang the SCRB bit in the CSRX register, where X is the NPCS number:

    .. code-block:: c

         aery_spi_setup_npcs(spi0, 0, SPI_MODE0, 16);
         spi0->CSR0.scbr = 32; // baudrate is now MCK/32

Now we are ready to enable spi peripheral

.. code-block:: c

    aery_spi_enable(spi0);

There's also function for disabling ``aery_spi_disable(spi0)``. To write data into SPI bus use the transmit function

.. code-block:: c

    uint16_t rd;
    rd = aery_spi_transmit(spi0, 0x55, 0, true); // writes 0x55 to SPI0, NPCS0

.. hint::
    
    ``aery_spi_transmit()`` writes and reads SPI bus simultaneusly. If you only want to read data, just ignore write data by sending dummy bits.

Here is the above SPI initialization and transmission in complete example code:

.. code-block:: c
    :linenos:

    #include <stdbool.h>
    #include "aery32/gpio.h"
    #include "aery32/spi.h"
    #include "board.h"

    #define SPI0_GPIO_MASK ((1 << 10) | (1 << 11) | (1 << 12) | (1 << 13))

    int main(void)
    {
        uint16_t rd; // received data

        init_board();

        aery_gpio_init_pins(porta, SPI0_GPIO_MASK, GPIO_FUNCTION_A);
        aery_spi_init_master(spi0);
        aery_spi_setup_npcs(spi0, 0, SPI_MODE0, 16);
        aery_spi_enable(spi0);

        for (;;) {
            rd = aery_spi_transmit(spi0, 0x55, 0, true); // writes 0x55 to SPI0, NPCS0
        }

        return 0;
    }

Sending arbitrary wide spi data
'''''''''''''''''''''''''''''''

The last parameter of ``aery_spi_transmit()`` function indicates for the spi peripheral whether the current transmission is the last on. If true, chip select line rises immediately when the last bit has been written. If it is defined false, CS ine is left low for the next chunk of the transmission. This feature allows to operate with spi buses with arbitrary wide shift registers. For example, to read and write 32 bit wide spi data you can do this:

.. code-block:: c

    uint32_t rd;
    
    aery_spi_setup_npcs(spi0, 0, SPI_MODE0, 8);

    rd = aery_spi_transmit(spi0, 0x55, 0, false);
    rd |= aery_spi_transmit(spi0, 0xf0, 0, false) << 8;
    rd |= aery_spi_transmit(spi0, 0x0f, 0, true) << 16; // complete