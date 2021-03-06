Delay functions, ``#include <aery32/delay.h>``
==============================================

There are three convenience delay functions that are intented to be used for short delays, from microseconds to milliseconds.

.. code-block:: c

    delay_us(10); /* wait 10 microseconds */
    delay_ms(500); /* wait half a second */

These functions are dependent on the CPU clock frequency that has to be provided via ``F_CPU`` definition before the ``delay.h`` header file has been included, like this

.. code-block:: c

    #define F_CPU 66000000UL
    #include <aery32/delay.h>

If ``F_CPU`` is not defined, you can still use a low level delay function that takes the number of master clock cycles as a parameter

.. code-block:: c

    delay_cycles(1000); /* wait 1000 master clock cycles */

.. hint::

    You can make smaller than 1 us delays with ``delay_cycles()``.


Use RTC for long delays
-----------------------

Realtime counter (RTC) is better for long time delays or countdowns that last minutes, hours, days or even months or years. For example, to toggle the LED every second one can use ``rtc_delay_cycles()`` function.

.. code-block:: c++
    :linenos:

    #include <aery32/gpio.h>
    #include <aery32/rtc.h>
    #include "board.h"

    using namespace aery;

    int main(void)
    {
        init_board();
        gpio_init_pin(LED, GPIO_OUTPUT);

        rtc_init(0, 0xffffffff, 0, RTC_SOURCE_RC);
        rtc_enable(false);

        for(;;) {
            gpio_toggle_pin(LED);
            rtc_delay_cycles(57500);
        }

        return 0;
    }

The application above first initialize the RTC to start counting from zero to 4 294 967 295, which is ``0xffffffff`` in hexadecimal. The third parameter is a prescaler for the RTC clock, which was set to be the in-built RC clock. The RC clock within the AVR32 UC3A1 works at 115 kHz frequency, but it's frequency is always divided by two when used for RTC. This means that the counter will be incremented every 1/(115000/2) second (approx. every 17.4 us).

When initialized, the RTC has to be enabled before it starts counting. This has been done by calling the ``rtc_enable()`` function. The first parameter indicates whether the interrupts are enabled too. Here we did not use those and thus false was passed as a parameter. However, when much longer delays or time calculations are desired, interrupts should be used. Below is an example program that toggles the LED every minute through the RTC interrupt event function, ``isrhandler_rtc()``.

.. code-block:: c++
    :linenos:

    #include <aery32/gpio.h>
    #include <aery32/intc.h>
    #include <aery32/rtc.h>
    #include "board.h"

    using namespace aery;

    void isrhandler_rtc(void)
    {
        gpio_toggle_pin(LED);
        rtc_clear_interrupt(); /* Remember to clear RTC interrupt */
    }

    int main(void)
    {
        init_board();
        gpio_init_pin(LED, GPIO_OUTPUT|GPIO_HIGH);

        rtc_init(0, 60*115000/2, 0, RTC_SOURCE_RC);

        intc_init();
        intc_register_isrhandler(&isrhandler_rtc, 1, 0);
        intc_enable_globally();

        rtc_enable(true);

        for(;;) {
        }

        return 0;
    }