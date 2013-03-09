Analog-to-Digital Converter, `#include <aery32/adc.h> <https://github.com/aery32/aery32/blob/master/aery32/aery32/adc.h>`_
===========================

AVR32 UC3A0/1 microcontrollers have eight 10-bit analog-to-digital converters.
The maximum ADC clock frequency for the 10-bit precision is 5 MHz and for
8-bit precision it's 8 MHz. This sampling frequency is related to the
frequency of the Peripheral Bus A (PBA), so take care when setting ADC
clock prescaler values. `Skip to example <https://github.com/aery32/aery32/blob/master/examples/adc.cpp>`_.

Initialization
--------------

.. code-block:: c++

    adc_init(
        7,    /* prescal, adclk = pba_clk / (2 * (prescal+1)) */
        true, /* hires, 10-bit (false would be 8-bit) */
        0,    /* shtim, sample and hold time = (shtim + 1) / adclk */
        0     /* startup, startup time = (startup + 1) * 8 / adclk */
    );

The initialization statement given above, uses the prescaler value 7, so if the PBA clock was 66 MHz, the ADC clock would be 4.125 MHz. After initialization, you have to enable the channels that you like to use for the conversion. For example, to enable channel three call

.. code-block:: c++

    adc_enable(1 << 3);

Reading the conversion
----------------------

Now you can start the conversion. Be sure to wait that the conversion is ready before reading the conversion value.

.. code-block:: c++

    uint16_t result;

    adc_start_cnv();
    while (adc_isbusy(1 << 3));
    result = adc_read_cnv(3);

If you only want to read the latest conversion, whatever was the channel, you can omit the channel mask for the busy function and read the conversion with another function like this

.. code-block:: c++

    while (adc_isbusy());
    result = adc_read_lastcnv();

ADC hardware triggers
---------------------

To setup the ADC hardware trigger, call ``adc_setup_trigger()`` after init

.. code-block:: c++

    adc_setup_trigger(EXTERNAL_TRG);

Other possible trigger sources, that can be used for example with the Timer/Counter module, are

.. hlist::
    :columns: 3

    - ``INTERNAL_TRG0``
    - ``INTERNAL_TRG1``
    - ``INTERNAL_TRG3``
    - ``INTERNAL_TRG4``
    - ``INTERNAL_TRG5``

.. note::

    You always have to call ``adc_start_cnv()`` individually for every started conversion. If you suspect that your conversions may have overrun, you can check this with the ``adc_has_overrun(chamask)`` function. If you omit the channel mask input param, all the channels will be checked, being essentially the same than calling ``adc_has_overrun(0xff)``.