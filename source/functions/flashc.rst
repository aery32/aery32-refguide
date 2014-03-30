Flash Controller, `#include <aery32/flashc.h> <https://github.com/aery32/aery32/blob/master/aery32/aery32/flashc.h>`_
================

.. image:: ../../images/avr32_flash_structure.png
    :width: 8 cm
    :target: ../_images/avr32_flash_structure.png
    :alt: AVR32 UC3A1/0 Flash Structure

Flash Controller provides low-level access to the chip's internal flash memory, whose structure has been sketched in the figure above. The init function of the Flash Controller sets the flash wait state (zero or one, ``FLASH_0WS`` or ``FLASH_1WS``, respectively). The last param of the init function enables/disables the sense amplifiers of flash controller.

.. code-block:: c++

    flashc_init(FLASH_1WS, true);

.. warning::

    Setting up the correct flash wait state is extremely important! Note that this has to be set correctly even if the flash read and write operations, described below, are not used. If CPU clock speed is higher than 33 MHz you have to use one wait state for flash. Otherwise you can use zero wait state.

.. warning::

    The uploaded program is also stored into the flash, so it is possible to overwrite it by using the Flash controller. The best practice for flash programming, is starting from the top. ``FLASH_LAST_PAGE`` macro definition gives the number of the last page in the flash. For 128 KB flash this would be 255.

.. warning::

    According to Atmel UC3A1's internal flash supports approximately 100,000 write cycles and it has 15-year data retention. However, you can easily make a for-loop with 100,000 writes to the same spot of flash and destroy your chip in a second. So be careful!

Read and write operations
-------------------------

Flash memory is accessed via pages that are 512 bytes long. This means that you have to make sure that your page buffer is large enough to read and write pages, like this

.. code-block:: c++

    #include <cstring>

    char buf[512];
    flashc_read_page(FLASH_LAST_PAGE, buf); /* Read the last page to separate page buffer */
    strcpy(buf, "foo");                     /* Save string "foo" to page buffer */
    flashc_save_page(FLASH_LAST_PAGE, buf); /* Write page buffer back to flash */

You can also read and write values with different types as long as the page buffer size is that 512 bytes. For example like this

.. code-block:: c++

    extern "C" #include <inttypes.h>

    uint16_t buf16[256];
    uint32_t buf32[128];

Page locking
------------

The flash page can be locked to prevent the write and erase sequences. To lock the first page (page number 0), call

.. code-block:: c++

    flashc_lock_page(0);

Locking is performed on a per-region basis, so the above statement does not lock only page zero, but all pages within the region. There are 16 pages per region so the above command locks also pages 1-15.

To unlock the page, call

.. code-block:: c++

    flashc_unlock_page(0);

You can also use, ``flashc_lock_page_region()`` and ``flashc_unlock_page_region()``. to lock and unlock pages by region. Furthermore, there is a function to check if the page is empty

.. code-block:: c++

    flashc_isempty(0);

User page
---------

The User page is an additional page, outside the regular flash array, that can be used to store
various data, like calibration data and serial numbers. This page is not erased by regular chip
erase. The User page can only be read and write by proprietary commands, which are

.. code-block:: c++

    uint8_t buf[512];
    flashc_read_userpage(buf);

and

.. code-block:: c++

    flashc_save_userpage(buf);

To check whether the user page is empty or not call

.. code-block:: c++

    flashc_userpage_isempty();

.. warning::

    Aery32 board ships with the preprogrammed bootloader, which uses the configuration
    data saved to the first word (32 bits) of user page. This configuration data
    includes the pin configuration for slide switch -- the switch that you have been using
    to vary between bootloader and program modes. So to keep the board slide switch
    working just do not alter the first word of user page.

General purpose fuse bits
-------------------------

You can read all 32 fuse bits into 32 bit variable by using the following command

.. code-block:: c++
    
    uint32_t fusebits;
    fusebits = flashc_read_fusebits();

To write one bit true or false use this:

.. code-block:: c++
    
    flashc_write_fusebit(uint16_t fusebit, bool value);

You can also write fuse bits by a byte at a time, like this

.. code-block:: c++

    flashc_write_fusebyte(0, 0xff);
    flashc_write_fusebyte(1, 0xff);
    flashc_write_fusebyte(2, 0xff);
    flashc_write_fusebyte(3, 0xff);

Now all fuse bits are written to 1. The first parameter is the byte address that can be 0-3 in a 32-bit word and the second one is the byte value.
