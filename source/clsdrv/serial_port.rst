Serial port class driver
========================

Serial port class driver implements serial port communication using USART.
The driver can be used to communicate with PC via COM port and with other
integrated chips (ICs) which provide RX and TX signal pins. Hardware
handshaking (the use of RTS and CTS signal pins) is also supported.

Header file:

`<aery32/serial_port_clsdrv.h> <https://github.com/aery32/aery32/blob/master/aery32/aery32/serial_port_clsdrv.h>`_

How to instantiate the class
----------------------------

To instantiate a serial port class we need to create input and output
buffers are needed, where the read and write characters. For these we
use Peripheral Input and Output DMA class drivers, so let's create those
first. So first allocate space for buffers. A buffer is an array, that
can be any size, for example 128 bytes as done below.

.. code-block:: c++

	volatile uint8_t bufdma0[128];
	volatile uint8_t bufdma1[128];

After then instantiate two DMA channels with these allocated buffer arrays.

.. code-block:: c++

	// Input DMA
	periph_idma dma0 = periph_idma(0, AVR32_PDCA_PID_USART0_RX, bufdma0, sizeof(bufdma0));

	// Output DMA
	periph_odma dma1 = periph_odma(1, AVR32_PDCA_PID_USART0_TX, bufdma1, sizeof(bufdma1));

The DMA pid value, which is the second parameter of the periph_i/odma constructor, defines
the USART data direction so be sure to select periph_idma or periph_odma properly. The pid
value also defines the USART module which to use. Here the USART0 has been choosed. We
need to remember this when instantiating a serial class driver that is done below

.. code-block:: c++

	serial_port pc = serial_port(usart0, dma0, dma1);
	pc.enable();

The first paramter is pointer to USART0 module register. The second param is
the reference to input DMA class driver and the third one to output DMA
class driver. When the serial port class driver is enabled it's ready to
be used. For example, the well known "Hello World!" example would go like
this

.. code-block:: c++

	pc << "Hello World!";


Setting speed, parity, etc.
---------------------------

By default the speed is set to 115200 bit/s. The default setting for parity,
stop and data bits are none, 1 and 8, respectively. All these settings can
be changed later.

To change speed call

.. code-block:: c++

	asd


Getline and line termination
----------------------------

asd
