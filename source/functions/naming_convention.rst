Naming convention and the calling order
=======================================

The common calling order for modules is the following: 1) initialize, 2) do some extra setuping and after then 3) enable the module. In pseudo code it looks like this

.. code-block:: c++

    module_init();
    module_setup_something();
    module_enable();

The init function may also look like ``module_init_something()``, for example, the SPI can be initialized as a master or slave, so the naming convention declares two init functions for SPI module: ``spi_init_master()`` and ``spi_init_slave()``.

If the module has been disabled, by using ``module_disable()`` function, it can be re-enabled without calling the init or setup functions. Most of the modules can also be reinitialized without disabling it before. For example, general clock frequencies can be changed by just calling the init function again -- this is also the quickest way to change the frequency

.. code-block:: c++

    pm_init_gclk(GCLK0, GCLK_SOURCE_PLL1, 1);
    pm_enable_gclk(GCLK0);

    /* Change the frequency divider */
    pm_init_gclk(GCLK0, GCLK_SOURCE_PLL1, 6);

If you have read through the MCU datasheet, you may wonder why you cannot set all the possible settings with the initialization and setup functions. This is because these functions set sane default values for those properties. These default values should work for 80-90% of use cases. However, sometimes you may have to fine tune these properties to match your needs. This can be done by bitbanging the module registers after you have called the init or setup function. For example, the SPI chip select baudrate is hard coded to `MCK/255` within the ``spi_setup_npcs()`` function. To make SPI bus faster you can bitbang the `SCRB bit` within `CSRn register`, where `n` is the NPCS number.

.. code-block:: c++

    spi_setup_npcs(spi0, 0, SPI_MODE1, 16);
    spi0->CSR0.scbr = 32; /* SPI baudrate for the CS0 is now MCK/32 */

.. note::

    Modules never take care of pin initialization, except GPIO module that's for this specific purpose. So, for example, when initializing SPI you have to take care of pin configuration!

    .. code-block:: c++

        #define SPI0_GPIO_MASK ((1 << 10) | (1 << 11) | (1 << 12) | (1 << 13))
        gpio_init_pins(porta, SPI0_GPIO_MASK, GPIO_FUNCTION_A);