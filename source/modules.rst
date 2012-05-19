Modules
=======

**Modules are library components that operate with the MCU's internal peripherals.** Every module has its own namespace according to the module name. For example, Power Manager has module namespace of ``pm_``, Realtime Counter falls under the ``rtc_`` namespace, etc. To use the module just include its header file. These header files are also named after the module name. So, for example, to include and use functions that operate with the Power Manager include the file `aery32/pm.h`, ``#include "aery32/pm.h"``.

.. hint::

    If you use C++, instead of .h file, include .hh.

General Periheral Input/Output (gpio)
-------------------------------------

To initialize any pin to output high, there is a oneliner which can be used

.. code-block:: c

    aery_gpio_init_pin(AVR32_PIN_PC04, GPIO_OUTPUT|GPIO_HIGH);

The first argument is the GPIO pin number and the second one is for options. For 100 pin Atmel AVR32UC3, the GPIO pin number is a decimal number from 0 to 69. Fortunately, you do not have to remember which number represent what port and pin. Instead you can use predefined aliases as it was done above with the pin PC04 (5th pin in port C if the PC00 is the 1st).

The available pin init options are:

- GPIO_OUTPUT
- GPIO_INPUT
- GPIO_HIGH
- GPIO_LOW
- GPIO_FUNCTION_A
- GPIO_FUNCTION_B
- GPIO_FUNCTION_C
- GPIO_FUNCTION_D
- GPIO_INT_PIN_CHANGE
- GPIO_INT_RAISING_EDGE
- GPIO_INT_FALLING_EDGE
- GPIO_PULLUP
- GPIO_OPENDRAIN
- GPIO_GLITCH_FILTER
- GPIO_HIZ

These options can be combined with the pipe operator (boolean OR) to carry out several commands at once. Without this feature the above oneliner should be written with two lines of code:

.. code-block:: c

        aery_gpio_init_pin(AVR32_PIN_PC04, GPIO_OUTPUT);
        aery_gpio_set_pin_high(AVR32_PIN_PC04);

Well now you also know how to set pin high, so you may guess that the following function sets it low

.. code-block:: c

    aery_gpio_set_pin_low(AVR32_PIN_PC04);

and that the following toggles it

.. code-block:: c

    aery_gpio_toggle_pin(AVR32_PIN_PC04);

and finally it should not be surprise that there is a read function too

.. code-block:: c

    state = aery_gpio_read_pin(AVR32_PIN_PC04);

But before going any further, let's quickly go through those pin init options. ``FUNCTION_A``, ``B``, ``C`` and ``D`` assing the pin to the specific peripheral function, see datasheet pages 45--48. ``INT_PIN_CHANGE``, ``RAISING_EDGE`` and ``FALLING_EDGE`` enables interrupt events on the pin. Interrupts are trigged on pin change, at the rising edge or at falling edge, respectively. ``GPIO_PULLUP`` connects pin to the internal pull up resistor. ``GPIO_OPENDRAIN`` in turn makes the pin operate as an open drain mode. This mode is gererally used with pull up resistors to guarantee a high level on line when no driver is active. Lastly ``GPIO_GLITCH_FILTER`` activates the glitch filter and ``GPIO_HIZ`` makes the pin high impedance.

.. note::

    Most of the combinations of GPIO init pin options do not make sense and have unknown consecuences.

Usually you want to init several pins at once -- not only one pin. This can be done for the pins that have the same port.

.. code-block:: c

    aery_gpio_init_pins(porta, 0xffffffff, GPIO_INPUT); // initializes all pins input

The first argument is a pointer to the port register and the second one is the pin mask. Aery32 GPIO module declares these ``porta``, ``b`` and ``c`` global pointers to the ports by default. Otherwise, you should have been more verbose and use ``&AVR32_GPIO.port[0]``, ``&AVR32_GPIO.port[1]`` and ``&AVR32_GPIO.port[2]``, respectively.

