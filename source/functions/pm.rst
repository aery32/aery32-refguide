Power Manager
=============

#include `<aery32/pm.h> <https://github.com/aery32/aery32/blob/master/aery32/aery32/pm.h>`_

Power Manager controls integrated oscillators and PLLs among other power related things. By default the MCU runs on the internal RC oscillator (115 kHz). However, it's often preferred to switch to the higher CPU clock frequency, so one of the first things what to do after the power up, is the initialization of oscillators. Aery32 Development Board has 12 MHz crystal oscillator connected to the OSC0. This can be started as

.. code-block:: c++

    pm_start_osc(
        0,               /* oscillator number */
        OSC_MODE_GAIN3,  /* oscillator mode, see datasheet p.74 */
        OSC_STARTUP_36ms /* oscillator startup time */
    );
    pm_wait_osc_to_stabilize(0);

When the oscillator has been stabilized it can be used for the master/main clock

.. code-block:: c++

    pm_select_mck(MCK_SOURCE_OSC0);

Now the CPU runs at 12 MHz frequency. The other possible source selections for the master clock are:

- ``MCK_SOURCE_OSC0``
- ``MCK_SOURCE_PLL0``
- ``MCK_SOURCE_PLL1``

Use PLLs to achieve higher clock frequencies
--------------------------------------------

Aery32 devboard can run at 66 MHz its fastest. To achieve these higher clock frequencies one must use PLLs. PLL has a voltage controlled oscillator (VCO) that has to be initialized first. After then the PLL itself can be enabled.

.. important::

    PLL VCO frequency has to fall between 80--180 MHz or 160--240 MHz with high frequency disabled or enabled, respectively. From these rules, one can realize that the smallest available PLL frequency is 40 MHz (the VCO frequency can be divided by two afterwards).

.. code-block:: c++

    pm_init_pllvco(
        pll0,            /* pointer to pll address */
        PLL_SOURCE_OSC0, /* source clock */
        11,              /* multiplier */
        1,               /* divider */
        false            /* high frequency */
    );

- If ``div > 0`` then ``f_vco = f_src * mul / div``
- If ``div = 0`` then ``f_vco = 2 * mul * f_src``

The above initialization sets PLL VCO frequency of PLL0 to 132 MHz -- that's ``12 MHz * 11 / 1 = 132 MHz``. After then PLL can be enabled and the VCO frequency appears on the PLL output. Remember that you can now also divide VCO frequency by two.

.. code-block:: c++

    pm_enable_pll(pll0, true  /* divide by two */); /* 132 MHz / 2 = 66 MHz */
    pm_wait_pll_to_lock(pll0);

Finally one can change the master clock (or main clock) to be clocked from the PLL0 that's 66 MHz.

.. code-block:: c++

    pm_select_mck(MCK_SOURCE_PLL0);

Fine tune the CPU and Periheral BUS frequencies
-----------------------------------------------

By default the clock domains, that are CPU and the Peripheral Busses (PBA and PBB) equal to the master clock. To fine tune these clock domains, the PM has a 3-bit prescaler, which can be used to divide the master clock, before it has been used for the specific domain. Using the prescaler you can choose the CPU clock between the OSC0 frequency and 40 MHz, that was the lower limit of the PLL. Assuming that the master clock was 66 MHz, the following function call changes the CPU and the bus frequencies to 33 MHz:

.. code-block:: c++

    pm_setup_clkdomain(1, CLKDOMAIN_ALL);

The first parameter defines the prescaler value and the second one selects the clock domain which to set up. Here all the domains are set to equal. The formula is ``f_mck / (2^prescaler)``. With the prescaler selection 0, the prescaler block will be disabled and the selected clock domain equals to the master clock that was the default setting.

The possible clock domain selections are

.. hlist::
    :columns: 2

    - ``CLKDOMAIN_CPU``
    - ``CLKDOMAIN_PBA``
    - ``CLKDOMAIN_PBB``
    - ``CLKDOMAIN_ALL``

