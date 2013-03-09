Serial Peripheral Bus
=====================

#include `<aery32/spi.h> <https://github.com/aery32/aery32/blob/master/aery32/aery32/spi.h>`_

AVR32 UC3A1 includes to separate SPI buses, SPI0 and SPI1. To initialize SPI bus it is good practice to define pin mask for the SPI related pins. Refering to datasheet page 45, SPI0 operates from PORTA:

- PA07, NPCS3
- PA08, NPCS1
- PA09, NPCS2
- PA10, NPCS0
- PA11, MISO 
- PA12, MOSI 
- PA13, SCK

So let's define the pin mask for SPI0 with NPCS0 (Numeric Processor Chip Select, also known as slave select or chip select):

.. code-block:: c++

    #define SPI0_GPIO_MASK ((1 << 10) | (1 << 11) | (1 << 12) | (1 << 13))

Next we have to assing these pins to the right peripheral function that is FUNCTION A. To do that use pin initializer from GPIO module:

.. code-block:: c++

    gpio_init_pins(porta, SPI0_GPIO_MASK, GPIO_FUNCTION_A);

Now the GPIO pins have been assigned appropriately and we are ready to initialize SPI0. Let's init it as a master:

.. code-block:: c++

    spi_init_master(spi0);

The only parameter is a pointer to the SPI register. Aery32 declares ``spi0`` and ``spi1`` global pointers by default.

.. hint::

    If the four SPI CS pins are not enough, you can use CS pins in multiplexed mode (of course you need an external multiplexer circuit then) and expand number of CS lines to 16. This can be done by bitbanging PCSDEC bit in SPI MR register after the initialization:

    .. code-block:: c++
 
        spi_init_master(spi0);
        spi0->MR.pcsdec = 1;

When the SPI peripheral has been initialized as a master, we still have to setup its CS line 0 (NPCS0) with the desired SPI mode and shift register width. To set these to SPI mode 0 and 16 bit, call the npcs setup function with the following parameters

.. code-block:: c++

    spi_setup_npcs(spi0, 0, SPI_MODE0, 16);

The minimum and maximum shift register widths are 8 and 16 bits, respectively, but you can still :ref:`use arbitrary wide transmission <sending-arbitrary-wide-spi-data>`.

.. hint::

    Chip select baudrate is hard coded to MCK/255. To make it faster you can bitbang the SCRB bit in the CSRX register, where X is the NPCS number:

    .. code-block:: c++

         spi_setup_npcs(spi0, 0, SPI_MODE0, 16);
         spi0->CSR0.scbr = 32; /* baudrate is now MCK/32 */

.. hint::

    Different CS lines can have separate SPI mode, baudrate and shift register width.

Now we are ready to enable SPI peripheral

.. code-block:: c++

    spi_enable(spi0);

There's also function for disabling the desired SPI peripheral, ``spi_disable()``.

To read and write data use the SPI transmit function

.. code-block:: c++

    uint16_t rd;
    rd = spi_transmit(spi0, 0, 0x55);

The above call to ``spi_transmit`` writes 0x55 to SPI0 using NPCS0 slave (or chip) select pin. Notice that ``spi_transmit()`` writes and reads the SPI bus simultaneusly. If you only want to read data, just ignore write data by sending dummy bits.

Here is the complete code for the above SPI initialization and transmission:

.. code-block:: c++
    :linenos:

    #include "board.h"
    #include <aery32/gpio.h>
    #include <aery32/spi.h>

    using namespace aery;

    #define SPI0_GPIO_MASK ((1 << 10) | (1 << 11) | (1 << 12) | (1 << 13))

    int main(void)
    {
        uint16_t rd; /* received data */

        init_board();

        gpio_init_pins(porta, SPI0_GPIO_MASK, GPIO_FUNCTION_A);
        spi_init_master(spi0);
        spi_setup_npcs(spi0, 0, SPI_MODE0, 16);
        spi_enable(spi0);

        for (;;) {
            rd = spi_transmit(spi0, 0, 0x55);
        }

        return 0;
    }

.. _sending-arbitrary-wide-spi-data:

Sending arbitrary wide SPI data
-------------------------------

``spi_transmit()`` supports arbitrary wide SPI transmits through its optional last parameter named as ``islast``. This param indicates if the chip select line should be left low or can be pulled high. Otherway around it tells whether the current transmission was the last one or not. Thus the param is called ``islast``. By default ``islast`` is set true and the CS line rises immediately when the last bit has been written. If ``islast`` is defined false, CS line is left low for the next transmission that should occur immediately after the previous one. This feature allows SPI to operate with arbitrary wide shift registers. For example, to read and write 24 bit wide SPI data you can do this:

.. code-block:: c++

    uint32_t rd;
    
    spi_setup_npcs(spi0, 0, SPI_MODE0, 8);

    rd = spi_transmit(spi0, 0, 0x55, false);
    rd |= spi_transmit(spi0, 0, 0xf0, false) << 8;
    rd |= spi_transmit(spi0, 0, 0x0f, true) << 16; /* Complete. Asserts the chip select */