.. hint::

    As ``porta``, ``b`` and ``c`` are pointers to the GPIO port, you can access its registers with arrow operator, for example, instead of using function ``aery_gpio_toggle_pin(AVR32_PIN_PC04)`` you could write ``portc->ovrt = (1 << 4);`` Refer to datasheet pages 175--177 for GPIO Register Map.

Local GPIO bus
''''''''''''''

AVR32 includes so called local bus interface that connects its CPU to device-specific high-speed systems, such as floating-point units and fast GPIO ports. To enable local bus call

.. code-block:: c

    aery_gpio_enable_localbus();

When enabled you have to operate with `local` GPIO registers. That is because, the convenience functions described above does not work local bus. To ease operating with local bus Aery32 GPIO module provides shortcuts to local ports by declaring ``lporta``, ``b`` and ``c`` global pointers. Use these to read and write local port registers. For example, to toggle pin through local bus you can write

.. code-block:: c

    lporta->ovrt = (1 << 4);

.. note::

    CPU clock has to match with PBB clock to make local bus functional

To disable local bus and go back to normal operation call

.. code-block:: c

    aery_gpio_disable_localbus();

Interrupt Controller (intc)
---------------------------

Before enabling interrupts define and register your interrupt service routine (ISR) functions. First write ISR function as you would do for any other functions

.. code-block:: c

    void myisr_for_group1(void) {
        /* do something */
    }

Then register this function

.. code-block:: c

    aery_intc_register_isrhandler(&myisr_for_group1, 1, 0);

Here the first parameter is a function pointer to your ``myisr_for_group1()`` function. The second parameter defines the which interrupt group calls this function and the last one tells the priority level.

.. hint::

    Refer Table 12-3 (Interrupt Request Signal Map) in datasheet page 41 to see what peripheral belongs to which group. For example, RTC belongs to group 1.

When all the ISR functions have been declared it is time to initialize interrupts. Use the following init function to do all the magic

.. code-block:: c

    aery_intc_init();

After initialization you can enable and disable interrupts globally by using these functions

.. code-block:: c

    aery_intc_enable_globally();

.. code-block:: c

    aery_intc_disable_globally();

Power Manager (pm)
------------------

Power Manager controls integrated oscillators and PLLs among other, well, power related things. One of the first things what to do after the board has been power up is initialization of oscillators and setting the master clock. In Aery32 development board this can be done in this way

.. code-block:: c

    // Starts OSC0
    aery_pm_start_osc(
        0,                  /* oscillator number */
        PM_OSC_MODE_GAIN3,  /* oscillator mode, see datasheet p.74 */
        PM_OSC_STARTUP_36ms /* oscillator startup time */
    );

    // Remember to wait OSC to stabilize
    aery_pm_wait_osc_to_stabilize(0);

    // Initialize f_vco0 of PLL0 to be 132 MHz.
    aery_pm_init_pllvco(
        pll0,               /* pointer to pll address */
        PM_PLL_SOURCE_OSC0, /* source clock */
        11,                 /* multiplier */
        1,                  /* divider */
        false               /* high frequency */
    );

    // Enables PLL0 with divide by two block, f_pll0 = 132 MHz / 2
    aery_pm_enable_pll(pll0, true  /* divide by two */);

    // Remember to wait until PLL0 has been locked
    aery_pm_wait_pll_to_lock(pll0);

    // Set the main clock source to PLL0 that is 66 MHz
    aery_pm_select_mck(PM_MCK_SOURCE_PLL0 /* master clock source */);

.. important::

    PLL VCO frequency, initialized by calling ``aery_pm_init_pllvco()`` has to be between 80--180 MHz or 160--240 MHz with high frequency enabled.

.. hint::

    For your convenience Aery32 PM module declares three global pointers by default, ``pm``, ``pll0`` and ``pll1``.

.. hint::

    To save power disable modules that you do not need, see datasheet page 70. This can be done by changing the peripheral clock masking. The following example disables clocks from the TWI, PWM, SSC, TC, ABDAC and all the USART modules

    .. code-block:: c

        #define PBAMASK_DEFAULT 0x0F
        pm->pbamask = PBAMASK_DEFAULT;

