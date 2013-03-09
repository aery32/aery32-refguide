Two-wire (I2C) Interface Bus
============================

#include `<aery32/twi.h> <https://github.com/aery32/aery32/blob/master/aery32/aery32/twi.h>`_

TWI interface is initialized as a master like this

.. code-block:: c++

    twi_init_master();

By default the initializer sets the bus' SCL frequency to 100 kHz. The maximum supported frequency is 400 kHz. This can be set up with ``twi_setup_clkwaveform()`` after you have called the init function. For example, to set SCL to 400 kHz with 50% dutycycle, call

.. code-block:: c++

    twi_setup_clkwaveform(1, 0x3f, 0x3f);

The first parameter is the clock divider. The second and third one define the dividers for the clock low and high states, respectively. Altering these divider values for clock high and low states you can modify the SCL waveform. However, you most likely want to set those equal to get 50% dutycycle for the clock.

.. note::

    TWI module does not have an enable function as other modules. The module is enabled when TWI pins are initialized with GPIO module:

    .. code-block:: c++

        #define TWI_PINS ((1 << 29) | (1 << 30))
        gpio_init_pins(porta, TWI_PINS, GPIO_FUNCTION_A | GPIO_OPENDRAIN);

.. warning::

    Important! Don't forget to connect the appropriate size external pull-up resistors to SDA and SCL pins. Try for example 4k7 value resistors. The exact optimal value depends on the SCL and the parasitic capacitance of the bus.

Read and write operations
-------------------------

The read and write operations for a single byte works like this

.. code-block:: c++

    uint8_t rd;

    twi_write_byte(0x04);
    twi_read_byte(&rd);

Both functions return the number of successfully written or read bytes. So for one byte read/write operation the return value would be 0 on error and 1 on success.

To read and write multiple bytes use ``twi_read/write_nbytes()``, like this

.. code-block:: c++

    uint8_t wd[3] = { 0x02, 0x04, 0x06 };
    uint8_t rd[3];

    twi_write_nbytes(wd, 3);
    twi_read_nbytes(rd, 3);

Using internal device address
-----------------------------

Both read and write functions can take an optional internal device address in their last param. Internal device address is the slave's internal register where data is written or read from. For example, the following snippet writes a byte ``0x04`` to the slave register (or in other words slave's internal address) ``0x80``.

.. code-block:: c++

    uint8_t byte = 0x04;
    uint8_t iadr = 0x80;

    twi_write_byte(byte, iadr);

When optional address has been given the same address is used in every read and write operations that follows the previous operation even if the address is omitted from the function call. To clear this behaviour, call ``twi_clear_internal_address()``.

If you want to use a wider than 8-bit internal device addresses, you have to indicate the address lenght via additional third parameter. For example to use 2 bytes long address, you may call the write function like this

.. code-block:: c++
    
    uint8_t byte = 0x04;
    uint16_t iadr = 0x8080;

    twi_write_byte(byte, iadr, 2);

The largest supported internal device address length is three bytes long.