Delay functions
===============

There are three convenience delay functions that are intented to be used for short delays, from microseconds to milliseconds.

.. code-block:: c

    aery_delay_us(10); // wait 10 microseconds
    aery_delay_ms(500); // wait half a second

These functions are dependent on the master clock frequency that has to be provided via ``F_CPU`` definition before the ``delay.h`` header file has been included, like this

.. code-block:: c

    #define F_CPU 66000000UL
    #include <aery32/delay.h>

If ``F_CPU`` is not defined, you can still use a low level delay function that takes the number of master clock cycles as a parameter

.. code-block:: c

    aery_delay_cycles(1000); // wait 1000 master clock cycles


Use RTC for long delays
-----------------------

Realtime counter (RTC) is better for long time delays that last minutes, hours, days or even months or years. However, it is also better to use RTC for one second delays too. For example, to toggle the LED every second one can use ``aery_rtc_delay_cycle`` function.

.. code-block:: c
    :linenos:

    #include <stdbool.h>
    #include <aery32/gpio.h>
    #include <aery32/rtc.h>
    #include "board.h"

    int main(void)
    {
        init_board();
        aery_gpio_init_pin(LED, GPIO_OUTPUT);

        aery_rtc_init(0, 0xffffffff, 0, RTC_SOURCE_RC);
        aery_rtc_enable(false);

        for(;;) {
            aery_gpio_toggle_pin(LED);
            aery_rtc_delay_cycle(57500);
        }

        return 0;
    }

The application above first initialize RTC to start counting from zero to 4 294 967 295, which is ``0xffffffff`` in hexadecimal. The third parameter is a prescaler for the RTC clock, which was set to in-built RC clock, ``RTC_SOURCE_RC``. The RC clock within the AVR32 UC3A1 works at 115 kHz frequency, but it's frequency is always divided by two. This means that the counter will be incremented every 1/(115000/2) second (approx. every 17.4 us).

When initialized, the RTC has to be still enabled before it starts counting. This has been done right after the ``aery_rtc_init()`` function by calling ``aery_rtc_enable()`` function. The first parameter indicates whether the interrupts are enabled too. Here we did not use those and thus false was passed as a parameter. However, when much longer delays are desired, interrupts should be used. Below is an example program that toggles the LED every minute through RTC interrupt event function, ``isrhandler_rtc()``.

.. code-block:: c
    :linenos:

    #include <stdbool.h>
    #include <aery32/gpio.h>
    #include <aery32/intc.h>
    #include <aery32/rtc.h>
    #include "board.h"

    #define LED AVR32_PIN_PC04

    void isrhandler_rtc(void)
    {
        aery_gpio_toggle_pin(LED);
        aery_rtc_set_value(0);
        aery_rtc_clear_interrupt(); // Remember to clear RTC interrupt
    }

    int main(void)
    {
        init_board();
        aery_gpio_init_pin(LED, GPIO_OUTPUT|GPIO_HIGH);

        aery_rtc_init(0, 60*115000/2, 0, RTC_SOURCE_RC);

        aery_intc_init();
        aery_intc_register_isrhandler(&isrhandler_rtc, 1, 0);
        aery_intc_enable_globally();

        aery_rtc_enable(true);

        for(;;) {
        }

        return 0;
    }