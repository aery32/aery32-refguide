Global variables and error handling
===================================

Every module declares global shortcut variables to the main registers of the module. For example, the GPIO module declares ``porta``, ``b`` and ``c`` global pointers to the MCU ports by default. Otherwise, you should have been more verbose and use ``&AVR32_GPIO.port[0]``, ``&AVR32_GPIO.port[1]`` and ``&AVR32_GPIO.port[2]``, respectively. Similarly, ``pll0`` and ``pll1`` declared in PM module provide quick access to MCU PLL registers etc.

.. hint::

    As ``porta``, ``b`` and ``c`` are pointers to the GPIO port, you can access its registers with arrow operator, for example, instead of using function ``gpio_toggle_pin(AVR32_PIN_PC04)`` you could have written ``portc->ovrt = (1 << 4);`` This is also way how you can set/unset/read/toggle multiple pins at once. Refer to the UC3A0/1 datasheet pages 175--177 for GPIO Register Map.

Error handling
--------------

All module functions will return -1 on general error. This will happen most probably because of invalid parameter values. Greater nagative return values have a specific meaning and a macro definition in the module's header file. For example, ``flashc_save_page()`` of Flash Controller may return -2 and -3, which have been defined with ``E`` prefixed names ``EFLASH_PAGE_LOCKED`` and ``EFLASH_PROG_ERR``, respectively.