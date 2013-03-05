Serial Port class driver
========================

#include `<aery32/serial_port_clsdrv.h> <https://github.com/aery32/aery32/blob/master/aery32/aery32/serial_port_clsdrv.h>`_

Serial Port class driver implements serial port communication using USART.
The driver can be used to communicate with PC via COM port and with other
integrated chips (ICs) which provide RX and TX signal pins. Hardware
handshaking (the use of RTS and CTS signal pins) is also supported.

Class instantiation
-------------------

.. code-block:: c++

    serial_port(volatile avr32_usart_t *usart, aery::periph_idma &idma, aery::periph_odma &odma);

To instantiate a Serial Port class driver we need to tell its constructor
method which USART module we like to use by giving a pointer to the USART
module register. Additionally input and output buffers, *idma* and *odma*,
are needed.

Let's first allocate some space for the needed buffers.

.. code-block:: c++

    volatile uint8_t bufdma0[128];
    volatile uint8_t bufdma1[128];

After then let's instantiate Peripheral DMA class drivers using the
buffers we just allocated.

.. code-block:: c++

    periph_idma dma0 = periph_idma(0, AVR32_PDCA_PID_USART0_RX, bufdma0, sizeof(bufdma0));
    periph_odma dma1 = periph_odma(1, AVR32_PDCA_PID_USART0_TX, bufdma1, sizeof(bufdma1));

The DMA pid value, which is the second parameter of the *periph_i/odma*
constructor, defines the USART data direction, so be sure to select
*periph_idma* and *periph_odma* classes properly.

Now the end is as easy as calling

.. code-block:: c++

    serial_port pc = serial_port(usart0, dma0, dma1);
    pc.enable();

.. note::

    The object name ``pc`` was used here, because of the example where
    the connection is intended to be use with PC. See the *Setting
    up the terminal software in PC side* below.


Hello World!
------------

When the Serial Port class driver is instantiated and enabled it's ready
to be used. The well known "Hello World!" example would work like this

.. code-block:: c++

    pc << "Hello Aery" << 32;

or like this

.. code-block:: c++

    pc.printf("Hello Aery%d", 32);

Setting speed, parity, stop bits etc.
-------------------------------------

By default the speed is set to 115200 bit/s. The default setting for parity
is none. Stop and data bits are 1 and 8, respectively. All these settings can
be changed via class member functions.

To change speed call

.. code-block:: c++

    pc.set_speed(speed);

Parity and stop bits can be set like this

.. code-block:: c++

    pc.set_parity(USART_PARITY_NONE);
    pc.set_stopbits(USART_STOPBITS_1);

The parity options:

.. hlist::
    :columns: 2

    - ``USART_PARITY_EVEN``
    - ``USART_PARITY_ODD``
    - ``USART_PARITY_MARKED``
    - ``USART_PARITY_SPACE``

Stop bits options:

.. hlist::
    :columns: 3

    - ``USART_STOPBITS_1``
    - ``USART_STOPBITS_1p5``
    - ``USART_STOPBITS_2``

To enable hardware handshaking just call

.. code-block:: c++

    pc.enable_hw_handshaking();

Getline and line termination
----------------------------

.. code-block:: c++

    char* getline(char *str, size_t *nread, char delim);
    char* getline(char *str, size_t *nread, const char *delim);

The upper two member functions can be used to get a user input as lines.
This means that characters are extracted to *str* (C string) until either
the DMA input buffer is full or the delimiting character is found.
The delimitation character *delim* can be either single character or two characters.
*nread* is the total number of characters read. Delimitation character and ``\0`` aren't
added to this value.

The following code would wait user input until the delimation character
``\n`` has been found.

.. code-block:: c++

    size_t nread = 0;
    char line[32] = "";

    pc.getline(line, &nread, '\n');

You can also omit the last two params (*nread* and *delim*). When *delim* has been
omitted the default setting ``\r\n`` is used. You can change this default setting by calling
``set_default_delim()`` member function as shown below.

.. code-block:: c++

    pc.set_default_delim('\n');
    pc.set_default_delim("\r\n");

.. note::

    Be specific with the ``''`` and ``""`` notation. For example, ``set_default_delim("\n");``
    would set the default line termination to ``\n\0`` instead of ``\n`` that you
    might have expected.

.. note::

    For input scanning it's a good practice first fetch the line and then use ``sscanf()``
    for that.

    .. code-block:: c++

        pc.getline(line);
        sscanf(line, "%d", &i);

.. hint::

    In main for loop you can skip empty lines this way

    .. code-block:: c++

        for (;;) {
            pc.getline(line, &nread);
            if (nread == 0) continue;

            // else do something
        }


Setting up the terminal software in PC side
-------------------------------------------
