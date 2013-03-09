Pulse Width Modulation, `#include <aery32/pwm.h> <https://github.com/aery32/aery32/blob/master/aery32/aery32/usart.h>`_
======================

Start by initializing the PWM channel which you want to use

.. code-block:: c++

    pwm_init_channel(2, MCK);

The above initializer sets channel's two PWM frequency equal to the main clock and omits the duration and period for default values. The default values for the duration and period are 0 and 0xFFFFF, respectively. If you like to start the channel with different values, you could have defined those too like this

.. code-block:: c++

    pwm_init_channel(2, MCK, 50, 100);

This gives you duty cycle of 50% from start. The maximum value for both the duration and the period is 0xFFFFF. It is also worth noting that when the period is set to its maximum value, the channel's duty cycle can be set most accurately.

The above initializers set the channel's frequency equal to the main clock. The other possible frequency selections are

.. hlist::
    :columns: 3

    - ``MCK_DIVIDED_BY_2``
    - ``MCK_DIVIDED_BY_4``
    - ``MCK_DIVIDED_BY_8``
    - ``MCK_DIVIDED_BY_16``
    - ``MCK_DIVIDED_BY_32``
    - ``MCK_DIVIDED_BY_64``
    - ``MCK_DIVIDED_BY_128``
    - ``MCK_DIVIDED_BY_256``
    - ``MCK_DIVIDED_BY_512``
    - ``MCK_DIVIDED_BY_1024``
    - ``PWM_CLKA``
    - ``PWM_CLKB``

``PWM_CLKA`` and ``PWM_CLKB`` are two extra PWM clock sources. The difference to other sources is an additional linear divider block that comes after the MCK prescaler. To initialize the divider block for the ``PWM_CLKA`` and ``PWM_CLKB`` call

.. code-block:: c++

    pwm_init_divab(MCK, 10, MCK_DIVIDED_BY_2, 10);

Now ``PWM_CLKA`` has the frequency of *MCK / 10* Hz and ``PWM_CLKB`` is *MCK / 2 / 10* Hz. If you don't care about ``CLKB``, you can omit the last two of the parameters like this

.. code-block:: c++

    pwm_init_divab(MCK, 10);

.. note::

    If the divider of ``PWM_CLKA`` or ``PWM_CLKB`` has been set zero, then the PWM clock will equal to the ``MCK``, ``MCK_DIVIDED_BY_2``, etc. Whatever was the chosen prescaler. So it does not make sense to set the divider of the extra PWM clock zero, because then you don't have any extra clock selection.

Setting up PWM mode
-------------------

Before enabling the initialized PWM channel or channels, you may like to setup the channel mode to set PWM alignment and polarity

.. code-block:: c++

    pwm_setup_chamode(2, LEFT_ALIGNED, START_HIGH);

The alignment (left or center, ``LEFT_ALIGNED`` and ``CENTER_ALIGNED``, respectively) defines the shape of PWM function, see datasheet page 680. The polarity defines the polarity of the duty cycle. With ``START_HIGH``, the duty cycle is 100% when *duration / period* of the PWM function gives 1. With ``START_LOW`` you would get 100% duty cycle when the *duration / period* is 0.

Enabling and disabling the PWM
------------------------------

PWM is enabled and disabled by channels. Several channels can be enabled at once to get synchronized output. To enable channels two and four call

.. code-block:: c++

    pwm_enable((1 << 2)|(1 << 4));

Same goes for the disabling the channels. The following call will disable the channel two

.. code-block:: c++

    pwm_disable(1 << 2);

The parameter of the enable and disable functions is a bitmask of the channels to be enabled or disabled. There is also function to check if the channel has been enabled already. The following snippet will do something if the channel two was already enabled

.. code-block:: c++

    if (pwm_is_enabled(1 << 2)) {
        /* Do something */
    }

Modulating the PWM output waveform
----------------------------------

You can modulate the PWM output waveform when it is active by changing its duty cycle like this

.. code-block:: c++

    pwm_update_dutycl(2, 0.5);

The above function call will update the channel's two duty cycle to 50%. In case you want to specify completely new values for the period and duration use these two functions

.. code-block:: c++
    
    pwm_update_period(2, 0x1000);
    pwm_update_duration(2, 0x10);

Furthermore, to keep PWM output at the desired state for the amount of periods, before changing its state again, use the wait function. This also allows you to do updates from the beginning of the next period and thus avoiding to overwrite the value too soon. For example, to wait 100 periods on channel two call

.. code-block:: c++
    
    pwm_wait_periods(2, 100);

With the combination of the update functions and the wait function, you can make a smoohtly blinking LED, just like this

.. code-block:: c++

    uint8_t channel = 2;
    uint32_t duration = 0;
    uint32_t period = 0x1000;

    for (;;) {
        for (; duration < period; duration++) {
            pwm_update_duration(channel, duration);
            pwm_wait_periods(channel, 500);
        }
        for (; duration > 0; duration--) {
            pwm_update_duration(channel, duration);
            pwm_wait_periods(channel, 500);
        }
    }

.. note::

    Duration has to be smaller or equal to period.