General clocks
''''''''''''''

PM can generate dedicated general clocks. These clocks can be assigned to GPIO pins or used for internal peripherals such as USB that needs 48 MHz clock to be functional. To offer this 48 MHz for USB peripheral you first have to initialize either of the PLLs to work at, for example, 96 MHz frequency:

.. code-block:: c

    aery_pm_init_pllvco(pll1, PM_PLL_SOURCE_OSC0, 16, 1, true); // 192 MHz
    aery_pm_enable_pll(pll1, true); // 96 MHz
    aery_pm_wait_pll_to_lock(pll1);

Then init and enable USB generic clock

.. code-block:: c

    aery_pm_init_gclk(
        PM_GCLK_USBB,        /* generic clock number, see the allocation
                              * table at datasheet p. 63 */
        PM_GCLK_SOURCE_PLL1, /* clock source for the generic clock */
        0                    /* divider, f_gclk = f_src/(2*(div+1)) */
    );
    aery_pm_enable_gclk(PM_GCLK_USBB);

To route generic clock to be at the output of GPIO pin, first init the desired GPIO pin appropriately and then a enable generic clock at this pin. You can do this, for example, to check that USB clock enabled above is correct

.. code-block:: c

    aery_gpio_init_pin(AVR32_PIN_PB19, GPIO_FUNCTION_B);
    aery_pm_init_gclk(PM_GCLK0, PM_GCLK_SOURCE_PLL1, 0);
    aery_pm_enable_gclk(PM_GCLK0);

.. hint::

    Generic clock can be changed when its running by just initializing it again. You do not have to disable it before doing this and you do not have to enable it again.


Real-time Counter (rtc)
----------------------

Real-time counter is for accurate real-time measurements. It enables periodic interrupts at long intervals and the measurement of real-time sequences. RTC has to be init to start counting from the chosen value to the chosen top value. This can be done in this way

.. code-block:: c

    aery_rtc_init(
        0,            /* value where to start counting */
        0xffffffff,   /* top value where to count */
        0,            /* prescaler for RTC clock */
        RTC_SOURCE_RC /* source oscillator */
    );

The available source oscillators are:

- RTC_SOURCE_RC (115 kHz RC oscillator within the AVR32)
- RTC_SOURCE_OSC32 (external low-frequency xtal, not assembled in Aery32 Devboard)

When initialized, remember to enable

.. code-block:: c

    aery_rtc_enable(false);

The boolean parameter here, tells if the interrupts are enabled or not. Here the interrupts are not enabled so it is your job to poll RTC to check whether the top value has been reached or not.

RTC can be used, for example, to provide delay function for the LED toggling application. This is done by initializing RTC to count into the high value and then counting *x* number of cycles forward from the current counter value

.. code-block:: c

    for(;;) {
        aery_gpio_toggle_pin(LED);
        aery_rtc_delay_cycle(57500 /* number of rtc clock cycles, ~1s */);
    }

Here the ``aery_rtc_delay_cycle()`` function is used to wait 57500 RTC cycles that with the initialization made above is approximately 1 second.


Serial Peripheral Bus (spi)
---------------------------

AVR32 UC3A1 includes to separate SPI buses, SPI0 and SPI1. To initialize SPI bus it is good practice to define pin mask for SPI related pins. Refering to datasheet page 45, SPI0 operates from PORTA:

- PA07  NPCS3
- PA08  NPCS1
- PA09  NPCS2
- PA10  NPCS0
- PA11  MISO 
- PA12  MOSI 
- PA13  SCK

So let's define the pin mask for SPI0 with NPCS0 (Numeric Processor Chip Select, also known as slave select):

.. code-block:: c

    #define SPI0_GPIO_MASK ((1 << 10) | (1 << 11) | (1 << 12) | (1 << 13))

Next we have to initialize these pins to the right peripheral function that is FUNCTION A. To do that use pin initializer from gpio module:

