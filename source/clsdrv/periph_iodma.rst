Peripheral Input/Output DMA, `#include <aery32/periph_iodma.h> <https://github.com/aery32/aery32/blob/master/aery32/aery32/periph_iodma.h>`_
===========================

Direct memory access (DMA) allows peripheral hardware to access MCU's memory
independently of the central processing unit (CPU). This feature is useful
when the CPU cannot keep up with the rate of data transfer, or where the CPU
needs to perform useful work while waiting for a relatively slow I/O data
transfer. (`Wikipedia <http://en.wikipedia.org/wiki/Direct_memory_access>`_ 2013)

Class instantiation
-------------------

.. code-block:: c++

    periph_idma(int chnum, int pid, volatile void *buf, size_t bufsize);
    periph_odma(int chnum, int pid, volatile void *buf, size_t bufsize);

To instantiate input or output type DMA class driver one need to tell which
DMA channel number *chnum* and peripheral id *pid* will be used. The total
number of DMA channels is 15 (0-14). Any channel can be assigned to any
peripheral id. Be specific with the peripheral id and class driver type,
RX is input and TX is output. The possible pid values are

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

When *chnum* and *pid* have been chosen, a buffer and its size **in bytes**
has to be given as well. One can use standard function of ``sizeof()`` to
calculate the buffer size in bytes, like this

.. code-block:: c++

    volatile uint8_t buf[128] = {};
    periph_idma dma0 = periph_idma(0, AVR32_PDCA_PID_USART0_RX, buf, sizeof(buf));

Note that the buffer has to be volatile type but the element size is not
limited. However, keep in mind that peripheral DMA can only work with 8, 16
or 32-bit wide size of transfers.

Lastly enable the DMA class driver

.. code-block:: c++

    dma0.enable();

Size of transfer
----------------

The size of transfer is set to 8-bit by default, but can be changed with the
``set_sizeof_transfer()`` member function. Either a byte, half-word or
word can be used (8-bit, 16-bit or 32-bit respectively). The example code
below shows how to use the size of transfer of 32-bit with Analog-to-Digital
converter.

.. code-block:: c++

    volatile uint32_t buf[32] = {};
    periph_idma dma0 = periph_idma(0, AVR32_PDCA_PID_ADC_RX, buf, sizeof(buf));
    dma0.set_sizeof_transfer(PDCA_TRANSFER_SIZE_WORD).enable();

Reading the input DMA
---------------------

.. code-block:: c++

    size_t read(uint8_t *dest, size_t n);
    size_t read(uint16_t *dest, size_t n);
    size_t read(uint32_t *dest, size_t n);

The read function of the Peripheral Input DMA returns the total number of
elements moved from the DMA input buffer to new destination *dest*. If there
was nothing to move, zero is returned despite the size of *n*. To poll the
input buffer whether there are bytes which to read call ``bytes_available()``.
``has_overflown()`` in turn tells if the buffer has been overflown.

In case you want to remove all bytes from the input buffer once and all call
``flush()``.

.. note::

    With 32-bit size of transfer one read operation will increase the available
    bytes by 4, because one word (32-bit) is 4 * 8-bit. 16-bit size of transfer
    in turn would increase the available bytes by 2.

Writing to the output DMA
-------------------------

.. code-block:: c++

    periph_odma& write(uint8_t *dest, size_t n);
    periph_odma& write(uint16_t *dest, size_t n);
    periph_odma& write(uint32_t *dest, size_t n);

The write function of the Peripheral Output DMA fills the output buffer, but
does not start the transmission yet. To start the transmission call
``flush()``. After then you can use ``bytes_in_progress()`` to follow the send
process. If you are unsure how many bytes you have written in the buffer call
``bytes_in_buffer()``.