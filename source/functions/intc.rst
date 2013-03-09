Interrupt Controller, `#include <aery32/intc.h> <https://github.com/aery32/aery32/blob/master/aery32/aery32/intc.h>`_
====================

Before enabling interrupts define and register your interrupt service routine (ISR) functions. First write ISR function as you would do for any other functions

.. code-block:: c++

    void myisr_for_group1(void) {
        /* do something */
    }

Then register this function

.. code-block:: c++

    intc_register_isrhandler(&myisr_for_group1, 1, 0);

Here the first parameter is a function pointer to your ``myisr_for_group1()`` function. The second parameter defines the which interrupt group calls this function and the last one tells the priority level.

.. hint::

    Refer Table 12-3 (Interrupt Request Signal Map) in datasheet page 41 to see what peripheral belongs to which group. For example, RTC belongs to group 1.

When all the ISR functions have been declared it is time to initialize interrupts. Use the following init function to do all the magic

.. code-block:: c++

    intc_init();

After initialization you can enable and disable interrupts globally by using these functions

.. code-block:: c++

    intc_enable_globally();

.. code-block:: c++

    intc_disable_globally();