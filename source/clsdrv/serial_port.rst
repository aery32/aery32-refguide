Serial Port class driver, `#include <aery32/serial_port_clsdrv.h> <https://github.com/aery32/aery32/blob/master/aery32/aery32/serial_port_clsdrv.h>`_
========================

Serial Port class driver implements serial port communication using :doc:`USART
module functions <../functions/usart>` and :doc:`Peripheral Input/Output DMA
class drivers <periph_iodma>`. The driver can be used to communicate
with PC via COM port or with other integrated chips (ICs) which provide
RX and TX signal pins. Hardware handshaking (the use of RTS and CTS signal
pins), which requires DMA to work, is also supported by the class. `Skip to
example <https://github.com/aery32/aery32/blob/master/examples/serial_port_class_driver.cpp>`_.

Class instantiation
-------------------

To instantiate a Serial Port class driver you need to tell its constructor
which USART module you like to use. Additionally input and output
DMA buffers, *idma* and *odma*, are needed.

First allocate some space for the the DMA buffers:

.. code-block:: c++

    volatile uint8_t bufdma0[128] = {};
    volatile uint8_t bufdma1[128] = {};

After then instantiate Peripheral DMA class drivers by using the buffers
you just created:

.. code-block:: c++

    periph_idma dma0 = periph_idma(0, AVR32_PDCA_PID_USART0_RX, bufdma0, sizeof(bufdma0));
    periph_odma dma1 = periph_odma(1, AVR32_PDCA_PID_USART0_TX, bufdma1, sizeof(bufdma1));

The DMA pid value, which is the second parameter of the *periph_idma* and
*periph_odma* constructors, defines the USART data direction, so be sure to
select Peripheral DMA class' direction properly.

When DMAs are instatiated, instantiate the Serial Port class driver:

.. code-block:: c++

    serial_port pc = serial_port(usart0, dma0 /* input */, dma1 /* output */);
    pc.enable();

.. note::

    The object name ``pc`` was used here, because the connection is intended
    to be use with PC. See the :ref:`setting-up-terminal` below.

By default the speed is set to 115200 bit/s (baud error 0.16% with 66 MHz PBA
frequency). The default setting for parity is none. Stop and data bits are
1 and 8, respectively. All these settings can be changed with the class
member functions.

To change speed call

.. code-block:: c++

    pc.set_speed(speed); // speed in bit/s

Everytime you change the speed, the baud error rate is set to the
public ``error`` member and can be checked by calling ``pc.error``.

Parity and stop bits can be set like this:

.. code-block:: c++

    pc.set_parity(USART_PARITY_NONE);
    pc.set_stopbits(USART_STOPBITS_1);

The possible parity options are ``USART_PARITY_EVEN``, ``USART_PARITY_ODD``,
``USART_PARITY_MARKED`` and ``USART_PARITY_SPACE``. The number of stop bits can be
``USART_STOPBITS_1``, ``USART_STOPBITS_1p5`` or ``USART_STOPBITS_2``.

The Serial Port class driver supports several data bits values from 5 to 9.
Generally 8 data bits is used, but it can be changed with ``set_databits()``
member function:

.. code-block:: c++

    pc.set_databits(USART_DATABITS_5);

.. warning::

    Keep in mind that if 9 data bits is used, you also have to change
    the size of transfer of the used *periph_idma* and *periph_odma* class
    drivers, because 9 bits do not fit in one byte, which is the default
    DMA transfer size.

Hello World!
------------

When the Serial Port class driver is enabled it's ready to be used.
The well known "Hello World!" example would work like this:

.. code-block:: c++

    pc << "Hello Aery" << 32;

or like this:

.. code-block:: c++

    pc.printf("Hello Aery%d", 32);

A single character can be read and write like this:

.. code-block:: c++

    char c = pc.getc();
    pc.putc(c);

If you like to put the read character back to the read buffer call

.. code-block:: c++

    pc.putback(c);


Getline and line termination
----------------------------

You can read user input in lines like this:

.. code-block:: c++

    char line[32] = {};
    pc.getline(line);

``getline()`` will extract characters to *line* C string until either
the DMA input buffer is full or the delimiting character, which is ``\r\n``
by defaut, is found. Characters that precede the char (del), which is a
backspace (decimal value 127), are discarded from the line.

The total number of the read characters can be saved like this:

.. code-block:: c++

    size_t nread;
    pc.getline(line, *nread);

Delimitation character and ``\0`` aren't added to *nread*.

The default *delim* can be set by calling ``set_default_delim()`` member
function in this way:

.. code-block:: c++

    pc.set_default_delim('\n');
    pc.set_default_delim("\r\n");

Note that the delimitation character *delim* can be either a single character
or two sequential characters. If you need to use occasionally some other
delimitation character, define it as a third argument like this:

.. code-block:: c++

    pc.getline(line, &nread, '\n');

.. note::

    Be specific with the ``''`` and ``""`` notation. For example,
    ``set_default_delim("\n");``     would set the default line
    termination to ``\n\0`` instead of ``\n``.

.. hint::

    For input scanning, it's a good practice first fetch the line
    and then use ``sscanf()`` for that.

    .. code-block:: c++

        pc.getline(line);
        sscanf(line, "%d", &i);

.. hint::

    In main for loop you can skip empty lines in this way

    .. code-block:: c++

        for (;;) {
            pc.getline(line, &nread);
            if (nread == 0)
                continue; // skip

            // ...
        }


Flush and other supportive functions
------------------------------------

Sometimes you need to flush all bytes read into the input buffer. This
can be done with ``flush()`` member function. If you like to know
how many bytes have been received, call ``bytes_available()``.

It's also possible that the input buffer gets overflown. This can can
be checked by calling ``has_overflown()``. If the buffer has been
overflown, you can reset the serial port by calling ``reset()``.

Hardware handshaking
--------------------

To enable hardware handshaking just call ``pc.enable_hw_handshaking();``.
When the handshaking is enabled the receiver drives the RTS pin and the level
on the CTS pin modifies the behavior of the transmitter.

.. _setting-up-terminal:

Setting up the terminal software in PC side
-------------------------------------------

There are several free terminal emulator software which to use in Windows.
`PuTTY <http://en.wikipedia.org/wiki/PuTTY>`_ and
`Tera Term <http://en.wikipedia.org/wiki/Tera_Term>`_ are most known and
widely used.

If you choose to use PuTTY, select serial and set up the port (serial line)
and speed. Before saving the session go to the Terminal slide and enable
*Implicit LF in every CR*. Additionally force the local echo to see what you
type. If you want to use Linux type line termination, select *Implicit CR in
every LF* and use **CTRL+J** to send lines instead of pressing **ENTER**.

.. image:: ../../images/putty1.png
    :width: 8 cm
    :target: ../_images/putty1.png
    :alt: PuTTY select serial line and speed

.. image:: ../../images/putty2.png
    :width: 8 cm
    :target: ../_images/putty2.png
    :alt: PuTTY enable implicit LF in every CR

