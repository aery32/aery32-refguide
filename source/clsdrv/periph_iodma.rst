Peripheral Input/Output DMA, `#include <aery32/periph_iodma.h> <https://github.com/aery32/aery32/blob/master/aery32/aery32/periph_iodma_clsdrv.h>`_
===========================

AVR32 microcontrollers have PDCA module (Peripheral DMA Controller), which
allows direct memory access (DMA) between peripheral hardware and MCU's memory
independently of the central processing unit (CPU). This feature is useful
when the CPU cannot keep up with the rate of data transfer, or where the CPU
needs to perform useful work while waiting for a relatively slow I/O data
transfer.

Aery32 Framework implements two types of peripheral DMA drivers, input and output.
Input type driver can transfer data from a peripheral to memory whereas output
type driver transfers data from memory to a peripheral. For example, when using
USART peripheral to communicate with PC, input DMA writes data from PC to
MCU and output DMA from MCU to PC.

Class instantiation
-------------------

To instantiate input or output type DMA class driver you need to tell which
DMA channel number *chnum* and peripheral identifier *pid* will be used.
Additionally you have to give a pointer to the preallocated buffer and tell
its size **in bytes**. 

.. code-block:: c++

    volatile uint8_t ibuf[128] = {};
    volatile uint8_t obuf[128] = {};

    periph_idma input = periph_idma(0, ipid, ibuf, sizeof(ibuf));
    periph_odma output = periph_odma(1, opid, obuf, sizeof(obuf));

Instantiation does not enable the DMA by default, so you have to enable it
before the driver activates the DMA channel

.. code-block:: c++

    input.enable();
    output.enable();

The total number of DMA channels in UC3A type MCUs is 15 (0-14). Any channel can
be assigned to any peripheral id. Be specific with the peripheral id and class
driver type, RX is input and TX is output. The possible pid values are:

.. hlist::
    :columns: 3
        
    - ``AVR32_PDCA_PID_ADC_RX``
    - ``AVR32_PDCA_PID_ABDAC_TX``
    - ``AVR32_PDCA_PID_SPI0_RX``
    - ``AVR32_PDCA_PID_SPI0_TX``
    - ``AVR32_PDCA_PID_SPI1_RX``
    - ``AVR32_PDCA_PID_SPI1_TX``
    - ``AVR32_PDCA_PID_SSC_RX``
    - ``AVR32_PDCA_PID_SSC_TX``
    - ``AVR32_PDCA_PID_TWI_RX``
    - ``AVR32_PDCA_PID_TWI_TX``
    - ``AVR32_PDCA_PID_USART0_RX``
    - ``AVR32_PDCA_PID_USART0_TX``
    - ``AVR32_PDCA_PID_USART1_RX``
    - ``AVR32_PDCA_PID_USART1_TX``
    - ``AVR32_PDCA_PID_USART2_RX``
    - ``AVR32_PDCA_PID_USART2_TX``
    - ``AVR32_PDCA_PID_USART3_RX``
    - ``AVR32_PDCA_PID_USART3_TX``

.. note::

    The peripheral id cannot be assigned more than one channel.


Size of transfer
----------------

Peripheral DMA can work with 8, 16 or 32-bit wide size of transfers.
The size of transfer is set to 8-bit by default, but can be changed with the
``set_sizeof_transfer()`` member function. Either a byte, half-word or
word can be used (8-bit, 16-bit or 32-bit respectively). The example code
below shows how to use 32-bit size of transfer with the analog-to-digital
converter.

.. code-block:: c++

    volatile uint32_t buf[32] = {};
    periph_idma dma0 = periph_idma(0, AVR32_PDCA_PID_ADC_RX, buf, sizeof(buf));

    dma0.set_sizeof_transfer(PDCA_TRANSFER_SIZE_WORD);
    dma0.enable();

Reading the input DMA, ``periph_idma``
--------------------------------------

The read member function of the Peripheral Input DMA returns the total
number of elements moved from the DMA input buffer to a new destination
*dest*. If there was nothing to move zero is returned.

.. code-block:: c++

    uint8_t dest;
    if (input.read(&dest, 1))
        // one byte read
    else
        // there was nothing to read

To poll the input buffer whether there are bytes which to read call

.. code-block:: c++

    input.bytes_available();

If you suspect that the buffer has been overflown and thus needs to be reset
you can do it like this:

.. code-block:: c++

    if (input.has_overflown())
        input.reset();

In case you want to remove all bytes from the input buffer once and for all
call:

.. code-block:: c++

    input.flush();

.. note::

    With 32-bit size of transfer one read operation will increase the available
    bytes by 4, because one word (32-bit) is 4 * 8-bit. 16-bit size of transfer
    in turn would increase the available bytes by 2.

Writing to the output DMA, ``periph_odma``
------------------------------------------

The write member function of the Peripheral Output DMA fills the output
buffer, but does not start the transmission yet. To start the transmission
you have to call ``flush()``.

.. code-block:: c++

    output.write(dest, 1);
    output.flush();

After calling ``flush()`` you can follow the send process like this:

.. code-block:: c++

    while (output.bytes_in_progress())
        // still trasmitting

If you are unsure how many bytes you have written into the output buffer,
you can check it like this:

.. code-block:: c++

    if (output.bytes_in_buffer() == output.bufsize)
        output.flush();  // buffer is full, flush it