.. code-block:: c

    aery_gpio_init_pins(porta, SPI0_GPIO_MASK, GPIO_FUNCTION_A);

Now the GPIO pins have been assigned appropriately and we are ready to initialize SPI0. Let's init it as a master:

.. code-block:: c

    aery_spi_init_master(spi0);

The only parameter is a pointer to the SPI register. Aery32 declares ``spi0`` and ``spi1`` global pointers by default.

.. hint::

    If the four SPI CS pins are not enough, you can use CS pins in multiplexed mode (of course you need an external multiplexer circuit then) and expand number of CS lines to 16. This can be done by bitbanging PCSDEC bit in SPI MR register after the initialization:

    .. code-block:: c
 
        aery_spi_init_master(spi0);
        spi0->MR.pcsdec = 1;

When the SPI peripheral has been initialized as a master, we still have to setup CS line 0 (NPCS0) with the desired SPI mode and shift register width. To set these to SPI mode 0 and 16 bit, call the npcs setup function with the following parameters

.. code-block:: c

    aery_spi_setup_npcs(spi0, 0, SPI_MODE0, 16);

The minimum and maximum shift register widths are 8 and 16 bits, respectively, but you can still :ref:`use arbitrary wide transmission <sending-arbitrary-wide-spi-data>`.

.. hint::

    Chip select baudrate is hard coded to MCK/255. To make it faster you can bitbang the SCRB bit in the CSRX register, where X is the NPCS number:

    .. code-block:: c

         aery_spi_setup_npcs(spi0, 0, SPI_MODE0, 16);
         spi0->CSR0.scbr = 32; // baudrate is now MCK/32

.. hint::

    Different CS lines can have separate SPI mode, baudrate and shift register width.

Now we are ready to enable spi peripheral

.. code-block:: c

    aery_spi_enable(spi0);

There's also function for disabling the desired SPI peripheral ``aery_spi_disable(spi0)``. To write data into SPI bus use the transmit function

.. code-block:: c

    uint16_t rd;
    rd = aery_spi_transmit(spi0, 0x55, 0, true); // writes 0x55 to SPI0, NPCS0

.. hint::
    
    ``aery_spi_transmit()`` writes and reads SPI bus simultaneusly. If you only want to read data, just ignore write data by sending dummy bits.

Here is the above SPI initialization and transmission in complete example code:

.. code-block:: c
    :linenos:

    #include <stdbool.h>
    #include "aery32/gpio.h"
    #include "aery32/spi.h"
    #include "board.h"

    #define SPI0_GPIO_MASK ((1 << 10) | (1 << 11) | (1 << 12) | (1 << 13))

    int main(void)
    {
        uint16_t rd; // received data

        init_board();

        aery_gpio_init_pins(porta, SPI0_GPIO_MASK, GPIO_FUNCTION_A);
        aery_spi_init_master(spi0);
        aery_spi_setup_npcs(spi0, 0, SPI_MODE0, 16);
        aery_spi_enable(spi0);

        for (;;) {
            rd = aery_spi_transmit(spi0, 0x55, 0, true); // writes 0x55 to SPI0, NPCS0
        }

        return 0;
    }

.. _sending-arbitrary-wide-spi-data:

Sending arbitrary wide SPI data
'''''''''''''''''''''''''''''''

The last parameter of ``aery_spi_transmit()`` function indicates for the spi peripheral whether the current transmission is the last on. If true, chip select line rises immediately when the last bit has been written. If it is defined false, CS ine is left low for the next chunk of the transmission. This feature allows to operate with spi buses with arbitrary wide shift registers. For example, to read and write 32 bit wide spi data you can do this:

.. code-block:: c

    uint32_t rd;
    
    aery_spi_setup_npcs(spi0, 0, SPI_MODE0, 8);

    rd = aery_spi_transmit(spi0, 0x55, 0, false);
    rd |= aery_spi_transmit(spi0, 0xf0, 0, false) << 8;
    rd |= aery_spi_transmit(spi0, 0x0f, 0, true) << 16; // complete