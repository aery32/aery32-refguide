String functions, ``#include <aery32/string.h>``
================================================

Generally you should use sprintf/sscanf family of functions to do string manipulations but sometimes it would be nice to use some smaller alternatives to save memory. In addition, you may like to avoid using these standard functions because of some nasty side effects like stack overflown etc.

Integer number to string
------------------------

Use this snippet to convert integer number to string

.. code-block:: c++

    char str[10] = "";
    itoa(100, str);

``itoa()`` returns the pointer to the ``str`` so you can use it with print functions. If you like to know how many characters were written in conversion, you can save the number into additional variable, like this

.. code-block:: c++

    int n;
    char str[10] = "";
    itoa(100, str, &n);

``utoa()`` works similarly with ``itoa()`` but only for unsigned integers.

Double to string
----------------

This example converts pi to string with 8 decimal numbers.

.. code-block:: c++

    char str[10] = "";
    dtoa(3.14159265, 8, str);