.. important::

    PBA and PBB clocks have to be less or equal to CPU clock. Morever, the flash wait state has to been taken into account at this point. If the CPU clock is over 33 MHz, the Flash controller has to be initialized with one wait state, like this ``flashc_init(FLASH_1WS, true)``. If the CPU clock speed is less or equal than 33 MHz, zero wait state is the correct setting for the flash.

.. hint::

    You can combine the clock domain selections with the pipe operator, like this ``CLKDOMAIN_CPU|CLKDOMAIN_PBB``. With this selection the PBA clock frequency won't be changed, but the CPU and PBB will be set up accordingly.

General clocks
--------------

PM can generate dedicated general clocks. These clocks can be assigned to GPIO pins or used for internal peripherals such as USB that needs 48 MHz clock to work. To offer this 48 MHz for the USB peripheral, you have to initialize either of the PLLs to work at 96 MHz frequency. As the PLL0 is commonly used for the master clock, PLL1 has been dedicated for general clocks. First initialize the VCO frequency and then enable the PLL

.. code-block:: c++

    pm_init_pllvco(pll1, PLL_SOURCE_OSC0, 16, 1, true); /* f_pll1_vco = 192 MHz */
    pm_enable_pll(pll1, true); /* f_pll1 = 96 MHz */
    pm_wait_pll_to_lock(pll1);

After then init and enable the USB generic clock

.. code-block:: c++

    pm_init_gclk(
        GCLK_USBB,        /* generic clock number */
        GCLK_SOURCE_PLL1, /* clock source for the generic clock */
        1                 /* divider */
    );
    pm_enable_gclk(GCLK_USBB);

- If ``div > 0`` then ``f_gclk = f_src/(2*div)``
- If ``div = 0`` then ``f_gclk = f_src``

There are five possible general clocks to be initialized:

.. hlist::
    :columns: 2

    - ``GCLK0``
    - ``GCLK1``
    - ``GCLK2``
    - ``GCLK3``
    - ``GCLK_USBB``
    - ``GCLK_ABDAC``

``GCLK_ABDAC`` is for Audio Bitstream DAC, ``GCLK0``, ``GCLK1``, etc. can be attached to GPIO pin, so that you can easily clock external devices. For example, to set generic clock to be at the output of GPIO pin, first init the desired GPIO pin appropriately and then enable the generic clock at this pin. You can do this, for example, to check that USB clock enabled above is correct

.. code-block:: c++

    gpio_init_pin(AVR32_PIN_PB19, GPIO_FUNCTION_B);
    pm_init_gclk(GCLK0, GCLK_SOURCE_PLL1, 1);
    pm_enable_gclk(GCLK0);

.. hint::

    Generic clock can be changed when its running by just initializing it again. You do not have to disable it before doing this and you do not have to enable it again.

Save power and use only the peripherals that you need
-----------------------------------------------------

By default all modules are enabled. You might be interested in to disable modules you are not using. This can done via the peripheral clock masking. The following example disables clocks from the TWI, PWM, SSC, TC, ABDAC and all the USART modules

.. code-block:: c++

    #define PBAMASK_DEFAULT 0x0F
    pm->pbamask = PBAMASK_DEFAULT;

Remember to wait when the change has been completed

.. code-block:: c++

    while (!(pm->isr & AVR32_PM_ISR_MSKRDY_MASK));
        /* Clocks are now masked according to (CPU/HSB/PBA/PBB)_MASK
         * registers. */

How much is the clock?
----------------------

Sometimes the current clock frequencies has to be checked programmatically. To get the main clock use the ``pm_get_fmck()`` function

.. code-block:: c++

    main_hz = pm_get_fmck();

Respectively, the clock domains can be fetched like this

.. code-block:: c++

    cpu_hz = pm_get_fclkdomain(CLKDOMAIN_CPU);
    pba_hz = pm_get_fclkdomain(CLKDOMAIN_PBA);
    pbb_hz = pm_get_fclkdomain(CLKDOMAIN_PBB);

These functions assume that OSC0 and OSC1 frequencies are 12 MHz and 16 MHz, respectively. If other oscillator frequencies are used, change the default values by editing ``CXXFLAGS`` in ``aery32/Makefile``.