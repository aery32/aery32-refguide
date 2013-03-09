Real-time Counter
=================

#include `<aery32/rtc.h> <https://github.com/aery32/aery32/blob/master/aery32/aery32/rtc.h>`_

Real-time counter is for accurate real-time measurements. It enables periodic interrupts at long intervals and the measurement of real-time sequences. RTC has to be init to start counting from the chosen value to the chosen top value. This can be done in this way

.. code-block:: c++

    rtc_init(
        RTC_SOURCE_RC, /* source oscillator */
        0,             /* prescaler for RTC clock */
        0,             /* value where to start counting */
        0xffffffff     /* top value where to count */
    );

The available source oscillators are:

- ``RTC_SOURCE_RC`` (115 kHz RC oscillator within the AVR32)
- ``RTC_SOURCE_OSC32`` (external low-frequency xtal, not assembled in Aery32 Devboard)

When initialized, remember to enable it too

.. code-block:: c++

    rtc_enable(false);

The boolean parameter here, tells if the interrupts are enabled or not. Here the interrupts are not enabled so it is your job to poll RTC to check whether the top value has been reached or not.