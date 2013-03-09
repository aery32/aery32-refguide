Universal Sync/Asynchronous Receiver/Transmitter, ``#include <aery32/usart.h>``
===============================================================================

.. note::

    There's also a class driver for serial port communication. Skip to
    :doc:`Serial port class driver <../clsdrv/serial_port>`.

USART module can be used for several kind of communication. The RS-232 serial
communication with a personal computer (PC) is likely the most common one.

Initialization
--------------

To initialize the USART0 for serial communication call

.. code-block:: c++

    usart_init_serial(usart0, USART_PARITY_NONE, USART_STOPBITS_1);

Serial communication initialized with no parity and 1 stop bits is the
default setup for most of the devices, so you may omit the last two
parameters if you like. Other options are ``EVEN``, ``ODD``, ``MARKED`` and
``SPACE`` for parity, and ``1p5`` and ``2`` for stop bits.

After then you have to set up the baud rate. The baud rate is derived from
clock of the peripheral bus B (PBA) or at external pin. This source clock
is then diveded in divider for integer part and additionally for fractional
part to achieve even smaller baudrate error. Assuming that the PBA bus
speed is 66 MHz we have to divide it by 71 to get the baud rate of 115 200
bit/s (error 0.8%).

.. code-block:: c++
    
    usart_setup_speed(usart0, USART_CLK_PBA, 71, 0);

The last parameter is the divider for the fractional part which could have
been also omitted because it was set to zero. We could have set the 
fractional part here to 5 to get even smaller baud rate error (0.015%).

At last you have to enable RX and TX channels separately for receiving
and transmitting data

.. code-block:: c++
    
    usart_enable_rx(usart0);
    usart_enable_rx(usart0);