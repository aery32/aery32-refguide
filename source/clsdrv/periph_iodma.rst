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
	periph_odma(int chnum, int pid,	volatile void *buf, size_t bufsize);

To instantiate input or output type DMA class driver one need to tell which
DMA channel number *chnum* and peripheral id *pid* will be used. The total
number of DMA channels is 15 (0-14). Any channel can be assigned to any
peripheral id. Be specific with the peripheral id and class driver type,
RX is input and TX is output. The possible pid values are

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

When *chnum* and *pid* have been chosen, a buffer and its size in bytes has to
be given as well. ``sizeof()`` can be then used to calculate the buffer size
in bytes, like this

.. code-block:: c++
	
	volatile uint16_t buf[128];
	periph_idma dma0 = periph_idma(0, AVR32_PDCA_PID_USART0_RX, buf, sizeof(buf));

Note that buffer has to be volatile type but the element size is not limited.
However, keep in mind that peripheral DMA can only work with 8, 16 or 32 bit
long size of transfers.

Size of transfer
----------------